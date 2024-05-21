# NV db-pod for v5.4

## History

- v1 - 2024/05/20

## Table of Contents

- [Background](#background)
- [Design](#design)
- [Discussion](#discussion)

## Background

In version 5.4, we will introduce a database pod, which functions as a database service within the NeuVector deployment. This feature is optional and can be deployed by users at their discretion.

TODO: mention if user do not have memory pressure in their environment, they don't need to enable while upgrade to v5.4.

// what we will store in db-pod
When activated, it will store scan report in the database pod rather than Consul, thereby reducing memory usage in Consul. The data to be stored in the database pod will be scan report, including benchmark data.

Each scan report has two keys in Consul (1) state key and (2) data key
Only `data key` will be migrated.

```
state key => scan/state/report/workload/81712...
data key => scan/data/report/workload/81712...
```

The scan/state will still be stored in Consul, only the scan/data part will be stored in db-pod
The reason is the scan/state plays an important role in the Controller. We want to preserve it so the changes can be minimized.

This document describes the process of migrating pre-existing scan reports from Consul to the database pod.

## Design

Following list some design for the db-pod:

### 1️⃣ db-pod is a `StatefulSet` in Kubernetes within the same namespace with Controller

The db-pod is a `StatefulSet` in Kubernetes which user need to provide `storageClassName` value in the `volumeClaimTemplates`

```
neuvector@ubuntu2204-A:~$ kubectl get statefulset -n neuvector
NAME               READY   AGE
neuvector-db-pod   1/1     80d  👈👈

neuvector@ubuntu2204-A:~$ kubectl get pods -n neuvector
NAME                                         READY   STATUS      RESTARTS      AGE
neuvector-controller-pod-744c7d9684-jh4b4    1/1     Running     0             24h
neuvector-db-pod-0                           1/1     Running     0             29h   👈👈
neuvector-enforcer-pod-4ppbt                 1/1     Running     4 (29d ago)   99d

neuvector@ubuntu2204-A:~$ kubectl get pvc -n neuvector
NAME                           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nvdb-file-neuvector-db-pod-0   Bound    pvc-90914783-f893-4fa5-8e8f-c4cd2e693f89   5Gi        RWO            nfs-client     80d
```

### 2️⃣ SQL over HTTP

reduce the Controller pod and db-pod dependency. db-pod currnetly server as a database service which accept SQL statement. This eliminate the dependency when we need to adjust schema. The logic can be done purely on Controller side.

### 3️⃣ Authentication

Using Mutual TLS (mTLS) for authentication, and only Controller can access db-pod.
It will use same certificate rotation mechanism in v5.4.

### 4️⃣ Migration

When user migrate to v5.4 with db-pod enabled, we want to migrate existing data stored in Consul.
The goal we want to achieve is if DB pod is installed, at a moment during upgrade, the migration happens automatically.

**How it Works**
When enabled, the lead Controller will do the migration task.
It monitor all controllers to ensure they are on the same supported version, v5.4.
Then it perform the migration using a set of goroutines with following steps:

1. Enumerate all scan report pairs from Consul (scan/state and scan/data).
2. Write the scan/data part to the db-pod. It's a HTTP call to db-pod.
3. Delete the scan/data from Consul.

**Performance:**
I conducted a few tests
[test1] A ScanReport containing 142 CVEs with a zipped size of 5 KB. It takes around 13 seconds to migrate 1,000 reports.
[test2] A ScanReport containing 1,371 CVEs with a zipped size of 33 KB. It takes around 17 seconds to migrate 1,000 reports.

The data flow looks like below:
controller => db-pod => nfs server (Persistent Volumes)

## Migration existing data from Consul to db-pod

**Mechanism**  
A scheduler routine within the `lead Controller` will commence every XX minutes to execute the migration process. Migration will only proceed when all Controllers are operating on the same supported version (e.g., v5.4).

Upon initiation, the routine will iterate through all existing scan reports in Consul, inserting them into the db-pod, and subsequently deleting the corresponding key in Consul.

The migration process can be interrupted (e.g., during a Controller pod restart). The scheduler routine will be invoked regularly (interval TBD, possibly every xx minutes).

This aligns with the nature of Kubernetes, where pods are replaceable and can be down at any time. This mechanism ensures that data can be migrated successfully.

Therefore, the migration process is continuous rather than a one-time task.

Consider a scenario where the network-mounted drive, such as an NFS server, is unavailable. In this case, the Controller will not be able to save data to the db-pod. Instead, the Controller can store the data in Consul first. This adds more resilience to NeuVector.

TODO: mention how to read data work if need to get scan report. We will patch GetScanReport() function so it will fetch data from db-pod.
If it's not available it will try local Consul instead.

**Performance**  
(1) Consul read performance (2) db-pod write performance. Use this to estimate the duration.

🍉 Test case 1: migration 10K scan report from Consul to db-pod (how much time it used?) // write performance
🍉 Test case 2: Consule restart, read these 10K scan report from db-pod (how much time it used, we might need to compare with Consul... but how to compare... using log?) // read performance

## Discussion

1. How does user enable the db-pod feature?
2. Can user change their mind by disable it?
