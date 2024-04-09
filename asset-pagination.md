## asset pagination support (v5.3.2)

### History

- v1 - 2024/04/08

### background

TODO: list the jira case

Describe the design, similar to vulnerablity api in Risk page.
First we need to create a query session and the backend will return a query token. And we can navigate via the query token.

In initial version, only `type=image` is supported

## Table of Contents

- [Usage for `v1/asset`](#usage-for-v1vulasset)
  - [Starting a Query Session](#starting-a-query-session)
  - [Navigating Within a Query Session](#navigating-within-a-query-session)
  - [Quick Filter Within a Query Session](#quick-filter-within-a-query-session)
- [Testing Environment](#testing-environment)

### Starting a Query Session

To initiate a query, use the POST method on the endpoint `/v1/asset`, providing filters and sorting options within the request body.

**Request Body**  
In initial version 5.3.2 we don't have filter options. So the request body is empty.

```
HTTP POST v1/vulasset

Request Body
{
    type='image'
}
```

**Response Body**

```
{
    "debug_perf_stats": [
      ...
    ],
    "query_token": "b1315231c773",  👈  // need to bring this value in the URL parameter to navigate this query session
    "summary": {
        "top_images": [
            {
                "display_name": "wurstmeister-zookeeper:latest",
                "high": 1385,
                "id": "dc00f1198a444104617989bde31132c22d7527c65e825b9de4bbe6313f22637f",
                "low": 69,
                "medium": 1009
            },
            {
                "display_name": "citadel:1.4.2",
                "high": 1051,
                "id": "9a48168d5ab29a332e14541be713b0be76f330c035f2dfbf115f2583c74edd33",
                "low": 87,
                "medium": 1148
            },
            {
                "display_name": "node-istio:latest",
                "high": 1344,
                "id": "21192ddd0a012fe978d28106c80be49b9a7bfec6c394e7036bbf3ea0286bfd3a",
                "low": 37,
                "medium": 845
            },
            {
                "display_name": "selvam_centos_http:latest",
                "high": 494,
                "id": "2fa01e609a4f5b9ee0b6af7711e582fa0a423c213f5ff9fcd63b5589f3ae8c4d",
                "low": 92,
                "medium": 779
            },
            {
                "display_name": "alpinedev:20230519-110159",
                "high": 111,
                "id": "53b0e93618b3cebff066a17d275b6017b4b9a3514437747326cc35dc4c44f4a7",
                "low": 1,
                "medium": 30
            }
        ]
    },
    "total_records": 4888
}
```

### Navigating Within a Query Session

To navigate within an existing search session, make an `HTTP GET` request to the same endpoint (`/v1/asset`) with the following query parameters.

Refer to the detailed fields and their corresponding values in the following raw data section.

If the user opts for a different column sorting, we can incorporate this change within the current query session by adding the "orderbyColumn" and "orderby" URL parameters, eliminating the need to create a new query session and thereby enhancing performance.

**Request**

```
GET v1/asset?token=eff501a8ce17&start=0&row=100

1️⃣ token: Indicates the query session; you can find this token in the response body.
2️⃣ start: Specifies the starting row.
3️⃣ row: Defines the number of rows to fetch. Use -1 to fetch all rows.
4️⃣ orderbyColumn: Use different column to sort
5️⃣ orderby: Use different sort type
6️⃣ qf: quick filter search term, this will be used to search the CVE Name and score (depends on the scoretype)
```

**Reponse**

```
    {
    "data": [{..},{..}],
    "type": "image"
    }

For type image, the detail field are

    {
      "base_os": "ubuntu:14.04",
      "created_at": "2019-02-26 23:47:49",
      "digest": "sha256:392b2dca75791568c7f240e88212e56127a53de9f885d2a917dd41dad20606f2",
      "high": 1344,
      "id": "21192ddd0a012fe978d28106c80be49b9a7bfec6c394e7036bbf3ea0286bfd3a",
      "low": 37,
      "medium": 845,
      "name": "node-istio:latest",
      "repo_name": "nvbox",
      "repo_url": "https://nvbox.azurecr.io/",
      "scanned_at": "2024-04-09 00:07:12",
      "size": 464185144
    },
```

### Quick Filter Within a Query Session

The current UI design includes a Filter function that enables users to refine their search within the existing results.
To achieve this, you can utilize the same endpoint with an `qf` URL parameter to specify the search term and `scoretype` to indicate score type.

The search scope is currently limited to the [name] and [score] fields.

**Request**

```
GET /v1/vulasset?token=aaa&qf=term&scoretype=v3&start=0&row=100

qf : indicate the quick filter term
scoretype: indicate the score type, values are "v2", "v3"

Reponse
{
"qf_matched_records": 0, // 👈 how many quick filter matched records
"vulnerabilities": [...]
}

```

Quick filter:

<p align="left">
<img src="./materials/asset-pagination-1.png" width="50%">
</p>

## Testing environment

I have set up an environment in the lab at 10.1.45.44. I will update the image with the latest work for testing purposes.

To access the management console, visit https://10.1.45.44:30590/#/login.
If necessary, you can also SSH into the machine to make any required changes.
The controller endpoint is accessible via curl at 10.1.45.44:31693.

You can also use these two scripts on 10.1.45.44 for testing.
