# 4 - Data Model

| Previous                              | Current        | Next | Beginning              |
| ------------------------------------- | -------------- | ---- | ---------------------- |
| [3 - Applications](3-Applications.md) | 4 - Data Model | ---  | [Readme](../README.md) |

---

## Contents

- [4.1 - OLTP Database](#41---oltp-database)
- [4.2 - OLAP Database](#42---olap-database)
- [4.3 - Raw Data Layer](#43---raw-data-layer)
- [4.4 - Trusted Data Layer](#44---trusted-data-layer)

---

### <a></a>4.1 - OLTP Database

The OLTP *ifood-arch-oltp-db* database follows a traditional entity-relationship modelling optimized for data size. It 
is composed by an append only *tb_process* and two normalized auxiliary tables: *tb_job* and *tb_status*. Its query 
definition and visual schema can be found **[here](https://github.com/andre-marcos-perez/ifood-arch-infra/tree/master/database/aws-rds)**.

### <a></a>4.2 - OLAP Database

The OLAP *ifood-arch-olap-db* database follows a dimensional modelling optimized for query speed. It is compose by three
dimension tables (*dim_date*, *dim_status* and *dim_job*) and one fact table (*fact_process*). The fact granularity is 
second and its numerical attribute is the amount of time in seconds (*duration_in_seconds*) a ETL process took to run. 
Its query definition and visual schema can be found **[here](https://github.com/andre-marcos-perez/ifood-arch-infra/tree/master/database/aws-rds)**.

### <a></a>4.3 - Raw Data Layer

The raw data layer is the pipeline data lake. Its data is a copy of the original data persisted in the Parquet format in
a partitioned fashion. The general partition strategy is state as bellow. The strategy allows favours geographical analysis 
to be efficiently performed since the consumer behaviour can vary between regions. The timestamp partition avoids 
shuffling to much data in **AWS Athena** *ad hoc* queries, speeding up the analysis and reducing costs.

1. Time based data such as *raw order* and *raw order statuses* are partitioned by an artificial timestamp (YYYY-MM-DD) 
built upon their respective *created at* column;
2. Geographical based data such as *raw order*, *raw consumer* and *raw restaurant* are partitioned by geographical 
features: country and state.

The partitions by dataset is stated bellow, checkout [here](https://github.com/andre-marcos-perez/ifood-arch-emr-etl/blob/master/docs/jobs.md) 
to know more.

- **Raw Order**: delivery_address_country, delivery_address_state and timestamp;
- **Raw Order Statuses**: timestamp;
- **Raw Consumer**: customer_phone_area;
- **Raw Restaurant**: merchant_country and merchant_state.

### <a></a>4.4 - Trusted Data Layer

The trusted data layer is the pipeline data warehouse. Its data is a copy of the original data persisted in the Parquet 
format in a partitioned fashion. The strategy allows favours geographical analysis to be efficiently performed since the 
consumer behaviour can vary between regions. The timestamp partition avoids shuffling to much data in **AWS Redshift** 
analytical queries, speeding up the analysis and reducing costs.

1. Time based data such as *trusted items*, *trusted statuses* and *trusted orders* are partitioned by an artificial 
timestamp (YYYY-MM-DD) built upon their respective *created at* column;
2. Geographical based data such as *trusted order* is partitioned by geographical features: country and state.

The partitions by dataset is stated bellow, checkout [here](https://github.com/andre-marcos-perez/ifood-arch-emr-etl/blob/master/docs/jobs.md) 
to know more.

- **Trusted Items**: timestamp;
- **Trusted Orders**: delivery_address_country, delivery_address_state and timestamp;
- **Trusted Statuses**: timestamp.