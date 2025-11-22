This is a simple SLURM cheatsheet that gathers hands-on knowledge that we gained by using it. For further and in-depth details, please refer to the official [SLURM Documentation](https://slurm.schedmd.com/quickstart.html).

[[_TOC_]]

---

# Common SLURM Workload Commands
The following is a simple list of command line commands to use while submitting workloads via SLURM. The same commands can be specified in `sbatch` scripts.

- **`--partition=<name>`**  
  Selects the partition (queue) where the job will run.

- **`--cpus-per-task=<n>`**  
  Requests the number of CPU cores allocated to the task.

- **`--mem=<size>`**  
  Sets the amount of memory reserved for the job (e.g., 16G, 32000M).

- **`--gpus=<n>`** or **`--gres=gpu:<n>`**  
  Requests GPU resources; format depends on cluster configuration.

- **`--time=<D-HH:MM:SS>`**  
  Defines the maximum allowed runtime for the job.

- **`--nodes=<n>`**  
  Requests a specific number of compute nodes.

- **`--nodelist=<node1,node2>`**  
  Forces the job to run on specific nodes.

- **`--exclude=<node>`**  
  Excludes specific nodes from job allocation.

- **`--job-name=<name>`**  
  Sets a custom name for the job.

- **`--output=<path>`**  
  Specifies the file to write standard output, supports patterns like `slurm-%j.out`.

- **`--error=<path>`**  
  Specifies the file to write standard error logs.

- **`--mail-type=<types>`**  
  Sends email notifications for job events (e.g., BEGIN, END, FAIL).

- **`--mail-user=<email>`**  
  Email address used for job notifications.

---

# Partitions

A partition in a SLURM system is a logical group of nodes where **jobs** can be submitted. You can specify the desired partition with `-p <partition-name>` argument.
- It is a way to **organize and divide cluster resources** (CPU, RAM, GPU, full nodes) into distinct sets.
- Each partition can have **different policies**: time limits, available resources, authorized users, priority levels, etc.
- When you submit a SLURM job (`sbatch`, `srun`), you can specify which partition to use.
- If you don't specify one, SLURM will use the default partition (usually the only or the most general one).

In our cluster, there are currently **two partitions**, designed to **isolate different types of work** (e.g., development vs. training), manage **priorities and quotas** across nodes, and ensure service quality — by limiting the resources available to dev/debug jobs in favor of training jobs.

| Partition     | Avail | TimeLimit  | Nodes | State | NodeList               |
| ------------- | ----- | ---------- | ----- | ----- | ---------------------- |
| *interactive* | up    | 12:00:00   | 2     | idle  | workstation-ai-[01-02] |
| *cuda*        | up    | 5-00:00:00 | 2     | idle  | workstation-ai-[01-02] |

- `interactive`
  - Partition designed for interactive sessions: **development, debugging, testing**
  - Has a time limit of **12 hours**
  - Both cluster nodes are assigned to this partition
  - Typically, you can request CPU and RAM here, but probably **no GPU** or with restricted GPU access (depends on policy)  
    **#To be completed: ask IT**

- `cuda`
  - Partition dedicated to heavy jobs that **require GPU** (deep learning, training, intensive inference)
  - Has a maximum time limit of **5 days**
  - The same nodes are assigned here, but with GPU resources activated and reserved (`gres/gpu=2`)


> Use the `interactive` partition for development, testing, and quick interactive debugging. Use the `cuda` partition **only when you need to run training** or tasks that truly require GPU resources.

---

# Time limit

Each SLURM partition has a `TimeLimit` parameter that sets the **maximum execution time** for a job running on that partition.

## `--time`

It is recommended to use the `--time` parameter when calling `srun` to explicitly define a maximum execution time for your job. If omitted, SLURM will assign a default time limit (e.g., 1.5h), after which the job will be terminated regardless of its state.

Example command with the `--time` parameter:
```shell
srun --partition=interactive --pty \
     --cpus-per-task=4 \
     --mem=16G \
     --time=6:00:00 \
     bash
```
This command will request a job that will be terminated **six hours after creation**.

> Respect the maximum time!
> If you request a lifetime longer than the maximum allowed for the selected partition, SLURM will automatically reject the job submission.

---
# Kill a Job

To terminate a job, you need its `<job_id>`. Example steps:
1. Use `squeue -u $USER` to list all active jobs for your user.
2. Identify the `<job_id>` of the job(s) you want to cancel.
3. Run `scancel <job_id>`

If you want to terminate **all your jobs**, use `scancel -u $USER`.

Once termination is requested, the job will enter the `CG` state (*completing*) and may take up to 30 seconds to fully stop and disappear from `squeue`.


---
# How to run workloads

There are multiple way by which you can submit workloads to SLURM. The following options are ranked starting from the most common one. The community standard is to submit them using `sbatch`, other options are listed for completeness.

**On our cluster, it's mandatory to allocate a SLURM compute node and then run the workloads in an [Apptainer container](/UniSR%2DHPC/Apptainer).**

## Run workloads with `sbatch`

This is the standard and most used method to submit SLURM workloads on an HPC cluster. The `sbatch` command can be either run by specifiying each parameter as a command line argument or by conveniently crafting a **shell script** that will list the run arguments along with a bash script to run.

We strongly advise to create a bash script and submit it through `sbatch` using:
```
sbatch <path-to-script.sh>
```

You conveniently baked the SLURM arguments inside the bash script that will be submitted, for example:
```bash
#!/bin/bash
#SBATCH --job-name=myjob
#SBATCH --partition=cuda
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --gpus=1
#SBATCH --time=12:00:00
#SBATCH --output=slurm-%j.out
#SBATCH --error=slurm-%j.err

#... your custom shell script ....
```

- If multiple GPUs are required, just ask for more resources with sbatch (`--gpus 2`) + obviously adapt the script for parallel computing.
- Slurm will automatically assign you the available workstation. If you wish to manually select it, add `-w workstation-ai-01` (or `-w workstation-ai-02`) just before `--wrap`.
- The above command should return the progressive identifier of the job, hereafter referred to as `{job_id}`. By default, Slurm creates a log in the current working directory, following the pattern `slurm-{job_id}.out`.

Monitor job output:

```
tail -f slurm-{job_id}.out
```

## Run workloads with `srun`

todo



