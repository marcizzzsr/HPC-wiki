Workload Managers for Dummies
=============================

_A Beginner’s Introduction for Research Computing_

* * *

What Problem Are Workload Managers Solving?
-------------------------------------------

In research, we often need more computing power than our laptops can provide — training machine learning models, processing huge datasets, or running simulations. That’s where a **cluster** comes in: a network of high-performance machines (nodes) that we all share.
But sharing computing resources is hard. Imagine 20 researchers all trying to run scripts on the same machines. What if:
*   Two people want to use the same GPU at once?
    
*   Someone launches a job that uses all the memory?
    
*   One job crashes another?
    
This is why we need a **workload manager**.
A workload manager is a system that **organizes, schedules, and runs jobs** (your scripts or commands) on a cluster so that resources are used **efficiently, fairly, and safely**.

* * *

What Is a Workload Manager?
---------------------------

A workload manager (also called a scheduler) is software that:
*   Accepts user requests for computational work (called **jobs**)
    
*   Places them in a **queue**
    
*   Allocates the necessary **resources** (CPUs, GPUs, memory, etc.)
    
*   Starts and monitors the jobs when resources become available
    
*   Ensures users don’t interfere with each other
    
*   Cleans up once jobs finish
    
In short, it's the **middleman** between you and the cluster.
On our system, this workload manager is **SLURM** — but the same concepts apply to others like PBS, LSF, or Grid Engine.

* * *

What Is a "Job"?
----------------

In workload manager terms, a **job** is just a unit of work — typically:
*   A script or command you want the system to run
    
*   Some information about **how much hardware** it needs (e.g. 2 GPUs, 16 GB RAM)
    
*   A time estimate (e.g. run for 3 hours)
    
*   Some optional metadata (job name, output file, etc.)
    
You tell the workload manager:

> "Run this script, using this much hardware, for this long."

Then your job is **queued** until that combination of resources becomes available. Once it runs, you get the results — whether it's printed output, model checkpoints, or processed data.

* * *

Why Not Just Log In and Run Things Directly?
--------------------------------------------

You **can’t** (and shouldn't) just SSH into a node and start running jobs, for a few key reasons:
1.  **Resources are shared:** We don’t want one person blocking GPUs all day.
    
2.  **Reproducibility:** Workload managers track jobs, resources, and history.
    
3.  **Efficiency:** The system knows how to fill gaps and schedule multiple jobs cleverly.
    
4.  **Safety:** Crashing jobs, memory hogs, or runaway scripts can be contained and managed.
    
Instead, you "submit" work through the workload manager, and the system takes care of running it correctly, safely, and fairly.

* * *

Interactive vs Batch Workflows
------------------------------

There are two main styles of using a workload manager in research.

### 1. Interactive Work

This is when you want to explore, debug, or run a script in real time. For example:
*   Trying out a notebook
    
*   Testing a container
    
*   Inspecting data or logs
    
In this case, you request a session (an interactive job), and the system gives you **temporary access** to a node. From there, you can work as if you were on your own machine — but you're isolated and safe from interfering with others.

### 2. Batch Work

This is for long, unattended jobs — training models overnight, running simulations, or doing bulk processing. You write a **job script** that says:

> "Here’s what I want to run, and here’s what I need."

You submit the job, walk away, and the system runs it when ready. When it finishes, your results are waiting in output files or logs.
Both styles are supported — and often used together in research: test interactively, then run longer jobs in batch mode.

* * *

Where Containers Fit In
-----------------------

Many researchers use **containers** (like Apptainer or Docker) to control the software environment: specific versions of libraries, dependencies, Python environments, etc.
The good news: **workload managers can run containers too.**
From the workload manager’s point of view, it doesn’t matter _how_ you run your code — just that you tell it what resources you need, and give it the command (e.g., running a script inside a container).
This allows you to combine **reproducibility (containers)** with **efficient, fair access to resources (workload managers)**.

* * *

What Happens Behind the Scenes?
-------------------------------

When you submit a job, the workload manager:
1.  **Queues your job** based on its priority, resources, and other jobs
    
2.  **Finds a suitable time and place** to run it on the cluster
    
3.  **Allocates the hardware** (e.g., 2 CPUs, 1 GPU, 8 GB RAM)
    
4.  **Launches your job**, monitors it, and handles logs and errors
    
5.  **Cleans up** when it finishes or fails
    
You don’t have to manage nodes, install drivers, or worry about system-level issues — that’s the job of the scheduler and cluster admins.

* * *

Summary: Why You Should Care
----------------------------

If you're using a research cluster, understanding workload managers is essential. They:
*   Let you **run your work efficiently** across powerful machines
    
*   Help you **share resources** with others fairly
    
*   Enable you to **scale up** from small scripts to large experiments
    
*   Allow you to **use containers** for reproducible environments
    
You don’t need to become a SLURM expert. You just need to understand the basics:
*   You **submit** work (jobs) to the system
    
*   You **request** the resources you need
    
*   The system **runs it safely and fairly** when the time is right
    
With this understanding, you're ready to explore hands-on use, whether that's running containers, submitting jobs, or scaling up your research.