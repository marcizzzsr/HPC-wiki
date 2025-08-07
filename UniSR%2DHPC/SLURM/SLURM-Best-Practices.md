
SLURM Best Practices
===========================================

Running jobs on a shared research cluster requires more than just launching a script. It’s about working efficiently, respectfully, and reproducibly — especially when using high-performance resources like GPUs. This guide distills best practices for using SLURM in a way that aligns with common research workflows, whether you're training a model, running simulations, or experimenting with new ideas.

* * *

Use Interactive Sessions Thoughtfully: For Development, Testing, and Exploration
--------------------------------------------------------------------------------

When you’re writing code, debugging, running notebooks, inspecting datasets, or testing container setups, your goal isn’t to consume huge compute — it’s to iterate quickly. That’s where interactive jobs (`salloc`) shine.
An interactive job gives you a temporary shell on a compute node, just like working on your own machine — but with more RAM, CPUs, and access to mounted data.
**Always use interactive jobs** for:
*   Writing and testing code in real time
    
*   Running a Jupyter Lab or notebook
    
*   Exploring datasets or model outputs
    
*   Debugging container configurations
    
**But**: start small. If you’re just testing logic or running lightweight operations, request 2–4 CPUs and modest memory. There’s rarely a need for GPUs here, unless you’re testing GPU-specific logic or performance-critical paths.
Example:

    salloc --cpus-per-task=4 --mem=8G --time=1:00:00
    

Avoid using batch jobs to "develop" or manually test code — they take longer to spin up, and it’s a waste of scheduling resources. Likewise, never do development directly on the login node — it’s not designed for compute and can impact everyone.
Think of interactive jobs as your personal sandbox. Use them frequently — but keep them short, and scoped to real development needs.

* * *

Plan and Submit Batch Jobs Responsibly
--------------------------------------

When it’s time to do real work — training a model, running a simulation, executing a containerized pipeline — use batch jobs with `sbatch`. A batch job is non-interactive and scheduled to run as soon as resources are available. This is the correct way to run any long-running or resource-heavy task.
To use SLURM effectively in batch mode, you need to:
*   Estimate the resources your job will need
    
*   Request time close to your expected runtime
    
*   Write a clear, minimal job script that runs only what is required
    
*   Name your job and organize output logs
    
Overestimating resources (e.g., asking for 2 GPUs and 64 GB RAM for a lightweight script) leads to longer wait times for you and blocks the queue for others. Instead, profile small jobs first and adjust upwards only when needed.
Use logging smartly. Capture standard output with `--output=logs/%x_%j.out`, and log your experiment results to separate files, not just stdout. Avoid chaining unrelated commands in one job; split steps (e.g., preprocessing, training, evaluation) into multiple batch jobs, or use job dependencies (`--dependency`) if order matters.
Also: if something goes wrong or you submit by mistake — cancel the job. `scancel JOBID` frees up resources right away and avoids unnecessary cluster load.

* * *

Manage Containers Carefully: Build Once, Run Everywhere
-------------------------------------------------------

SLURM works well with container systems like Apptainer (or Docker). But poor container usage leads to bloated environments, compatibility issues, and hard-to-reproduce results.
Use containers when:
*   You need specific software versions or dependencies
    
*   You want to encapsulate an experiment or environment
    
*   You’re running something complex or multi-step
    
However, [**avoid using containers in ways that violate the purpose of modularity**](/UniSR%2DHPC/Apptainer/Containers-best-practices). 
    
Use interactive sessions to test your container setup (`apptainer exec container.sif bash`), and once it works, run it via batch.

* * *

Monitor, Iterate, and Clean Up
------------------------------

After every run, check what actually happened. SLURM tools like `seff JOBID` (if available) show you memory and CPU usage — did you use what you asked for? If not, adjust. If your job used only 5% of memory, request less next time. If it hit the wall and was killed early, request more.
Keep logs organized. Use job names that reflect the task (`--job-name=pretrain_nlp`) and store logs in task-specific folders. Over time, this will save you hours in debugging and reviewing experiments.
When you're done: clean up. Remove intermediate files you don’t need, archive logs, and release disk space. Remember — this is a shared environment. Keeping your space tidy helps everyone.
And finally, if you're unsure how to configure your job or interpret results — ask. It’s always better to get feedback before launching a 48-hour multi-GPU job that fails in 2 minutes.

* * *

Summary: Think Before You Submit
--------------------------------

Whether you’re debugging code or launching an experiment, the most important thing is to match your workflow to the **right kind of job**, with the **right level of resources**, using **clear and maintainable environments**.
*   Use interactive jobs for development, light testing, and exploration — no GPUs unless truly needed
    
*   Use batch jobs for running long or heavy computations — and request only what you need
    
*   Use containers purposefully — avoid mixing unrelated tools into the same image
    
*   Monitor job usage and adjust requests accordingly
    
*   Clean up logs, outputs, and unused files
    
Efficient SLURM usage isn’t just polite — it’s how we make sure research moves faster for everyone.