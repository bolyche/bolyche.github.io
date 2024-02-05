---
layout: post
date: 2023-09-26
title: "Database groupings"
categories: [ shitpost, programming ]
tags: 
description: "Choosing your database"
featured: false
hidden: false
image: assets/images/dbs.jpg
---

There are different ways to classify or group databases together. Some of the common ways include

<style>
table, th, td {
   border: 1px solid #ffffff5e;
   background-color: #a5a5a51f;
   padding: 10px
}
</style>

| Grouping | Types | Description |
|----------|-------|-------------|
| Processing Type | OLAP, OLTP, OLEP | Databases are optimised generally for specific types of workloads, as such this is one of the most important considerations |
| Hosting | On Prem or Cloud | You might already have your backend or data with a particular cloud provider or hosting yourself on your local servers. Not all solutions are available everywhere - AWS for example has it's own databases for which it's optimised |
| DB type | Relational or NoSQL (not only SQL) | This isn't an easy question because NoSQL holds a vast variety of databases to choose from

<br>
But these groupings don't necessarily tell us much.

We might know that we need an OLTP database but the options are from there remain numerous. The same goes for cloud provider and NoSQL vs SQL.

These days there is a wide and interesting variety of databases out there which can actually be _optimised_ for certain use cases - and that is very exciting, it allows us to be more efficient. Some of the types I've put into a table below

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
From the above, it becomes a bit simpler to really just start asking questions in order to figure out what you might need. For example:

- Do you have absolutely ridiculous amounts of incremental timestamp data to record? -> Yes? -> Maybe a time series db is your friend
- Do you frequently query the relationship between multiple independent entities? -> Yes? -> A Graph db may be your friend (Perhaps overlaid over a DB to process large volumes?)
- Do you not know the structure of your data and need to rapidly prototype, but want to be able to parse and access all the key-values quickly? -> Yes? -> Potentially a document DB
- Have you scaled past your PostgreSQL instance and desperately need horizontal scaling? -> Yes? -> Migrating to NewSQL might not be too bad
- Do you need something to keep your web data for eCommerce basket flows? -> Yes? -> Key-value might suit well
- How about a massive amount of concurrent writes are required for your worldwide shipping management? -> Sounds bad? -> Column db might be right here

Ultimately whatever you pick won't necessarily be obvious and may be dictated by things like budget and the existing skillsets of your team. However, picking something unsuited for purpose (e.g. collecting trillions of IoT timestampted data points and expecting Redshift to be efficient with it) might in the end cost you lots of money. You may also find you want more than one solution in order to meet your business needs. 
