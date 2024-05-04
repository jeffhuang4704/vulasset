# migrate partial Consul data to db-pod (v5.4)

## History

- v1 - 2024/05/04

## Table of Contents

- [Background](#background)
- [Starting a Query Session](#starting-a-query-session)
- [Navigating Within a Query Session](#navigating-within-a-query-session)
- [Testing Environment](#testing-environment)

## Background

In version 5.4, we will introduce a database pod, which functions as a database service within the NeuVector deployment. This feature is optional and can be deployed by users at their discretion. When activated, it will store scan report in the database pod rather than Consul, thereby reducing memory usage in Consul. The data to be stored in the database pod will be scan report, including benchmark data.

This document describes the process of migrating pre-existing scan reports from Consul to the database pod.

## Migration existing data from Consul to db-pod

**Scope**  
Each scan report has two keys in Consul (1) state key and (2) data key
Only `data key` will be migrated.

```
state key => scan/state/report/workload/81712...
data key => scan/data/report/workload/81712...
```

**Mechanism**  
A scheduler routine within the `lead Controller` will commence every XX minutes to execute the migration process. Migration will only proceed when all Controllers are operating on the same supported version (e.g., v5.4).  
Upon initiation, the routine will iterate through all existing scan reports in Consul, inserting them into the database pod, and subsequently deleting the corresponding key in Consul.

**Performance**  
(1) Consul read performance (2) db-pod write performance. Use this to estimate the duration.

Test case 1: migration 10K scan report from Consul to db-pod (how much time it used?)
Test case 2: Consule restart, read these 10K scan report from db-pod (how much time it used, we might need to compare with Consul... but how to compare... using log?)
