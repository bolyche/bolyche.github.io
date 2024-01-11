---
layout: post
date: 2023-09-26
title: "Database groupings"
categories: [ shitpost, programming ]
tags: 
description: "Choosing your database"
featured: true
hidden: true
image: assets/images/dbs.jpg
---

There are different ways to classify or group databases together, for example:

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
| DB type | Relational no NoSQL (not only SQL) | This isn't an easy question because NoSQL holds a vast variety of databases to choose from

<br>
But these groupings don't necessarily tell us much.

We might know that we need an OLTP database but the options are from there remain numerous. The same goes for cloud provider and NoSQL vs SQL.

When you try to answer this question online often you'll get the "SQL vs NoSQL debate". To me, this feels like a slightly outdated way to consider databases.

These days there is a wide and interesting variety of databases out there which can actually be _optimised_ for certain use cases - and that is very exciting.

---------------

### Database Types

Some popular types include:

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
From the above, it becomes a bit simpler to make some small generalisation. For example:

- Do you have absolutely ridiculous amounts of incremental timestamp data to record? -> Yes? -> Maybe a time series db is your friend
- Do you frequently query the relationship between multiple independent entities? -> Yes? -> Maybe a nice graph DB? (Perhaps overlaid over a DB to process large volumes?)
- Do you not know the structure of your data and need to rapidly prototype, but want to be able to parse and access all the key-values quickly? -> Yes? -> Okay so maybe a document DB?
- Have you scaled past your PostgreSQL instance and desperately need horizontal scaling? -> Yes? -> Migrating to NewSQL might not be too bad
- Do you need something to keep your web data for eCommerce basket flows? -> Yes? -> A beautiful key-value db for you then good Sir
- How about a massive amount of concurrent writes are required for your worldwide shipping management? -> Sounds bad? -> It'll probably be okay with a nice column db

I joke only a little here, because obviously the solution is not .. well, obvious. You'll likely find you need multiple solutions also to meet your business need. 
