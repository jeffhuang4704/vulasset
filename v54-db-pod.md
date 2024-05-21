# NV db-pod for v5.4

## History

- v1 - 2024/05/20

## Table of Contents

- [Background](#background)

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

### `StatefulSet` in Kubernetes within the same namespace with Controller

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

### SQL over HTTP

reduce the Controller pod and db-pod dependency. db-pod currnetly server as a database service which accept SQL statement. This eliminate the dependency when we need to adjust schema. The logic can be done purely on Controller side.

### Authentication the authentication mechanism will use the same one in the upcoming v5.4. Like certificate rotation will be leveraged.

### Migration

TODO: explain what is migration
TODO: what's our goal // from Gary: we can do some preparation, what in my mind is, if DB pod is installed, at a moment during upgrade, the migration happens automatically

## Migration existing data from Consul to db-pod

**Mechanism**  
A scheduler routine within the `lead Controller` will commence every XX minutes to execute the migration process. Migration will only proceed when all Controllers are operating on the same supported version (e.g., v5.4).  
Upon initiation, the routine will iterate through all existing scan reports in Consul, inserting them into the database pod, and subsequently deleting the corresponding key in Consul.

**Performance**  
(1) Consul read performance (2) db-pod write performance. Use this to estimate the duration.

🍉 Test case 1: migration 10K scan report from Consul to db-pod (how much time it used?) // write performance
🍉 Test case 2: Consule restart, read these 10K scan report from db-pod (how much time it used, we might need to compare with Consul... but how to compare... using log?) // read performance
