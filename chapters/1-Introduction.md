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

The proposed solution is a cloud native architecture built on **[Amazon Web Services](https://aws.amazon.com/pt/)**. It 
leverages **AWS CloudWatch** and **AWS Step Functions** for orchestration, pairs **AWS Lambda** and **AWS EMR** as 
computing units, uses **AWS RDS** and **AWS s3** as persistence layers and **AWS Athena** and **AWS Redshift** as query
engines. The solution focus more on low cost execution rather than performance and seeks to fill business need by 
leveraging data partition.   

![ifood-arch](../media/ifood-arch.png)

### <a></a>1.2 - Features

- **Business**

1. The ETL pipeline is configured to run daily at 03:00 am BRT when restaurants are closed;
2. The data data is always partitioned by geographical features (country and state) to favour regional analysis and 
handle skewed data.

- **Cost**

1. Source data is staged to avoid external access in the case of failures, mitigating data transfer in costs;
2. The **AWS EMR** cluster infrastructure provisions on-spot machines that are up to 90% cheaper than on demand-machines;
3. The ETL computing workload is split between **AWS EMR** and **AWS Lambda** computing units, letting orchestration 
tasks to be performed by the **AWS Lambda** (cheaper) and heavy computing tasks to be performed by **AWS EMR** (more expensive);
4. The data in the data lake is partitioned by geographical features and timestamp to optimize **AWS Athena** queries;
5. The data in the data warehouse is persisted on the **AWS s3** to optimize **AWS Redshift** costs through **AWS Redshift Spectrum**,
essentially turning the service into a caching layer for analytical queries.

- **Security**

1. The pipeline users are segmented in data engineers and data scientists and the **[principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)** 
is applied to the data layers, i.e., data engineers have full access while data analysts and data scientists have only 
access to the data warehouse (trusted layer);
2. The data in the data warehouse is cleaned and masked to be compliant with the **[LGPD](https://pt.wikipedia.org/wiki/Lei_Geral_de_Prote%C3%A7%C3%A3o_de_Dados_Pessoais)**.

