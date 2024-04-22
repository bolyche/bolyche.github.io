---
layout: post
date: 2024-04-18
title: "Terraform's Immutability"
categories: [ python, programming ]
tags: 
description: "Terraform 101 vs immutability"
featured: true
hidden: true
image: assets/images/terraform_mars.jpg
---

I recently did a teaching session on terraform at work and it was a good reminder of how excellent the hashicorp documentation is. 

Sometimes I see things like duplicated code or lack of variables or just more often, extremely big long-running dependency graphs being built on running `terraform plan` and I am sorely tempted to send them page 1 of the terraform docs :)

In planning for this teaching session I did a lot of browsing. Discovered that the CTO (back when he used to do lots of videos, I think he stopped about 4 years ago) is remarkably good at explaining concepts. There's one video which comes to mind: explaining what Immutable vs Mutable means in terms of terraform configurations.

I found it interesting because he was suggesting that terraform tends towards immutable architecture. To explain his concept a little (without you needing to watch the video yourself):
* Mutable infra is like upgrading nginx on an EC2 instance used as a webserver (using some configuration manager like chef, puppet or ansible) - You try to upgrade one component that is tightly coupled with others; if it fails a little it fails on one component and you're stuck in limbo
* Immutable infra is maybe a bit more like a blue green deployment (not his words) - You don't upgrade in place; instead you use a new box, get all the configuration you want running and then you switch the traffic to the right box on success (and kill the old machine)

He makes a great point about stateful resources, about how resources can be coupled/uncoupled and how you want to use different approaches in different ways for such resources. Databases being mutable for example (due to persistent state) and instances being immutable, since you can create and redeploy them without state. 

On watching it though, I felt unsure that terraform's immutability still reigns. His video is [here](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure) if you want to watch it. It's from 2018 so I wonder if back then, there simply weren't many providers dealing their hand into the world of stateful resources.

In the last 5 years or so using terraform, I've noticed that plugins can vary a lot. The fact that people use terraform for Snowflake (and how common it is), is for example interesting. 

With Snowflake in particular, the problems I personally saw were actually not with the setup of pipes, streams, dbs or datasets (the snowflake plugins is I believe developed by Snowflake and so, they are doing a good job on it). But rather dealing with things like the incredibly fast volatility of table creation (which makes no sense at all to hold in tf) and permissions grants needing to result from those creations. Holding a waterfall of inherited grants makes a lot of sense in tf - But, what if those grants must be re-applied upon each table creation? Ultimately lots of statefulness, lots of volatility, highly coupled architecture ... suggests something for on-creation-callbacks rather than the more staid terraform CICD. 

This has been a bit of a tangential comment, but it'll be interesting to see how the landscape of plugins (and mutability) continues to change with IaC.

-
-



That 1-pager I mentioned up above? The below is its summary. 

#### Resources
* Resources are items that can be provisioned

#### Mutable vs Immutable architecture
* Terraform's conception was for immutable architecture (but this does actually depend on the resources and the provider plugin)

#### Modules
* Encapsulate your configuration in modules to reduce duplicate code 
* Modules can be versioned
* Can be local or can create in [private module registry](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules)
* Design to be smaller and have input variables ; make it flexible and reusable
* Limit child modules to max depth 2 (one module calls another)

##### Workspaces and statefiles
* A workspace manages a single statefile
* Consider blast radius and try to keep small (how much resources affect each other)
* Group together necessary and logically related resources
* e.g. compute vs db may be separate since they can operate independently

##### Volatility
* How frequently does the resource change? -> Group by that frequency and how tightly couples the resources are
* Reduce risk of unexpected changes but grouping resources frequently changed together

##### Statefulness 
* Consider whether your resource has state i.e. if you recreate it from scratch, is something lost?
* Protect against loss of state by grouping stateless vs stateful resources

##### Permissions
* Group by team and expected permissions level - Each team is responsible for their own workspaces - it means they can edit their resources
* e.g. App team may need access to compute resources but no IAM, devops team will need IAM priviledges 
* If there are team interdependencies, use [outputs to coordinate](https://registry.terraform.io/providers/hashicorp/tfe/latest/docs/data-sources/outputs)
* Permissions can be different across for example, test vs prod environments. As such separate workspaces are helpful here

##### Speed and Dependency
* Avoid huge plans that take a while to build -> Dependency graphs may eventually require huge RAM

##### Projects
* Group workspaces into projects for easier permissions management

##### Statefiles
* They show the current expected configuration
* Anything not declared will be ignored / not known
* Resources created but changed in the UI may not be picked up
