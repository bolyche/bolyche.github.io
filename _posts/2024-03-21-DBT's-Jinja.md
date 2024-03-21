---
layout: post
date: 2024-03-21
title: "DBT's Jinja"
categories: [ python, programming ]
tags: 
description: "DBT actually feels ridiculously flexible"
featured: true
hidden: true
image: assets/images/jinja.jpg
---
{% raw %}
I've seen some great examples of jinja in recent years outside of web development. Airbyte's connector development for example has some nice jinja templating going on. They enable you to generate a pre-populated folder with templated classes after launching a script collecting a few options from the user. I hadn't had much of a chance to use it further but I _did_ like the implementation (I tried to get my former company to let me create more open source connectors as my work.. they didn't applaud the idea).

Recently my current company decided to bring in DBT for a bunch of our model transforms and I was tasked with setting up a big chunk. I found the jinja functionality they enable super cool and so wanted to share some examples with you.

Now many will be familar with the standard `{{item_name}}` accessed for example through `{% set item_name = 'test' %}` or through for loops such as `{% for item_name in name_of_iterable %}`. In DBT this is used similarly but in the context of querying data and resulting in a model (a table or view) being made via a select statement. The select statement itself is the outcome of rendering the jinja. 

As such a simple example could be just a set statement used with a select:

```sql
{% set variable_name = 'this is the string that i want' %}

select '{{variable_name}}' as {{variable_name.replace(" ", "_")}}

{# Renders to
select 'this is the string that i want' as this_is_the_string_that_i_want
#}
```

Similarly a loop statement can be created. In the example below we can utilise data structure such as lists and tuples to group a set of items we wish to use in sql statement. These select statements can be unioned per below, as you can see in the "Renders to" comment. This templating is incredibly useful for cases where you have a vast number of disparately incoming datasets which need to be used together. It makes it far easier to read and check which tables or columns are used.

```sql
{%- set table_and_column_tuples = [('table_a', 'cats'), ('table_b', 'dogs')] -%}

{%- for tablename, animal in table_and_column_tuples -%}

select 
    '{{tablename}}' as table_name,
    {{animal}},
    case when lower({{animal}}) like '%cat%' then 1 else 0 end as cat_flag
from zoo_schema.{{tablename}}
group by 2

{% if not loop.last -%}
union all 
{%- endif %}
{% endfor %}


{# Renders to
select 
    'table_a' as table_name,
    cats,
    case when lower(cats) like '%cat%' then 1 else 0 end as cat_flag
from zoo_schema.table_a
group by 2

union all

select 
    'table_b' as table_name,
    dogs,
    case when lower(dogs) like '%cat%' then 1 else 0 end as cat_flag
from zoo_schema.table_b
group by 2
#}
```

If we expand this further, we can include queries (as macros or standalone inside the sql) to first get a particular snippet of data, then utilise it. For example, we can format a query to find the tables matching a string e.g. tables matching orangutan; then utilise the results. Since there isn't a limit to stringing the queries together (or nesting them, as far as I know), in the example below we could have also found the _column names_ for each of those tables and perhaps even done some matching logic. 

```sql
{%- set query_for_table_matches -%}
select 
    table_name 
from animal_schema.INFORMATION_SCHEMA.TABLES
    where table_name like 'orangutan%'
{%- endset -%}

{%- set results = run_query(query_for_table_matches) -%}
{%- if execute -%}
    {%- set results_list = results.rows -%}
{%- else -%}
    {%- set results_list = [] -%}
{%- endif -%}

{%- for table in results_list -%}

select * from animal_schema.{{table[0]}}

{%- if not loop.last %}
union all 
{%- endif -%}

{% endfor %}

{# Renders to
select * from animal_schema.orangutan_brazil
union all
select * from animal_schema.orangutan_australia
union all
select * from animal_schema.orangutan_dreams
#}
```

As with regular jinja you can utilise macros and access them, so if we transform our above example then we have a macro of:

```sql
{% macro get_table_names_like(schema_name, table_name) %}

{%- set query_for_table_matches -%}
select 
    table_name 
from {{schema_name}}.INFORMATION_SCHEMA.TABLES
    where table_name like '{{table_name}}%'
{%- endset -%}

{%- set results = run_query(query_for_table_matches) -%}
{%- if execute -%}
    {%- set results_list = results.rows -%}
{%- else -%}
    {%- set results_list = [] -%}
{%- endif -%}

{{ return(results_list) }}

{% endmacro %}
```

And sql of below:

```sql
{%- for table in get_table_names_like('animal_schema', 'orangutan') -%}
select * from animal_schema.{{table[0]}}
{%- if not loop.last %}
union all 
{%- endif -%}
{%- endfor -%}

{# This again renders to the same thing, but this time with a reusable macro and input arguments
select * from animal_schema.orangutan_brazil
union all
select * from animal_schema.orangutan_australia
union all
select * from animal_schema.orangutan_dreams
#}
```

The macros themselves can be nested. One macro can call on ten other existing macros; those macros can call other macros as part their implementation. It really can become a glorious house of sql cards. (To be honest I'm not sure that would be a good thing at all, certainly not for sql. But isn't it fun to have the possibility?). 

The other possibilities for DBT's jinja that I've certainly enjoyed is the capacity to add it into their yml files. They have a much touted functionality of `run-on-start` or `run-on-end` which can call a macro prior to model builds, or directly after. This last one is great for logging as you can access easily (via their `results` class) the details of the job run (execution time, destination schema, destination table etc). Also great for things like dynamically adding grants, for rerunning PII checks or for teardown, if any is needed. Similarly the `on-start` allows checks on or creation of schemas; particularly useful if you're often running new environments. 

```yml
on-run-end: 
  - "{{ logging(results) }}"
```

My last example is simple but sweet. While you can add a config per sql file, DBT recommend having a high level configuration in a yml file such as their dbt_project.yml. This is useful for getting an overview of which schemas you're creating, how you're materialising the models and any other relevant generic configs. 

As such I can do something standard - such as setting my schema and db for my folder containing the logging sql (logging_directory). I can also do something jinja. Such as as dynamically setting my database (here analogous to a GCP project) based on the target i.e the job destination; dynamically setting the materialisation likewise; and even setting my table to expire if I'm not using production data.

```yml
models:
  +labels:
    source: dbt
  project_a:
    logging_directory:
      +schema: logging
      +database: 'take your pick'
      +materialized: table
    animals_directory:
      +database: "{{ env_var('DBT_GCP_PROJECT') if target.name == 'prod' else 'a gcp project ID here' }}"
      +materialized: "{{ 'table' if target.name == 'prod' else 'view' }}"
      +hours_to_expiration: "{{ None if target.name == 'prod' else 1 }}"
```

All that seems pretty cool and I didn't even get into how DBT deals with code running on multiple platforms (Snowflake, Redshift, BigQuery and so forth. I'll let you know though - 🤫 Surprise surprise, it's also macros! But more fancy - Read here on [dispatch](https://docs.getdbt.com/reference/dbt-jinja-functions/dispatch)).

{% endraw %}