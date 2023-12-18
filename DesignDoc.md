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

<b>Process</b> This phase is triggered when the UI requests page data. Given its sensitivity and criticality to performance, optimization efforts are concentrated in this phase. The primary objective here is to compile data specific to the query, considering variations due to user roles, filter criteria, and time. Upon completion, a temporary table is generated to store the results, and a query_token is returned to the caller. This token allows the UI to fetch real data and navigate through the results.

<b>Post-process</b> This phase is activated when the UI fetches the data. To minimize tasks during the process phase, certain operations are intentionally deferred to this stage. These deferred tasks are executed when the user expresses the need to view the data.
