---
layout: post
date: 2023-08-25
title: "Thoughts on DataOps"
categories: [ shitpost ]
tags: 
description: "DataOps"
featured: true
hidden: true
image: assets/images/devops.jpg
---

I had a look around the other day for any new and interesting tooling used in the DataOps sphere. It seemed a bit weird to me that though a lot of sites agree on what DataOps _is_ they don't seem to call out the right tools in order to implement it well?

My experience of doing some DataOps pretty much boils down to believing the following things are most important:
- Ensure everything is in version control
- Ensure everything has a lifeycle policy
- Ensure everything runs on anyone's machine / in cloud with dependencies fully managed
- Ensure your infrastructure is IaC managed
- Ensure scalability
- Ensure everything is reproducible (this is where uni-directional data flows are _essential_ especially for raw data -> this is why I like data flow diagrams, because it's great to understand data movement/transforms but also the access policies involved in those)
- Ensure everything is _tested_ and ensure code isn't merged unless it's got tests (This relies on your engineers being good engineers)
- Have your CI/CD pipeline running and merging into production on successes (ideally with integration tests)
- Have your permissions clearly managed for dev vs prod - imho nobody should have access to write or delete to production - except the prod pipeline itself - More on that later
- Test or ad-hoc areas per the lifecycle policy should have an auto-delete scheduled after a period of disuse
- Your pipelines and processes should have logging - ELK stack is lovely for this
- Your pipelines themselves must be scalable and retryable
- Monitoring and alerting should be in place - Grafana is great for this - Prometheus is lovely if you're using python
- Data quality / integrity checks should be automated - and frankly imo should be done thoroughly on incoming data (to immediately flag problems, rather than after all the processing(!))

Now if we take these bundle of things and we look at the tooling, this is what I think of:
- Github (or ok something else if you like)
- Docker
- Kubernetes
- Terraform (also manage permissions)
- CI/CD tool like Jenkins, CircleCI or Github Actions (though the latter for me was terrible: it absolutely eats money)
- ELK or even just Elasticsearch and Kibana
- Throw in some Prometheus and Grafana too
- Pipelining - Apache Airflow (or Dagster, Luigi etc)

Everything outside of that for me is icing. The data quality checks can depend on the system but also there's fancy tools out there you can attach into your setup. There's loads of companies doing more fancy stuff with AI which I'm not going to get into. 

But really, a large part of DataOps is in having transparency and visibility into all the operations you're running and into the current status of any pipeline or entity. It means knowing the integrity of what you have and ensuring traceability for any operation or change undertaken. This means, nobody should write to production ;)

NB. If you forsee the need to (or currently have it!) to edit prod data.. then that transform needs to be codified as a step in your processing pipeline. There's nothing worse than someone editing some data / forgotten table dependencies / recreation of the data failing..
