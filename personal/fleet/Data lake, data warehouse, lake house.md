
## Data lake
First generation analytic storage platform
chứa unstructured data
archived data
cheap storage cost


## Data warehouse
Sync from data lake using ETL into structured/schema database with good analytic performance => resource redundancy

Second generation analytic storage platform

## Data warehouse decoupling storage with computation (snowflake)

Some limitation (TODO)???
Building An Elastic Query Engine on Disaggregated Storage: https://15721.courses.cs.cmu.edu/spring2023/papers/02-modern/vuppalapati-nsdi22.pdf

## Lake house
Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics: https://15721.courses.cs.cmu.edu/spring2023/papers/02-modern/armbrust-cidr21.pdf
Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores: https://www.vldb.org/pvldb/vol13/p3411-armbrust.pdf

Storage layer on top of cheap platform like s3/gcs with the usage of delta engine
still parquet file storing historical data

Support ACID

Use case for low write through put