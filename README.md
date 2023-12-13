## Vulnerablity Page changes (v1)

### History
v1 - 2023/12/13 

## Endpoint
The endpoint name can be updated to enhance clarity. One suggestion is to use `/v2/scan/asset`:

```
v1/vulasset
```

## Usage
Given that the request involves filtering and sorting, using `HTTP GET` might not be optimal. After discussions with Steven from the Security Page, we recommend the following pagination protocol.

For a fresh query, use PATCH to `v1/asset` with advanced filter and sorting options in request body. This will create a session.
To navigate within a search session, make a `HTTP GET` request to the same endpoint with the following query parameters, and you need to send the filtering options.

```
HTTP PATCH v1/vulasset

Request Body
{
    "publishedType": "before",      // all, before, after
    "publishedTime": 1702252800000, // timetick
    "packageType": "withfix",       // all, withfix, withoutfix
    "severityType": "high",         // high, medium
    "scoreType": "v3",              // v2, v3
    "scoreV3Min": 1,
    "scoreV3Max": 4,
    "matchTypeService": "contains", // contains, equals
    "serviceName": "svc",           
    "matchTypeNs": "contains",
    "selectedDomains": [
        "ns1",
        "ns7"
    ],
    "matchTypeImage": "equals",     // contains, equals
    "imageName": "img",            
    "matchTypeNode": "equals",      // contains, equals
    "nodeName": "node",
    "matchTypeContainer": "equals", // contains, equals
    "containerName": "cont",
    "orderbycolumn": "scorev3",
    "orderby": "desc"
}

Response Body
{
    "debug_error": 0,    // 👈 please ignore fields start with "debug".
    "debug_error_message": "",
    "debug_perf_stats": [
        "step-1, get allowed resources, took=792.405µs",
        "step-2, get filtered vulasset from db, took=46.983231ms",
        "step-3, populate result to tmp session table, took=29.151944ms"
    ],
    "query_token": "mm_eff501a8ce17",   👈  // need to bring this value in the URL parameter to navigate this query session
    "summary": {   👈   // this section of data now is a placeholder, it does not contains real data yet..
        "count_distribution": {   1️⃣
            "container": 3,
            "high": 20,
            "image": 8,
            "low": 10,
            "medium": 15,
            "node": 12,
            "platform": 5
        },
        "top_images": [           2️⃣
            {
                "display_name": "Image1",
                "high": 5,
                "id": "0",
                "low": 2,
                "medium": 3
            },
            {
                "display_name": "Image2",
                "high": 4,
                "id": "1",
                "low": 1,
                "medium": 2
            },
            {
                "display_name": "Image3",
                "high": 3,
                "id": "2",
                "low": 1,
                "medium": 2
            },
            {
                "display_name": "Image4",
                "high": 2,
                "id": "3",
                "low": 1,
                "medium": 2
            },
            {
                "display_name": "Image5",
                "high": 3,
                "id": "4",
                "low": 1,
                "medium": 2
            }
        ],
        "top_nodes": [              3️⃣
            {
                "display_name": "Node1",
                "high": 8,
                "id": "0",
                "low": 2,
                "medium": 4
            },
            {
                "display_name": "Node2",
                "high": 6,
                "id": "1",
                "low": 1,
                "medium": 3
            },
            {
                "display_name": "Node3",
                "high": 5,
                "id": "2",
                "low": 1,
                "medium": 3
            },
            {
                "display_name": "Node4",
                "high": 4,
                "id": "3",
                "low": 1,
                "medium": 3
            },
            {
                "display_name": "Node5",
                "high": 3,
                "id": "4",
                "low": 1,
                "medium": 3
            }
        ]
    },
    "total_matched_records": 161,   👈
    "total_records": 279            👈
}
```


```
GET v1/vulasset?token=mm_eff501a8ce17&start=0&row=100

token: Indicates the query session; you can find this token in the response body.
start: Specifies the starting row.
row: Defines the number of rows to fetch. Use -1 to fetch all rows.

Reponse

{
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

## Testing environment
I have set up an environment in the lab at 10.1.45.44. I will update the image with the latest work for testing purposes.


To access the management console, visit https://10.1.45.44:30590/#/login.
If necessary, you can also SSH into the machine to make any required changes.
The controller endpoint is accessible via curl at 10.1.45.44:31693.

You can also use these two scripts on 10.1.45.44 for testing.

```
neuvector@ubuntu2204-E:~/ui_perf$ ./c_new_vulasset_step1_POST.sh
{ ..... "query_token":"mm_ec1ab4a190d0" 👈 ,"summary":{"count_distribution".... ,"total_matched_records":3006,"total_records":3006}

neuvector@ubuntu2204-E:~/ui_perf$ ./d_new_vulasset_step2_GET.sh mm_ec1ab4a190d0 👈 | jq
```
