---
layout: post
date: 2023-09-26
title: "(WIP) Database Summaries"
categories: [ programming ]
tags: 
description: "Database summaries / cheatsheet"
featured: true
hidden: true
image: assets/images/db_summaries.jpg
---

I plan to add to this post - absolutely not comprehensive, rather a few notes on dbs / warehousing solutions I've used. 
Also given the pace of deleveopment at places like Snowflake, there's a distinct risk this page will be rapidly out of date 😂

**Snowflake**

- Snowflake is a data warehousing solution with a very pleasing UI
- It can sit over different cloud providers
- Separate storage and compute
- Warehouse (compute) management can be problematic to understand query loading, but warehouse auto-timeouts are effective
- Rich SQL, though no support (still?) for regex lookarounds or dynamic SQL
- Can enable row, table, schema, db level security with hierarchical roles
- Can use table constraints though these are largely for indicative purposes
- Can use snowpipe + streams and tasks for data streams, though there is also the new Snowflake Streaming, enabling direct streaming into the DWH from sources such as Kafka
- Time Travel & FailSafe allow for checking or restoring historical data at the time it was queried
- Functions can be written in a few languages (python, javascript) and are easy to utilise, as are stored procedures; Unfortunately no triggers (when I last checked)
- Dyanmic data masking to protect PII data is now a feature, but no clear integrations to auto-recognise problematic columns
- Great python / snowpark features for data science


**BigQuery**

- Google Cloud Platform data warehousing solution
- Optimal for OLAP on a larger scale
- Optimised for denormalised or nested datasets
- Works well on wide data; also well with long data which can be sharded or partitioned if needed (with auto-sharding on date possible)
- Slow for joins and when using multiple tables
- Inserts / deletes will operate on a similar timeframe for a few rows vs a few thousand
- Can stream insert data via streaming API or use batching to speed up ingests
- Caches query results
- Fully managed service; storage and compute scale well without interference
- Great integrations with services like Firebase, Google Analytics, Looker, Dataproc, Tableau
- Exists in the GCP space of numerous integration connectors (Kafka, CouchDB, Redis, Redshift, etc)
- Costing can be more tricky than in other systems


**Redshift**

- Amazon Web Services column oriented data warehousing solution
- Can hold up to 16 PB of data on a cluster
- Redshift spectrum allows simultaneous querying of structured and semistructured s3 data without loading directly into the cluster; equally data can be quickly loaded from DynamoDB or EMR
- Downside of Spectrum is that it charges per scanned byte
- Slower JDBC loads from other sources
- Not really optimised for real-time; (can ingest streaming data via Kinesis and use materialised views which update); better for batching and hot/cold stores
- Table Sort and Dist keys should be optimised (not automatically managed) which can be both a plus and a minus as engineers will need to know how the data will be used
- Analyse and Explain commands can be extremely helpful for understanding the query performance, query plan and to plan the query queue; but the workload management (WLM) can be difficult
- Problematic re concurrant usage; multitenancy access may limit the system, serialisation errors can be a problem
- No checks for primary key constraints (when I used it)
- Automatic re-sorting of table rows and reclaiming of space (vacuuming) is a plus
- Materialised views appear to refresh on auto-vaccuuming which increases cost - is a minus
- Dynamic data masking to protect PII data
- Potential to integrate with AWS tooling to e.g. use Amazon Comprehend to recognise PII columns


**Elasticsearch**

- Distributed near-realtime JSON document search - can act as a NoSQL db but lacks distributed transactions
- Shards can be replicable, with each node in a cluster able to host multiple shards whilst coordinating operations
- Automatic rebalancing and rerouting is a plus, as is the effectiveness of their indexes
- Kibana (often used with) effectively visualises the data
- Multiple APIs available in difference languages such as PHP, Ruby, python
- Can have problematic cluster restructuring or upgrading; equally large data volumes may need to be archived.. and restoring such indexes is a tricky procedure
- Not easy to restucture or change your index
- Not suited to relational data that will require joins between documents
- Further info: [this is a good (though maybe old) link on it](https://www.datadoghq.com/blog/monitor-elasticsearch-performance-metrics/)
