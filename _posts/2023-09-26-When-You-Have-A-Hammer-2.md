---
layout: post
date: 2023-09-26
title: "When You Have a Hammer..(part 2)"
categories: [ shitpost, programming ]
tags: 
description: "Choosing your database"
featured: true
hidden: true
image: assets/images/dbs.jpg
---

Continuing from the Part 1 post - an overview of database types, use cases, & notes on a few systems (DWHs mostly) I've used myself.

### Grouping Databases

There are different ways to classify or group databases together. 

The following table shows some of the groupings

<style>
table, th, td {
   border: 2px solid white;
   background-color: #a5a5a5a8;
}
</style>

| Grouping | Types | Description |
|----------|-------|-------------|
| Processing Type | OLAP, OLTP, OLEP | Databases are optimised generally for specific types of workloads, as such this is one of the most important considerations |
| Hosting | On Prem or Cloud | You might already have your backend or data with a particular cloud provider or hosting yourself on your local servers. Not all solutions are available everywhere - AWS for example has it's own databases for which it's optimised |
| DB type | Relational no NoSQL (not only SQL) | This isn't an easy question because NoSQL holds a vast variety of databases to choose from. As such I'll add a small discussion of it below

<br>
But this doesn't necessarily tell us much. 

Perhaps we know based on our requirements that we need an OLTP database. There's still many to choose from. 
Similarly, if we choose a particular cloud provider (say, AWS), then we still have a wide choice of what to deploy unless we want to go for a fully managed database option.
Similarly again, if we decide we want an OLTP database which resides in AWS - Have our options narrowed down by much? Quite a few NoSQL databases as well as relational databases are ACID compliant.


When you investigate this question online often you'll get the "SQL vs NoSQL debate" happened. Something about that strikes me as off though. 

These days, there is a wide and interesting variety of dbs out there which can actually be _optimised_ for certain use cases - and that is very exciting.

---------------

### Database Types

When considering the database type, it can almost be simpler to consider the different _kinds_ of databases rather than SQL vs Non. 

Here are some of the more popular types:

| Database | Use case | Examples | Notes |
|:---------|:---------|:---------|:------|
| Relational | eCommerce, financial transactions, CRMs, PRMs | MariaDB, PostgreSQL, SQLite, Redshift, MySQL | Generally a great solution for applications generating structured, relational data |
| Key-value | Caching (cookies), catalogs, gaming scoring, session management, ecommerce systems | DynamoDB, Redis, Memcached | Very lightweight, great for in-memory storage in sites on apps |
| Document | Formats like JSON/BSON/XML, used for rapid development, content management, user profiles | MongoDB | Data queried together is often stored in the same document. Highly flexible, fast filtering of documents based on attributes, easy to change documents |
| Column family | Network traffic analysis, large scale scientific data, search, wide scale trade management | Cassandra, BigTable | Good for big data distributed applications that require many concurrent writes & can tolerate short inconsistency |
| Graph | Social networks, network management, product and service recommendations, fraud detection | Neo4J, Neptune, OrientDB | Establishes relationships between entities using nodes, edges and properties - Useful when you want to establish relationships between different users or systems, which in a traditional database would require many joins |
| Time series | IoT, weather monitoring stations, Industrial telemetry, application logging or system performance | Prometheus, TimescaleDB | Heavily write-oriented for historical data keeping |
| NewSQL | Similar to Relational but where horizontal scalability is required | CockroachDB, Couchbase | Typically CP (out of CAP), losing availability. Less flexible than popular relational dbs |

<br>
We can start to guess at a few easy things from this. 

- Do you have absolutely ridiculous amounts of incremental timestamp data to record? -> Yes? -> Maybe a time series db is your friend
- Do you frequently query the relationship between multiple independent entities? -> Yes? -> Maybe a nice graph DB? (Perhaps overlaid over a DB to process large volumes?)
- Do you not know the structure of your data and need to rapidly prototype, but want to be able to parse and access all the key-values quickly? -> Yes? -> Okay so maybe a document DB?
- Have you scaled past your PostgreSQL instance and desperately need horizontal scaling? -> Yes? -> Migrating to NewSQL might not be too bad
- Do you need something to keep your web data for eCommerce basket flows? -> Yes? -> A beautiful key-value db for you then good Sir
- How about a massive amount of concurrent writes are required for your worldwide shipping management? -> Sounds bad? -> It'll probably be okay with a nice column db

I joke only a little here, because obviously the solution is not .. well, obvious. You'll likely find you need multiple solutions also to meet your business need. 



---------------

### A few I'm familiar with 

I might expand this post or even move it to another page, but for the present, here's a small overview from me, for some of the systems I've used

**Snowflake**

- Snowflake is a data warehousing solution with a very pleasing UI
- It can sit over different cloud providers
- Separate storage and compute
- Warehouse (compute) management can be problematic to understand query loading, but warehouse auto-timeouts are effective
- Rich SQL, though no support (still?) for regex lookarounds or dyanmic SQL
- Can enable row, table, schema, db level security with hierarchical roles
- Can use table constraints though these are largely for indicative purposes
- Can use snowpipe + streams and tasks for data streams, though there is also the new Snowflake Streaming, enabling direct streaming into the DWH from sources such as Kafka
- Time Travel & FailSafe allow for checking or restoring historical data at the time it was queried
- Functions can be written in a few languages (python, javascript) and are easy to utilise, as are stored procedures; Unfortunately no triggers
- Dyanmic data masking to protect PII data is now a feature, but no clear integrations to auto-recognise problematic columns
- Great python / snowpark features for data science


**BigQuery**

- Google Cloud Platform data warehousing solution
- Optimal for OLAP on a larger scale
- Optimised for denormalised (nested) datasets
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
- Downside of Spectrum is that is charges per scanned byte
- Slower JDBC loads from other sources
- Not really optimised for real-time; (can ingest streaming data via Kinesis and use materialised views which update); better for batching and hot/cold stores
- Table Sort and Dist keys should be optimised (not automatically managed) which can be both a plus and a minus as engineers will need to know how the data will be used
- Analyse and Explain commands can be extremely helpful for understanding the query performance, query plan and to plan the query queue; but the workload management (WLM) can be difficult
- Problematic re concurrant usage; multitenancy access may limit the system, serialisation errors can be a problem
- No checks for primary key constraints
- Automatic re-sorting of table rows and reclaiming of space
- Materialised views appear to refresh on auto-vaccuuming which increases cost
- Dynamic data masking to protect PII data
- Potential to integrate with AWS tooling to e.g. use Amazon Comprehend to recognise PII columns


**Elasticsearch**

- Distributed near-realtime JSON document search - can act as a NoSQL db but lacks distributed transactions
- Shards can be replicable, with each node in a cluster able to host multiple shards whilst coordinating operations
- Automatic rebalancing and rerouting is a plus, as is the effectiveness of their indexes
- Kibana (often used with) effectively visualises the data
- Multiple APIs available in difference languages such as PHP, Ruby, python
- Can have problematic cluster restructuring or upgrading; equally large data volumes may need to be archived and restoring such indexes is a tricky procedure
- Not easy to restucture or change your index
- Not suited to relational data that will require joins between documents

Further info: [this is a good (though maybe old) link on it](https://www.datadoghq.com/blog/monitor-elasticsearch-performance-metrics/)
