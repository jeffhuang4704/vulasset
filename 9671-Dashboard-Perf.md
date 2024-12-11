## NVSHAS-9671 Evaluate loading time on the main dashboard page (lower section)

### APIs used in the dashboard page

```
1️⃣ /v1/scan/config
2️⃣ /v1/workload?view=pod<&f_domain=namespace>
3️⃣ /v1/host?start=0&limit=0<&f_domain=namespace>
4️⃣ /v1/group?view=pod&scope=local<&f_domain=namespace>
5️⃣ /v1/policy/rule<?f_domain=namespace>
6️⃣ /v1/conversation<?f_domain=namespace>
```

### APIs payload - 1️⃣ /v1/scan/config

```

```

### APIs payload - 2️⃣ /v1/workload

It returns workloads array.

```
{
  "workloads": []
}
```

<details><summary>One workload payload</summary>

```
{
      "applications": [],
      "author": "",
      "baseline_profile": "zero-drift",
      "cap_change_mode": true,
      "cap_quarantine": true,
      "cap_sniff": true,
      "children": [],
      "cpus": "",
      "created_at": "2024-11-15T04:21:13Z",
      "display_name": "cattle-cleanup-job--1-mqdbt",
      "domain": "default",
      "enforcer_id": "35f5b7af089514ea45e16efef3351ea94f413a7dac389c8458601e5ae2972b5c",
      "enforcer_name": "neuvector-enforcer-pod-g2hrm",
      "exit_code": 0,
      "finished_at": "",
      "has_datapath": true,
      "host_id": "ubuntu2204-A:J34I:M2CR:RM54:Z24R:HRMR:2DLN:ISHL:2AVY:FW63:SAKO:KEBW:33IO",
      "host_name": "ubuntu2204-A",
      "id": "2426ae02f5f3781f87d2eee18504df65c02fda9fe4a13b189d0279f92f0e6347",
      "image": "k8s.gcr.io/pause:3.4.1",
      "image_created_at": "2021-01-13T04:24:21Z",
      "image_id": "0f8457a4c2ecaceac160805013dc3c61c63a1ff3dee74a473a36249a748e0253",
      "image_reg_scanned": false,
      "interfaces": {
        "eth0": [
          {
            "gateway": "",
            "ip": "10.32.0.23",
            "ip_prefix": 12
          }
        ]
      },
      "labels": {
        "annotation.kubernetes.io/config.seen": "2024-11-14T20:21:13.273401483-08:00",
        "annotation.kubernetes.io/config.source": "api",
        "controller-uid": "9e36113d-ca71-49d5-b7b9-6a7b7ebc6bc4",
        "io.kubernetes.container.name": "POD",
        "io.kubernetes.docker.type": "podsandbox",
        "io.kubernetes.pod.name": "cattle-cleanup-job--1-mqdbt",
        "io.kubernetes.pod.namespace": "default",
        "io.kubernetes.pod.uid": "ce69c1b6-ac2e-4fd4-8b32-c56d66581e2b",
        "job-name": "cattle-cleanup-job"
      },
      "memory_limit": 0,
      "name": "k8s_POD_cattle-cleanup-job--1-mqdbt_default_ce69c1b6-ac2e-4fd4-8b32-c56d66581e2b_0",
      "network_mode": "none",
      "platform_role": "",
      "pod_name": "cattle-cleanup-job--1-mqdbt",
      "policy_mode": "Discover",
      "ports": [],
      "privileged": false,
      "profile_mode": "Discover",
      "run_as_root": true,
      "running": true,
      "scan_summary": {
        "base_os": "",
        "cvedb_create_time": "2024-09-18T00:40:11Z",
        "high": 0,
        "medium": 0,
        "result": "succeeded",
        "scanned_at": "2024-12-11T00:22:14Z",
        "scanned_timestamp": 1733876534,
        "scanner_version": "3.559",
        "status": "finished"
      },
      "secured_at": "2024-11-15T04:24:01Z",
      "service": "cattle-cleanup-job-.default",
      "service_account": "",
      "service_group": "nv.cattle-cleanup-job-.default",
      "service_mesh": false,
      "service_mesh_sidecar": false,
      "started_at": "2024-11-15T04:21:19Z",
      "state": "discover"
    }
```

</details>

### APIs payload - 3️⃣ /v1/host?start=0

### APIs payload - 4️⃣ /v1/group

### APIs payload - 5️⃣ /v1/policy/rule

### APIs payload - 2️⃣ 6️⃣ /v1/conversation
