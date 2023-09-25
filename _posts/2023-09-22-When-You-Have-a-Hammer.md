---
layout: post
date: 2023-09-22
title: "When You Have a Hammer..(part 1)"
categories: [ shitpost, programming ]
tags: 
description: "Choosing your database"
featured: true
hidden: true
image: assets/images/servers.jpg
---

## (This is a post on databases)

When you have a hammer.. everything looks like a nail. Similarly, when you know Postgres.. Postgres is good for all problems right? It's like the proverbial porridge in the 3 Bears childrens story. Not too hot, not too cold. Open source, scales well, many integrations, well maintained... What more could you possibly need?

😂

### Considerations in choosing your DB

To be honest I've known various people guilty of simply choosing the tool they're familiar with - from all walks of seniority and field. Sometimes when you need to implement something quickly, it actually really makes a difference to _know_ all the problems, abilities and inabilities of a chosen tool. To be an expert in one of them takes time and experience - not to be taken for granted!

Often however it really pays to consider your use case and to answer questions such as:

- How much data will you be storing?
- How stable (or volatile) is your data - Or will your data and schemas need to change?
- Is it transactional or analytical processing you need - or both?
- How many simultaneous requests do you expect at peak usage?
- What is your database powering - Is it the backend for an application which varies wildly in peak usage? Does it need to be fault tolerant? 
- How does your database need to integrate with other applications or processes?
- Does it need to span across many geographical zones (and so comply with many different data protection rules)?
- What type of data is it? What is it's structure? Does it have relationships?
- What is the setup and maintenance budget?

-------------

### Important terms

There are various terms bandied around when looking at databases so it seems useful to define them or understand a bit more what they mean.

**Latency**:

- The measure of time between the request and response of an application in a network
- When considering end to end latency, this includes the client request, network delays, time used by intermediaries and the database until the response is received
- Latency can increase with increasing throughput due to the database needing more resources to tidy up or due to throttling / bottlenecking elsewhere
- If your database needs to have low latency (e.g. an eCommerce business) then you'll need to understand what this looks like then the throughout is high (and your business is booming!) as often at low throughput systems can easily give low latency.. but this may not scale

**Throughput**:

- Typically the transactions processed per minute (TPM) or per second (TPS)
- Can also be considered in terms of the number of simultaneous requests the system is capable of processing
- Some systems like Cassandra can have a very high throughput whilst having a less optimal latency due to ability to process many concurrent requests per machine, plus the distributed nature allowing high horizontal scaling
- When considering throughput, it's important to consider what you expect your system to handle. Databases set up for analytical processing typically don't need high throughput for example.

**ACID**:

ACID compliance should be used when you need your transactions to be processed reliably and guarantee valid data. Standing for 
- Atomicity: A transaction either succeeds entirely for the database or fails with no intermediate state; preventing partial updates in the case of errors, node crashes, power failures or other
- Consistency: Transactions are consistent with database constraints; if the data is invalid or is in an illegal state, the transaction should fail
- Isoltation: Transactions are isolated and don't interfere with each other
- Durability: Transactions that complete persist even in the case of post-transaction errors, node crashes, power failures or other 

It's worth noting that it's not just RDBMS' that are ACID-compliant these days. NoSQL databases such as MongoDB and DynamoDB can also support ACID compliance.


**Vertical scaling**:

- This involves changing your databases resources such as CPU, memory, storage, and potentially networking capacity
- It's generally easy to do and doesn't add complexity
- There is a limit to how much you can scale this way, since there's a physical limit to how much CPU, memory, harddrives and network interfaces can be present on one machine
- Scaling this way may result in downtime if it requires migration between hardware

**Horizontal scaling**:

- This involves adding machines / nodes to your system
- The data can be partitioned or sharded between machines (MongoDB does this) or replicated across many machines (AWS RDS does this for systems such as Postgres)
- It can be complex to manage and maintain
- Sharded databases will be effected by how the data is queried and stored; the way your systems interact with the database would change

**CAP**: 

- This refers to a theory originally proposed by Professor Eric A. Brewer about distributed systems 23 odd years ago
- The acronym represents Consistency, Availability and Partition Tolerance
- It posits that you have to compromise between Availability and Consistency in a distributed data store, with Availability defined as "every request gets a response" and Consistency as "Every read gets the most recent write"
- These two sound and indeed are mutually exclusive, since when one of your partitions fails, you either have to cancel the operation (losing availability) or continue the operation and have a momentary inconsistency; You can't have a guarantee in both


### Decisions

Wouldn't it be grand if there was an up to date flow diagram for how to make such decisions?

In my next post I'll outline various databases of interest (anyone want a time series db?) and mention some interesting benchmarking data I've seen. 