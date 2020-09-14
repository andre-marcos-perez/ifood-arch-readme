# 2 - Architecture

| Previous                              | Current          | Next                                  | Beginning              |
| ------------------------------------- | ---------------- | ------------------------------------- | ---------------------- |
| [1 - Introduction](1-Introduction.md) | 2 - Architecture | [3 - Applications](3-Applications.md) | [Readme](../README.md) |

---

### Contents

- [2.1 - Overview](#21---overview)
- [2.2 - Orchestration Layer](#22---orchestration-layer)
- [2.3 - Stage #1: Raw Processing](#23---stage-1-raw-processing)
- [2.4 - Stage #2: Trusted Processing](#24---stage-2-trusted-processing)
- [2.5 - Stage #3: Measure Processing](#25---stage-3-measure-processing)
- [2.6 - Visualization Layer](#26---visualization-layer)

---

### <a></a>2.1 - Overview

![ifood-arch](../media/ifood-arch.png)

### <a></a>2.2 - Orchestration Layer

The orchestration layer is responsible for managing the entire ETL process. The **AWS CloudWatch** starts **AWS Step 
Functions** using a cron trigger that runs every day at **03:00 am BRT** when all restaurants are closed, supposedly. 
The **AWS Step Functions** runs the *ifood-arch-orchestrator* flow that performs the following actions:

1. Creates an **AWS EMR** cluster;
2. Runs the first stage to bring data from the staging to the data lake;
3. Runs the second stage to bring data from the data lake to the data warehouse;
4. Terminates the **AWS EMR** cluster; 
5. Runs the last stage yo measure the finished ETL process.

Each step is described in details on their own topic. The **AWS Step Functions** *ifood-arch-orchestrator* flow definition
(both script and visual layout) can be found [here](https://github.com/andre-marcos-perez/ifood-arch-infra/tree/master/orchestration/aws-step-functions).

- Related projects:

> **[iFood Arch Infra](https://github.com/andre-marcos-perez/ifood-arch-infra)** - Application to centre infrastructure related scripts

### <a></a>2.3 - Stage #1: Raw Processing

The first stage is responsible for processing the source data into raw data. The **AWS Lambda** copies the raw data from 
**AWS s3** source bucket *ifood-data-architect-test-source* to the **AWS s3** staging bucket *ifood-arch-raw-staging* 
using a threading approach. It also connects to the **AWS RDS** *ifood-arch-oltp-db* database to register the process 
(start time, end time, etc.). The orchestrator then starts multiple **AWS EMR** jobs in parallel to process the raw data. 
It reads data from **AWS s3** staging bucket *ifood-arch-raw-staging*, enrich the data and persists it to **AWS s3** raw 
bucket *ifood-arch-raw*. At the end, **AWS Athena** is plugged into **AWS s3** raw bucket *ifood-arch-raw* to allow further 
*ad hoc* queries.

- Related projects:

> **[iFood Arch Lambda ETL](https://github.com/andre-marcos-perez/ifood-arch-lambda-etl)** - Serverless application to inexpensively perform orchestration tasks

> **[iFood Arch EMR ETL](https://github.com/andre-marcos-perez/ifood-arch-emr-etl)** - Serverless application to perform high volume ETL tasks

> **[iFood Arch Infra](https://github.com/andre-marcos-perez/ifood-arch-infra)** - Application to centre infrastructure related scripts

### <a></a>2.4 - Stage #2: Trusted Processing

The second stage is responsible for processing the raw data into trusted data. The orchestrator starts **AWS Lambda** to 
connect to the **AWS RDS** *ifood-arch-oltp-db* database to register the process (start time, end time, etc.) and starts 
multiple *AWS EMR* in parallel to enrich data from **AWS s3** raw bucket *ifood-arch-raw* to the **AWS s3** trusted bucket 
*ifood-arch-trusted*. At the end, **AWS Redshift** is plugged into **AWS s3** trusted bucket *ifood-arch-trusted* using 
**AWS Redshift Spectrum** to allow further analytics queries. This setup essentially turns **AWS Redshift** into a 
caching layer for **AWS s3**, slightly reducing its performance but greatly saving costs.

- Related projects:

> **[iFood Arch Lambda ETL](https://github.com/andre-marcos-perez/ifood-arch-lambda-etl)** - Serverless application to inexpensively perform orchestration tasks

> **[iFood Arch EMR ETL](https://github.com/andre-marcos-perez/ifood-arch-emr-etl)** - Serverless application to perform high volume ETL tasks

> **[iFood Arch Infra](https://github.com/andre-marcos-perez/ifood-arch-infra)** - Application to centre infrastructure related scripts

### <a></a>2.5 - Stage #3: Measure Processing

The third and last stage is responsible for measuring the ETL process. The **AWS Lambda** extracts ETL process metrics 
from **AWS RDS** *ifood-arch-oltp-db* database and loads into **AWS RDS** *ifood-arch-olap-db* database using dimensional 
modelling techniques to ease the usage of analytical queries to extract runtime metrics for dashboards. Thus adding a 
layer of observability to the pipeline.

- Related projects:

> **[iFood Arch Lambda ETL](https://github.com/andre-marcos-perez/ifood-arch-lambda-etl)** - Serverless application to inexpensively perform orchestration tasks

> **[iFood Arch Infra](https://github.com/andre-marcos-perez/ifood-arch-infra)** - Application to centre infrastructure related scripts

### <a></a>2.6 - Visualization Layer

The visualization layer adds visibility to the pipeline. The **AWS Athena** is plugged into the **AWS s3** *ifood-arch-raw*
bucket to allow on demand *ad hoc* queries. The **AWS Redshift Spectrum** is plugged into the **AWS s3** *ifood-arch-trusted* 
bucket to allow on demand analytics queries. It is proposed (but not implemented) the visualization software **[Redash](https://redash.io/)**
to allow **AWS Athena** and **AWS Redshift** queries for external users plus a dashboard built upon the **AWS RDS** 
*ifood-arch-olap-db* database.

- Related projects:

> **[iFood Arch Infra](https://github.com/andre-marcos-perez/ifood-arch-infra)** - Application to centre infrastructure related scripts