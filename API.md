## Vulnerablity Page changes (v4)

### History

v11 - 2024/12/15 Add feed rating in risk page

NVSHAS-9614 Include a sortable feed_rating column into vulnerabilities tab

```
# response
/v1/vulasset?token=$TOKEN&row=100&start=0&orderbyColumn=feed_rating&orderby=desc
{
  ....
  "feed_rating": "Important",   👈 new field
  "last_modified_timestamp": 1547424000,
  "link": "https://access.redhat.com/errata/RHSA-2019:0049",
  "name": "RHSA-2019:0049"
  ....
  ....
}

```

**Sorting**

```
/v1/vulasset?token=$TOKEN&row=100&start=0&orderbyColumn=feed_rating&orderby=desc
                                                ☝️                   ☝️
orderbyColumn=feed_rating   👈
orderby=desc | asc
```

**Quick Filter**
The 'Feed Rating' column data is now included in the scope of the Quick Search feature.

**Testing**
A testing environment has been set up in the lab and is accessible at `10.1.45.40:31786`.

<details><summary>previous changes</summary>

```
- v1 - 2023/12/13
- v2 - 2023/12/17
- v3 - 2023/12/19, update `v1/assetvul`
- v4 - 2023/12/20, quick search
- v5 - 2024/01/08, adjust query filter `scoreV2`, `scoreV3`. Now the `last_modified_timestamp` is for both `/v1/vulasset` and `v1/assetvul`
- v6 - 2024/01/11, adjust quick filter behavior
- v7 - 2024/01/17, add `lastmtime` for `GET v1/vulasset`
- v8 - 2024/01/24, add `impact` sort column
- v9 - 2024/03/13 (v5.3.1)
- v10 - 2024/03/20 (plan for v5.3.2, [NVSHAS-7522](https://jira.suse.com/browse/NVSHAS-7522) )

```

</details>

## Table of Contents

- [Usage for `v1/vulasset`](#usage-for-v1vulasset)
  - [Starting a Query Session](#starting-a-query-session)
  - [Navigating Within a Query Session](#navigating-within-a-query-session)
  - [Quick Filter Within a Query Session](#quick-filter-within-a-query-session)
- [Usage for `v1/assetvul`](#usage-for-v1assetvul)
  - [v5.3.2 - embed image information](#the-response-in-v532) 🆕
- [Testing Environment](#testing-environment)

## Usage for `v1/vulasset`

To utilize the v1/vulasset feature, begin by initiating a query session through the submission of advanced filters and sorting options. The backend will generate a session specific to the provided query and furnish you with a token for navigating within that session. If a user modifies the filter criteria, it is essential to initiate a new query session.

### Starting a Query Session

To initiate a query, use the POST method on the endpoint `/v1/vulasset`, providing advanced filters and sorting options within the request body.

<details><summary>Request</summary>

```

HTTP POST v1/vulasset

Request Body
{
"publishedType": "before", // "all" (default), "before", "after"; UI sends all if no published timestamp is selected
"publishedTime": 1605353432, // time tick
"packageType": "withFix", // "all" (default), "withfix", "withoutfix" => "withFix", "withoutFix" (2024/03/13, for v5.3.1)
"severityType": "high", // "all" (default), "high", "medium", "low"
"scoreType": "v3", // "v2", "v3" (default)

    "last_modified_timestamp": 1605353432 // time tick
    "scoreV2": [6, 10],          // score v2 filter
    "scoreV3": [5, 10],          // score v3 filter

    "matchTypeService": "contains", // "contains", "equals"
    "serviceName": "svc",

    "matchTypeNs": "contains",      // "contains", "equals"
    "selectedDomains": [
        "ns1",
        "ns7"
    ],
    "matchTypeImage": "equals",     // "contains", "equals"
    "imageName": "img",

    "matchTypeNode": "equals",      // "contains", "equals"
    "nodeName": "node",

    "matchTypeContainer": "equals", // "contains", "equals"
    "containerName": "cont",

    "orderbycolumn": "scorev3",     // "name" (default), "score", "score_v3", "published_timestamp", "impact"
    "orderby": "desc",              // "asc", "desc" (default)

    "viewType": "all",              // "all" (default), "containers", "infrastructure", "registry"

}
```

</details>

<details><summary>Response</summary>

```
Response Body

{
"debug_error": 0, // please ignore fields start with "debug".
"debug_error_message": "",
"debug_perf_stats": [
"step-1, get allowed resources, took=792.405µs",
...
],
"query_token": "eff501a8ce17", 👈 // need to bring this value in the URL parameter to navigate this query session
"summary": {
"count_distribution": { 1️⃣
"high": 20, // In the searched result,
"low": 10, // how many distinct CVEs has high severity (based on the score type, v2 or v3)
"medium": 15, //

            "container": 3, // In the searched result, how many distinct CVEs has container impact.
            "image": 8,     //  .... has image impact.
            "node": 12,     //  .... has node impact.
            "platform": 5   //  .... has platform impact.
        },
        "top_images": [           2️⃣
            {
                "display_name": "Image1",
                "high": 5,
                "id": "0",
                "low": 2,
                "medium": 3
            }
            ...
        ],
        "top_nodes": [              3️⃣
            {
                "display_name": "Node1",
                "high": 8,
                "id": "0",
                "low": 2,
                "medium": 4
            },
            ...
        ]
    },
    "total_matched_records": 161,   👈
    "total_records": 279            👈

}
```

</details>

### Navigating Within a Query Session

To navigate within an existing search session, make an `HTTP GET` request to the same endpoint (`/v1/vulasset`) with the following query parameters.

Refer to the detailed fields and their corresponding values in the following raw data section.

If the user opts for a different column sorting, we can incorporate this change within the current query session by adding the "orderbyColumn" and "orderby" URL parameters, eliminating the need to create a new query session and thereby enhancing performance.

<details><summary>Request</summary>

```

GET v1/vulasset?token=eff501a8ce17&start=0&row=100

1️⃣ token: Indicates the query session; you can find this token in the response body.
2️⃣ start: Specifies the starting row.
3️⃣ row: Defines the number of rows to fetch. Use -1 to fetch all rows.
4️⃣ orderbyColumn: Use different column to sort
5️⃣ orderby: Use different sort type
6️⃣ qf: quick filter search term, this will be used to search the CVE Name, score (depends on the scoretype) and feed rating
7️⃣ scoretype: v3 or v2
8️⃣ lastmtime: 1605353432 // time tick 👈

```

</details>

<details><summary>Response</summary>

```
{
    "qf_matched_records": 0,    // 👈 how many quick filter matched records
    "vulnerabilities": [
        {
            "description": "Docker before 1.5 allows local users to have unspecified impact via vectors involving unsafe /tmp usage.",
            "images": [
                {
                    "display_name": "wurstmeister-zookeeper:latest",
                    "domains": null,
                    "id": "dc00f1198a444104617989bde31132c22d7527c65e825b9de4bbe6313f22637f",
                    "policy_mode": ""
                }
            ],
            "last_modified_timestamp": 1507923932,
            "link": "http://people.ubuntu.com/~ubuntu-security/cve/CVE-2014-0047",
            "name": "CVE-2014-0047",
            "nodes": [
                {
                    "display_name": "ubuntu2204-A",
                    "domains": [],
                    "id": "ubuntu2204-A:J34I:M2CR:RM54:Z24R:HRMR:2DLN:ISHL:2AVY:FW63:SAKO:KEBW:33IO",
                    "policy_mode": "Discover"
                },
                {
                    "display_name": "ubuntu2204-B",
                    "domains": [],
                    "id": "ubuntu2204-B:SGC3:5QOQ:EVL4:WLO5:MP2D:SESI:KF2R:T6UF:OZ7V:IJFT:IGBI:JAMU",
                    "policy_mode": "Discover"
                },
                {
                    "display_name": "ubuntu2204-C",
                    "domains": [],
                    "id": "ubuntu2204-C:BW54:QWBZ:GOKY:BH37:27FH:ZMG6:SHQ4:UXIZ:SQXM:TSDT:GQBB:YQY6",
                    "policy_mode": "Discover"
                }
            ],
            "packages": {
                "docker.io": [
                    {
                        "fixed_version": "1.6.2~dfsg1-1ubuntu4~14.04.1",
                        "package_version": "1.0.1~dfsg1-0ubuntu1~ubuntu0.14.04.1"
                    }
                ]
            },
            "platforms": [],
            "published_timestamp": 1507923932,
            "score": 4.6,
            "score_v3": 7.8,
            "severity": "High",
            "vectors": "AV:L/AC:L/Au:N/C:P/I:P/A:P",
            "vectors_v3": "CVSS:3.0/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H",
            "workloads": [
                {
                    "display_name": "wurstmeister",
                    "domains": [
                        "default"
                    ],
                    "id": "d3ecf62b28ec00259f46041311fb9ea7231c6ae673b37ddb98ae0e71493de2ca",
                    "image": "quay.io/nvlab/wurstmeister-zookeeper:latest",
                    "policy_mode": "Discover",
                    "service": "my-dep1.default"
                }
            ]
        }
    ]
}
```

</details>

### Quick Filter Within a Query Session

The current UI design includes a Filter function that enables users to refine their search within the existing results.
To achieve this, you can utilize the same endpoint with an `qf` URL parameter to specify the search term and `scoretype` to indicate score type.

The search scope is currently limited to the [name], [score] and [feed_rating] fields.

<details><summary>Request</summary>

```
Request

GET /v1/vulasset?token=aaa&qf=term&scoretype=v3&start=0&row=100

qf : indicate the quick filter term
scoretype: indicate the score type, values are "v2", "v3"
```

</details>

<details><summary>Response</summary>

```
Reponse

{
    "qf_matched_records": 0, // 👈 how many quick filter matched records
    "vulnerabilities": [...]
}

```

</details>

Quick filter:

<p align="left">
<img src="./materials/quick-search.png" width="40%">
</p>

Quick filter scope on these two values only:

<p align="left">
<img src="./materials/quick-search2.png" width="40%">
</p>

## Usage for `v1/assetvul`

This endpoint is to serving printouts of the Assets View, presenting vulnerability data associated with a particular asset.

No pagination functionality is required for this endpoint. It is specifically implemented for generating printouts, and therefore, the entire content can be retrieved in a single request.

### The request

Use `HTTP POST` for this request.

The output will be sourced from the existing query session within the main UI. As a result, it is essential to include the query_token in the URL parameters. To refine the output, you can include the `lastModifiedTime` in the request body.

```
HTTP POST v1/vulasset

POST v1/vulasset?token=eff501a8ce17

Request Body
{
    "last_modified_timestamp": // 1605353432;  or 0 for [all]
}
```

### The response

The response from this API call contains all the necessary data in a single retrieval.
The response strucutre like this.

<p align="left">
<img src="./materials/assetvul-2.png" width="40%">
</p>

The `vulnerabilities` array comprises distinct CVEs from `workloads`, `nodes`, `platforms`, and `images`. Its structure mirrors the output of `v1/vulasset`, with certain fields removed to optimize bandwidth usage.

🔑 The CVE name will be prefixed with its severity indicator: H*, M*, or L\_. Callers should extract the format based on the intended display.

<details><summary>Response</summary>

```
{
    "workloads": [
        {
            "name": "kube-proxy-84bkd",
            "domain": "kube-system",
            "applications": [
                "TCP/10249",
                "TCP/10256"
            ],
            "policy*mode": "Discover",
            "service_group": "nv.kube-proxy.kubesystem",
            "high": 12,
            "medium": 5,
            "low": 2,
            "vulnerabilities": [
                "H_CVE-2023-29383",
                "L_CVE-2022-4899"
            ],
            "scanned_at": "2023-12-11T01:21:10Z"
        }
    ],
    "nodes": [
        {
            "name": "master",
            "os": "Ubuntu 22.04 LTS",
            "kernel": "5.15.0-71-generic",
            "cpus": 4,
            "memory": 8335712256,
            "containers": 16,
            "policy_mode": "Discover",
            "high": 12,
            "medium": 5,
            "low": 2,
            "vulnerabilities": [
                "H_CVE-2023-29383",
                "L_CVE-2022-4899"
            ],
            "scanned_at": "2023-12-11T01:21:10Z"
        }
    ],
    "platforms": [
        {
            "name": "Kubernetes",
            "version": "1.23.17",
            "base_os": "",
            "high": 12,
            "medium": 5,
            "low": 2,
            "vulnerabilities": [
                "H_CVE-2023-29383",  👈 \*\* prefixed with its severity indicator: H* for high, M* for medium, or L* for low.
                "L_CVE-2022-4899"
            ]
        }
    ],
    "images": [
        {
            "name": "gcr.io/google-samples/microservices-demo/adservice:v0.5.0",
            "high": 12,
            "medium": 5,
            "low": 2,
            "vulnerabilities": [
                "H_CVE-2023-29383",
                "L_CVE-2022-4899"
            ]
        }
    ],
    "vulnerabilities": [
        {
            "description": "Docker before 1.5 allows local users to have unspecified impact via vectors involving unsafe /tmp usage.",
            "last_modified_timestamp": 1507923932,
            "link": "http://people.ubuntu.com/~ubuntu-security/cve/CVE-2014-0047",
            "name": "CVE-2014-0047",
            "packages": {
                "docker.io": [
                    {
                        "fixed_version": "1.6.2~dfsg1-1ubuntu4~14.04.1",
                        "package_version": "1.0.1~dfsg1-0ubuntu1~ubuntu0.14.04.1"
                    }
                ]
            },
            "published_timestamp": 1507923932,
            "score": 4.6,
            "score_v3": 7.8,
            "severity": "High",
            "vectors": "AV:L/AC:L/Au:N/C:P/I:P/A:P",
            "vectors_v3": "CVSS:3.0/AV:L/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H"
        }
    ]
}

```

</details>

### The response (in v5.3.2)

Two new fields (`id` and `regname`) were added in the `images` object.
Caller can use these two fields to fetch detail image information using endpoint `/v1/scan/registry/{registry}/image/{image-id}`

<details><summary>Example output</summary>

```
{
    "images": [
        {
            "high": 9,
            "id": "49176f190c7e9cdb51ac85ab6c6d5e4512352218190cd69b08e6fd803ffbf3da",   👈
            "low": 0,
            "medium": 13,
            "name": "alpine:3.17.0",
            "regname": "nvbox",     👈
            "vulnerabilities": [
                "H_CVE-2022-4450",
                "M_CVE-2023-3446",
                "M_CVE-2024-0727"
            ]
        }
    ]
}
```

</details>

## Testing environment

A testing environment has been set up in the lab and is accessible at `10.1.45.40:31786`.
