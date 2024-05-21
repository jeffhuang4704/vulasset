# NV v5.4, db-pod - Consul data migration

## History

- v1 - 2024/05/20

## Table of Contents

- [Background](#background)
- [Design](#design)
- [Discussion](#discussion)

## Background

In version 5.4, we will introduce a database pod, which functions as a database service within the NeuVector deployment. This feature is optional and can be deployed by users at their discretion. If users do not have memory pressure in their environment, they do not need to enable it when upgrading to v5.4.

When enabled, it will store scan report in the database pod rather than Consul, thereby reducing memory usage in Consul.

### What data will be stored in db-pod

The data to be stored in the database pod will be scan report, including benchmark data.
Each scan report has two keys in Consul: (1) a state key and (2) a data key. The following is an example:

```
state key => scan/state/report/workload/81712...
data key => scan/data/report/workload/81712...
```

The `scan/state` will still be stored in Consul, while only the `scan/data` part will be stored in the db-pod. The reason is that `scan/state` plays an important role in the Controller, and preserving it minimizes changes.

### What code will be changed on Controller

We will patch the write scan report function (like `putScanReportToCluster()`). Originally, it save to Consul, but now it will write to db-pod if enabled. However, if db-pod encounters any issues, such as a network-mounted drive error, it will fall back to writing to Consul.

The migration process (introduced in a later section) will resume to db-pod once it is back online.

We will also patch the read scan report function `GetScanReport()` to fetch data from the db-pod.

### Simple diagram illustration

<p align="left">
<img src="./materials/dbpod1.png" width="50%">
</p>

> [!NOTE]
> This document outlines the process of migrating existing scan reports from Consul to the database pod.

## Design

Following list some design for the db-pod:

### 1️⃣ The core app

The core app in db-pod is an HTTP server that uses an embedded database to store data persistently in Kubernetes-provisioned storage.

### 2️⃣ db-pod is a `StatefulSet` in Kubernetes within the same namespace with Controller

The db-pod is a `StatefulSet` in Kubernetes which user need to provide `storageClassName` value in the `volumeClaimTemplates`

<details><summary>related resources</summary>

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

neuvector@ubuntu2204-A:~$ kubectl get svc -n neuvector
NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                         AGE
neuvector-service-webui                   LoadBalancer   10.101.229.74    <pending>     8443:32731/TCP                  199d
neuvector-svc-admission-webhook           ClusterIP      10.99.41.38      <none>        443/TCP                         199d
neuvector-svc-controller                  ClusterIP      None             <none>        18300/TCP,18301/TCP,18301/UDP   199d
neuvector-svc-crd-webhook                 ClusterIP      10.102.221.234   <none>        443/TCP                         199d
neuvector-svc-db                          ClusterIP      10.106.197.162   <none>        4001/TCP                        80d  👈👈
```

</details>

### 3️⃣ SQL over HTTP

The HTTP endpoints serve two functions. The caller uses these functions to manipulate data in the database. The data manipulation logic resides entirely on the Controller side.

```
    http.HandleFunc("/db/query", s.handlerExpQuery)
    http.HandleFunc("/db/execute", s.handlerExpExecute)
```

This can reduce the dependency between the Controller and the db-pod, as the logic is on the caller side (Controller), which has ultimate control of the schema.

In this initial version, we will store the data originally from Consul.

This generic approach allows us to easily extend it to store other data as needed without changing the db-pod.

### 4️⃣ Authentication

Using Mutual TLS (mTLS) for authentication, and only Controller can access db-pod.

It will use same certificate rotation mechanism in v5.4.

### 5️⃣ 👉👉 Migration

When user migrate to v5.4 with db-pod enabled, we want to migrate existing data stored in Consul.
The goal we want to achieve is if DB pod is installed, at a moment during upgrade, the migration happens automatically.

## Migration existing data from Consul to db-pod

**Mechanism**  
A scheduler routine within the `lead Controller` will commence every XX minutes to execute the migration process. Migration will only proceed when all Controllers are operating on the same supported version (e.g., v5.4).

Upon execution, following routines will be performed.

1. Enumerate all scan report pairs from Consul (scan/state and scan/data).
2. Write the scan/data part to the db-pod. It's a HTTP call which wrap SQL statement to db-pod.
3. Delete the scan/data from Consul.

The migration process can be interrupted (e.g., during a Controller pod restart). The scheduler routine will be invoked regularly (interval TBD, possibly every xx minutes). This aligns with the nature of Kubernetes, where pods are replaceable and can be down at any time. This mechanism ensures that data can be migrated successfully.

Therefore, the migration process is continuous rather than a one-time task.

Consider a scenario where a network-mounted drive, such as an NFS server, is unavailable. In this case, the Controller will not be able to save data to the db-pod. Instead, the Controller can store the data in Consul first. Continuous migration can help in this scenario, adding more resilience to NeuVector.

During migration period, NeuVector function will not be impacted as the patched read/write scan report function will act accordingly.

During the migration period, NeuVector's functionality will not be affected, as the patched read/write scan report function mentioned above will handle it.

**Performance:**

| Scan Report count | CVE Size | Each Report Size | Total Time (migrate 1000 scan report) |
| ----------------- | -------- | ---------------- | ------------------------------------- |
| 1000              | 142      | 5KB in zip       | ~ 13 seconds                          |
| 1000              | 1371     | 33KB in zip      | ~ 17 seconds                          |
| 10000             | 1371     | 33KB in zip      | < 3 minutes                           |

<p align="left">
<img src="./materials/dbpod2.png" width="70%">
</p>

## Discussion

1. How does a user enable the db-pod feature?
2. Does the existence of a db-pod imply automatic usage?
3. Can a user disable it if they change their mind?
