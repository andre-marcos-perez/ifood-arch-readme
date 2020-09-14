# 1 - Introduction

| Previous | Current          | Next                                  | Beginning              |
| -------- | ---------------- | ------------------------------------- | ---------------------- |
| ---      | 1 - Introduction | [2 - Architecture](2-Architecture.md) | [Readme](../README.md) |

---

### Contents

- [1.1 - Overview](#11---overview)
- [1.2 - Features](#12---features)

---

### <a></a>1.1 - Overview

### <a></a>1.2 - Features

- **Cost**

1. The **AWS EMR** cluster infra provisions on-spot machines that are up to 90% cheaper than on demand machines;
2. The ETL computing workload is split between **AWS EMR** and **AWS Lambda** computing units, letting orchestration 
tasks to be performed by the **AWS Lambda** (cheaper) and heavy computing tasks to be performed by **AWS EMR** (more expensive);
3. The data in the data lake is partitioned by geographical features and timestamp to optimize **AWS Athena** queries;
4. The data in the data warehouse is persisted on the **AWS s3** to optimize **AWS Redshift** costs, through **AWS Redshift Spectrum**,
essentially turning the service into a caching layer for analytical queries.

- **Security**

- **LGPD**

