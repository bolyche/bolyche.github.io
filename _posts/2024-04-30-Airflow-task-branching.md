---
layout: post
date: 2024-04-30
title: "Airflow Branching"
categories: [ python, programming ]
tags: 
description: "Airflow Tasks & Branching"
featured: true
hidden: true
image: assets/images/branch.jpg
---

I've been setting up various workflows in Airflow for my job. One thing I've seen in online examples is task branching. The thing I haven't seen is branching (and expansion!) _with passed values between tasks_.

In my experience it's slightly rare to have tasks without any inputs or outputs - Often it'll be something like the location of a file or the name of the temporary table being operated on, or even just the date range for an operation done in a previous task. Such info can either be stored in a remote database (which takes longer connecting to and retrieving) or in Airflow's metadata db which was optimised for it.

So let's work up to it by reminding ourselves of the basics. The below has two tasks (using Taskflow), one getting the value from the other.

```python
import logging
import pendulum
from functools import wraps
from airflow.decorators import task, dag


def log_this(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        logging.info(f"{func.__name__} has args {args}, kwargs {kwargs}")
        return func(*args, **kwargs)
    return wrapper


@task
@log_this
def one(alpha: int) -> int:
    return alpha*10

@task 
@log_this
def two(beta: int) -> str:
    return f"We have {beta} oranges"


@dag(
    start_date=pendulum.today("UTC").add(days=-1), catchup=False, schedule=None
)
def example_1():
    ex1 = one(5)
    ex2 = two(ex1)

    ex1 >> ex2

example_1()
```

From task `one` we get 
```
{ex_1.py:11} INFO - one has args (5,), kwargs {}
{python.py:183} INFO - Done. Returned value was: 50
```

and from task `two` we get the string output of `We have 50 oranges` as expected
```
{ex_1.py:11} INFO - two has args (50,), kwargs {}
{python.py:183} INFO - Done. Returned value was: We have 50 oranges
```

This is pretty simple and self-explanatory. It's also easy to play around with. 

What happens if we add branching though? If we remind outself, branching generally looks as so:
``` python
@task.branch
def where_to_return_value(passed_val):
    if passed_val > 0:
        return "next_task_name"
    else:
        return "backup_task_name"
```
In which case there isn't room for tacking a return value on as it won't be an acceptable branch value. So how do we pass along useful information?

One answer is that we can use xcoms directly (_if someone else knows a better way, do let me know_). 

In which case our example becomes a little more complicated: we have to import `get_current_context` (or use context passed as an argument) and we have to specify the key we store some value under. This necessarily introduces dependencies, as we would expect. We end up linking the branch we're heading to with key-value pair we're adding into xcomms.

```python
import logging
import pendulum
from functools import wraps
from airflow.decorators import task, dag
from airflow.operators.python import get_current_context
# import log_this from above


@task.branch
@log_this
def first_task(anum: int) -> str:
    context = get_current_context()
    if anum > 0:
        context["ti"].xcom_push(key=f"branch_task_option_1", value=anum)
        return "branch_task_option_1"
    else:
        context["ti"].xcom_push(key=f"branch_task_option_2", value=anum)
        return "branch_task_option_2"


@task
@log_this
def branch_task_option_1() -> None:
    context = get_current_context()
    current_int = context["ti"].xcom_pull(
        key=f"branch_task_option_1", task_ids="first_task"
    )
    logging.info(f"xcom_pull resulted in: {current_int}")


@task
@log_this
def branch_task_option_2() -> None:
    context = get_current_context()
    current_int = context["ti"].xcom_pull(key=f"branch_task_option_2", task_ids="first_task")
    logging.info(f"xcom_pull resulted in: {current_int}")


@dag(
    start_date=pendulum.today("UTC").add(days=-1), catchup=False, schedule=None
)
def example_2():
    f1 = first_task(-5)
    b_1 = branch_task_option_1()
    b_2 = branch_task_option_2()

    f1 >> [b_1, b_2]


example_2()
```

Now with this we can vary the input (above it's set to -5) and trigger either `branch_task_option_1` or `branch_task_option_2` with the resultant logs of

```
{ex_b_1.py:11} INFO - branch_task_option_1 has args (), kwargs {}
{ex_b_1.py:36} INFO - xcom_pull resulted in: 5
{python.py:183} INFO - Done. Returned value was: None
or
{ex_b_1.py:11} INFO - branch_task_option_2 has args (), kwargs {}
{ex_b_1.py:44} INFO - xcom_pull resulted in: -5
{python.py:183} INFO - Done. Returned value was: None
```

and the alternating skip occurring for the branch not taken, nicely seen in the UI. 

![Task Branching](/assets/images/taskbranch1.png)

The next level from here is to add parallelism for running tasks. For example, say:
* We get an input of x number of files in a list to process (e.g. from pubsub)
* We want to process them in the same task in parallel, then send each task to the appropriate next task based on some criteria (branching - it could be size, it could be the origin of the file, etc)
* These next tasks (which are branched) should operate on each subtask passed - this means that if we started with 10 files and split them into two branches of 3 files from Google, 7 from Apple -> then branch_google should have parallelism of 3 mapped tasks and branch_apple 7 mapped tasks
* Once that's done, all tasks should ouput some value to a final task -> This final task should occur per number of files originally passed

It sounds like a simple enough example and it's certainly simpler in python scripting. The idea of having multiple running processes using the same framework makes sense, if that framework enables it. 

With Airflow the parallelism is achieved through ["expand" (dynamic task mapping)](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/dynamic-task-mapping.html). The sticking point is how mapped values (which all appear as one task per below) can pass through the relevant values to the next task (and how that can branch out again).

![Dynamic Mapping](/assets/images/taskbranch2.png)

In Airflow you can't seem to chain branched and dynamic tasks easily. Creating 20 (mapped) subtasks won't result in those 20 subtasks going and each having their subsequent task occur independently - oh no. 20 mapped subtasks will output a LazyXComAccess object with 20 returned values inside it - Which granted you could likely feed (dynamically) into your next task. It makes it tidier but also adds constraints.

In the content of branching, what happens is that you can add a "collection" task to get all the mapped return values per branch if you've split out the outcomes of the mapped tasks. These can then be input into the next dynamic task mapping themselves to expand into howevever many tasks fell into the proverbial task-bucket. A similar thing of a collection phase seems necessary to combine LazyXComAccess, which you'd do at the end in order to collect all the values together.

The below example shows exactly that. 

![Task Graph](/assets/images/taskbranch3.png)

```python
import logging
import pendulum
from typing import List
from functools import wraps
from airflow.decorators import task, dag
from airflow.operators.python import get_current_context
from airflow.utils.edgemodifier import Label
# import log_this from above


@task
@log_this
def first_task(anum: int) -> List:
    return [i * i for i in range(0, anum)]


@task.branch
@log_this
def second_task_branching(current_status: str, input_int: int) -> str:
    print(f"Status: {current_status}")
    context = get_current_context()
    
    if input_int >5:
        context["ti"].xcom_push(key=f"branch_one", value=input_int)
        print(f"input int is: {input_int}")
        return "branch_one"
    else:
        context["ti"].xcom_push(key=f"branch_two", value=input_int)
        print(f"input int is: {input_int}")
        return "branch_two"


@task
@log_this
def branch_one() -> int:
    context = get_current_context()
    current_int = context["ti"].xcom_pull(
        key=f"branch_one", task_ids="second_task_branching"
    )
    logging.info(f"xcom_pull resulted in: {current_int}")
    return current_int


@task
@log_this
def branch_two() -> int:
    context = get_current_context()
    current_int = context["ti"].xcom_pull(
        key=f"branch_two", task_ids="second_task_branching"
    )
    logging.info(f"xcom_pull resulted in: {current_int}")
    return current_int


@task
@log_this
def branch_one_two(input_int):
    return input_int + 100


@task
@log_this
def branch_two_two(input_int):
    return input_int - 100


@task
@log_this
def combine(a,b):
    return list(a)+list(b)


@task
@log_this
def final_task(input_int: int) -> None:
    if input_int < 0:
        print(f"Amazing! This number is negative!")
    else:
        print(f"Amazing! This number is positive!")


@dag(
    start_date=pendulum.today("UTC").add(days=-1), catchup=False, schedule=None
)
def run_branch_tasks_2():
    f1 = first_task(10)
    f2 = second_task_branching.partial(current_status="GREEN").expand(
        input_int=f1
    )

    b1 = branch_one()
    b2 = branch_two()
    b1_2 = branch_one_two.expand(input_int=b1)
    b2_2 = branch_two_two.expand(input_int=b2)

    c1 = combine(b1_2, b2_2)
    final = final_task.expand(input_int=c1)

    f1 >> f2
    f2 >> Label(">5") >> b1 >> b1_2 >> c1
    f2 >> Label("<5") >> b2 >> b2_2 >> c1
    c1 >> final
```
We can break down what's happening in the code above:
* `second_task_branching` runs 10 tasks and splits them into 2 branches
* The following two tasks collect the LazyXComAccess results per key (named to the branch they're going to) and per task_id (note you can also grab the mapped_index since last year's code change, though less relevant for this example)
* This means that with the current input value `branch_two` will collect 3 results and `branch_one` will collect 7
* Following collection via `branch_two` and `branch_one` we can again `expand` (dynamically map) these tasks to their children, `branch_two_two` and `branch_one_two`
* These kids do some operation and are then combined in the second-to-last task, `combine`
* Following which they all get printed in `final`

I quite like this example because it's not immediately obvious that mapped tasks with the same xcom key and task_id will combine into a list of returned values accessible via xcom_pull. Given this behaviour isn't in the documentation I do hope this is expected rather than accidental. Regardless it allows us a decent degree of flexibility in handling dyanmically changing inputs.



