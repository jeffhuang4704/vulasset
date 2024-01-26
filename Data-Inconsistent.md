# Data Inconsistent

### History
- v1 - 2024/01/26 initial version

## Introduction
We have encountered an issue related to the fluctuating scanning status. Whether users observe the status on the UI or retrieve it through the REST endpoint, the status is inconsistent.

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Reproduce](#section-2-reproduce)
- [Section 3: Log snippet](#section-3-log-snippet)
- [Section 4: Log](#section-4-related-cases)

## Section 1: Overview

Based on the currently reproduced cases, it appears to occur in multi-controller environments.
This issue occurs in both version 5.2.4 and the latest version 5.3.

## Section 2: Reproduce

We have identified two methods to reproduce this issue:
- Allow the controller to scan some workloads, like 800 workloads.
- Deploy a specific application, yaml file can be found at https://raw.githubusercontent.com/microservices-demo/microservices-demo/master/deploy/kubernetes/complete-demo.yaml

⚠️⚠️⚠️ Note: In a multi-controller environment, when observing data through the UI or calling the REST endpoint via a service, incorrect results may be obtained, as the data may come from any one of the three controllers. ⚠️⚠️⚠️   

This might lead you to believe that there is no reproduction.

**To confirm the reproduction**
If you observe unfinished statuses from the UI, the issue is reproduced.   
However, if all statuses appear as finished, it might not be accurate, as the result may come from the functioning controller. To verify this, follow these steps:


**Call the REST endpoint directly**

Call the individual REST endpoint to retrieve all asset IDs and their respective statuses, and save the results to a file. This operation needs to be performed within the cluster.

This process needs to be executed on all three controllers, resulting in three sets of outcomes.

```
curl -k -H "Content-Type: application/json" -H "X-Auth-Token: $TOKEN" "https://$K8sNodeIP:$ControllerSvcPORT/v1/workload?brief=False" | jq -r '.workloads | sort_by(.id)[] | "\(.id) \(.scan_summary.status)"' > $K8sNodeIP.txt
```

**Diff the result**

```
root@toolbox-69fbf76d68-54f7q:~/jeff# diff 10.32.0.25.txt 10.44.0.3.txt
61c61
< 81830faa1e7514f9505217d0544f1b4a8c5d567d9a3459633d9742d02d17af4a finished
---
> 81830faa1e7514f9505217d0544f1b4a8c5d567d9a3459633d9742d02d17af4a scanning
121c121
< f9505c36c2d948c6967af275eb2867c45919f6b9269965ce09eb1980450fe53b finished
---
> f9505c36c2d948c6967af275eb2867c45919f6b9269965ce09eb1980450fe53b scanning
```

## Section 3: Log snippet

Here is an example of an inconsistent entry with ID (59d6903...).

By solely observing this log, the #3 controller did not receive the status=Finished call. Consequently, it did not have a chance to invoke `scanDone()` and update its status.

```
// #1 controller-hzp6w (lead)
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  status=scheduled
cache.(*scanTask).Handler: - id=59d6903...
..
cache.updateScanState:  ...   status=scanning
cluster.Put:  value={"scanned_at":"0001-01-01T00:00:00Z","status":"scanning"}
..
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  status=scanning
..
cache.putScanReportToCluster: id=59d690.. result=succeeded type=CONTAINER
cache.updateScanState:  ...   status=finished
..
cache.scanStateHandler:  ...  type=modify
2024-01-25T23:51:39.402 cache.scanStateHandler:  ...  status=finished
....
cache.scanDone: - id=59d690
```

```
// #2 controller-27l9f
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  status=scheduled
....
......
cache.scanStateHandler:  ...  type=modify
cache.scanStateHandler:  ...  type=modify
2024-01-25T23:51:39.404 cache.scanStateHandler:  ...  status=status=finished
....cache.scanDone: - id=59d690.. result=succeeded type=CONTAINER
```

```
// #3 controller-twqkl
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  type=add
cache.scanStateHandler:  ...  status=scheduled
....
......
2024-01-25T23:51:39.424 cache.scanStateHandler:  ...  type=modify   👈👈👈 (last line of this asset ID)
```

## Section 4: Related Cases

[NVSHAS-8590 Inconsistent scan reports from the Neuvector API](https://jira.suse.com/browse/NVSHAS-8590?filter=-1)  
[NVSHAS-8555 registry scan may not work properly](https://jira.suse.com/browse/NVSHAS-8555?filter=-1)