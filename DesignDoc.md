# UI Vulnerablity Page Improvement

## Introduction

Briefly introduce the purpose and scope of the technical document.

## Table of Contents

- [Section 1: Overview](#section-1-overview)
- [Section 2: Installation](#section-2-architecture)
- [Section 3: Usage](#section-3-usage)
- [Section 4: Examples](#section-4-examples)
- [Section 5: Troubleshooting](#section-5-troubleshooting)
- [Section 6: Conclusion](#section-6-conclusion)

## Section 1: Overview

The primary objective is to enhance the performance of the Vulnerability Page, focusing on:
- Load time optimization
- Reducing memory consumption to mitigate potential out-of-memory errors in browsers.

Originally, the plan involved integrating memory reduction measures in both the Consul and Controller processes. However, after thorough exploration, we have not identified an optimal solution considering the deployment model and data synchronization. Consequently, this aspect will be excluded from the current release and is planned to be deferred to the subsequent version.

## Section 2: Architecture

Our system utilizes SQLite3 as its embedded database, with each controller maintaining an database. No supplementary processes are introduced, and SQLite access is facilitated through an imported package. 

SQLite3 enhances our data querying capabilities, offers a file-based structure for simplified management, and contributes to system efficiency with its lightweight design.

### data handling process

The entire data handling process is divided into three distinct phases based on timing: 
- 1️⃣ pre-process
- 2️⃣ process
- 3️⃣ post-process

<b>Pre-process</b> This phase initiates when raw data becomes available, typically following the completion of a scan report. During this stage, the data is populated into the database.

<b>Process</b> This phase is triggered when the UI requests page data. The primary objective here is to compile data specific to the query, considering variations due to user roles, filter criteria, and time. Upon completion, a temporary table is generated to store the results, and a query_token is returned to the caller. This token allows the UI to fetch data and navigate through the results.

<b>Post-process</b> This phase is activated when the UI fetches the data. To minimize tasks during the process phase, certain operations are intentionally deferred to this stage. These deferred tasks are executed when the user expresses the need to view the data.

### data hook point

The data hook point is initiated upon completion of the scan task (scanDone()). Raw data obtained from the scan is processed through functions like FillVulTraits() function to generate a comprehensive report. Subsequently, the details of this report are stored in the database.

At this stage, an existing mechanism, the Consul KV watcher, is employed. This watcher enables the system to detect new data additions or restorations, triggering the initiation of a cache-building process. The newly acquired data is subjected to ETL operations before being saved to the database.

### database design

The database table is crafted to optimize the Vulnerability Page. This page necessitates the incorporation of two distinct aspects of data: vulnerability-based and asset-based information.

### SQL
TODO

### multi-controller

Given that each controller operates independently and the database (it's embedded to the Controller process) is not shared, an essential mechanism is required to enable other controllers to construct the same session temporary table. To achieve this, a request containing user roles, advanced filters, and query_token is written to Consul. This action serves as a signal to inform other controllers. Subsequently, these controllers can utilize the provided query_token to serve requests at a later stage.

### security (SQL Injection prevention)

TODO: list the file, no additional resource required in current k8s manifest.

The code uses parameterized queries, also known as prepared statements, as a best practice for writing SQL queries. This approach treats user input and other variables as parameters rather than integral parts of the SQL statement. By doing so, the system mitigates the risk of SQL injection attacks and ensures a more secure interaction with the database.

### session temp table cleanup

Given the dynamic nature of query results, the system employs session temporary tables for storage. Typically, a new session is unnecessary when users perform subsequent queries, such as changing filter criteria. To streamline resource usage, a maximum of 5 queries per user/apikey is enforced. This limitation ensures that older sessions, which are no longer needed, are systematically cleaned up. 

### session temp table 

To optimize performance during the process phase, the system employs a strategic approach. The session temporary table is initially written to a memory-based database to promptly fulfill the first initial request. Concurrently, in the background, a file-based database is created. Once the file-based table has been successfully created, the memory-based table is systematically deleted. This dual-step process effectively balances the imperative for rapid response times.