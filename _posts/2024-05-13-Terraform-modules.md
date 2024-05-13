---
layout: post
date: 2024-05-13
title: "Terraform Modules inputs/outputs"
categories: [ python, programming ]
tags: 
description: "Terraform modules (inputs and outputs)"
featured: false
hidden: false
image: assets/images/terraform_world.png
---

Balancing example code when teaching always feels like tricky business. If you take real world examples then they might be too lengthy in code and too verbose to explain. On the other hand, I see examples sometimes (outside of module docs) that don't actually outline solutions to real world problems. 

My last blog post was related to this. Wanting to showcase Airflow tasks that don't exist in the perfect universe of pure tasks which sit outside of workflow dependencies (i.e. show that sometimes yes, you do actually want to pass across a value between branched tasks and yes, that that could still enable idempotent and effective workflows).

My latest lunch&learn at work was digging into passing terraform outputs between modules - and the fun ways you can structure outputs. So let's dive into it here, too. 

The code is all [available here](https://github.com/bolyche/terraform_examples/tree/main/pubsub).

For the sake of this example we've got some trivial modules.

In module `topic` we have
* A pubsub topic
* A data import which grabs the GCS storage service account
* An IAM binding for that role, for the topic
* And lastly some trivial outputs

In module `subscription` we have the even sparser
* subscription
* storage notification

These require some bucket to already exist and its name is passed across as a variable. 
There are also other variables we pass across - such as the environment name (used to prefix the topic name) etc.

Now, terraform advises against dynamic inputs. It makes your infra less stable, less predicable, and if you've formatted something wierdly you might end up creating something you absolutely don't want. In the case where you have dependencies (the output of one module being input into a separate module) they recommend staging the apply for these and validating expected outcomes prior to appying the next module. 

I say all this because I am going to slightly break this rule - Not simply for fun though. More to demonstrate what might happen in a one-to-many relationship between modules and their resources. In this particular case yes, we could absolutely have hard-coded lists of values being passed instead of passing them dynamically. Or we could separate the tfstates are ensure careful validation of inputs and outputs. And indeed we could apply the modules separately - and you likely should. 

Now with those caveats in mind, lets look at some examples.

In this first example we have a _list_ of topics to be created i.e 

N.B. (please note that you should set tfvars instead of a default here, but for the case of making the examples more readable you'll find all things that _should_ be tfvars set as the default var instead) 

```
variable "topics" {
  type    = list(string)
  default = ["topic_a", "topic_b", "topic_c"]
}
```

And we invoke all three of them in the module
```
module "default_topics" {
  source     = "../module/pubsub_topics"
  for_each   = toset(var.topics)
  env        = "test"
  topic_name = each.value
}
```

This results in a three topics of this form being created: `module.default_topics["topic_a"].google_pubsub_topic.topic`

The outputs of these (specified in the topics module) can now be passed through to the next module to create a subscription for each, as so:

```
module "default_subscriptions" {
  source            = "../module/pubsub_subscriptions"
  for_each          = module.default_topics
  env               = "test"
  topic_name        = each.value.topic_name
  topic_id          = each.value.topic_id
  bucket_name       = var.bucket_name
  subscription_name = "name_of_subscription"
}
```

Note we're using the module outputs and cycling through it again using `for_each` mapped onto `module.default_topics`. The subscription name is only set once and directly in calling the module. A tidier version of this would use tfvars and variables.

So this results in multiple resources created (specifically 12 resources! a topic, subscription, IAM binding and storage notification) per a list with 3 items inside it. This is pretty effective for us if we want lots of topics created for the same bucket.

We can easily do a similar (reverse) thing of passing through a single topic and using a more complex input to set everything to enable multiple subscriptions per that topic. This would be invoked by the following two modules, and would use a variable such as the one below.

```
module "default_topics" {
  source     = "../module/pubsub_topics"
  env        = "test"
  topic_name = "topic_d"
}


module "default_subscriptions" {
  source            = "../module/pubsub_subscriptions"
  for_each          = var.project
  env               = each.value.env
  topic_name        = module.default_topics.topic_name
  topic_id          = module.default_topics.topic_id
  bucket_name       = each.value.bucket_name
  subscription_name = each.value.subscription_name
}
```

```
variable "project" {
  description = "Map of project configuration for subscriptions."
  type        = map(any)

  default = {
    first_subscription = {
      bucket_name        = "bucket_example_1"
      env               = "dev"
      subscription_name = "tree"
    },
    second_subscription = {
      bucket_name        = "bucket_example_2"
      env               = "test"
      subscription_name = "field"
    },
    third_subscription = {
      bucket_name        = "bucket_example_3"
      env               = "test"
      subscription_name = "bush"
    }
  }
}
```

The above would result in 1 topic created and 3 subscriptions for it. The map variable provided is an effective way to loop through configuration settings, especially for varying resources. Resources that remain the same (e.g. billing project, or env if keeping the same env environment for this state file) would be better kept as a separate variable.

We can also play around with such mapped values within the code block calling the module itself - but this is risky behaviour. A more transparent method might be to output configurations for later usage within your pipeline. For example, we might want an output of all active / applied configurations. This would be combined information from both the output of the topics module and from the configuration file used. There's a variety of functions in terraform to structure the output in the way you'd want it - One such way might be a list of json objects each holding one configuration. 

```
output "subscription_detail" {
  value = flatten([
    for topic_key, topic_value in module.default_topics : [
      for project_item in var.project : {
        project_id        = project_item.project_id
        env               = project_item.env
        subscription_name = project_item.subscription_name
        topic_id          = topic_value.topic_id
        topic_name        = topic_value.topic_name
      }
    ]
  ])
}
```

These outputs can be output to a file if desired by simply running `terraform output > filename.txt`