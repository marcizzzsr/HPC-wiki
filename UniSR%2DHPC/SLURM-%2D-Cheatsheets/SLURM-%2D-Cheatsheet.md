[[_TOC_]]

# Partitions

A partition in a SLURM system is a logical group of nodes where **jobs** can be submitted.
- It is a way to **organize and divide cluster resources** (CPU, RAM, GPU, full nodes) into distinct sets.
- Each partition can have **different policies**: time limits, available resources, authorized users, priority levels, etc.
- When you submit a SLURM job (`sbatch`, `srun`), you can specify which partition to use.
- If you don't specify one, SLURM will use the default partition (usually the only or the most general one).  
  **#To be completed: What happens on our cluster?**

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

> [!tip]
> Use the `interactive` partition for development, testing, and quick interactive debugging. Use the `cuda` partition **only when you need to run training** or tasks that truly require GPU resources.

# Time limit

Each SLURM partition has a `TimeLimit` parameter that sets the **maximum execution time** for a job running on that partition.

## `--time`

It is recommended to use the `--time` parameter when calling `srun` to explicitly define a maximum execution time for your job. If omitted, SLURM will assign a default time limit (e.g., 1.5h), after which the job will be terminated regardless of its state.

Example command with the `--time` parameter:
```bash
srun --partition=interactive --pty --cpus-per-task=4 --mem=16G --time=6:00:00 bash
```
This will request a job that will be terminated **six hours after creation**.

> [!warning] Respect the maximum time
> If you request a lifetime longer than the maximum allowed for the selected partition, SLURM will automatically reject the job submission.

# Kill a Job

To terminate a job, you need its `<job_id>`. Example steps:
1. Use `squeue -u $USER` to list all active jobs for your user.
2. Identify the `<job_id>` of the job(s) you want to cancel.
3. Run `scancel <job_id>`

If you want to terminate **all your jobs**, use `scancel -u $USER`.

Once termination is requested, the job will enter the `CG` state (*completing*) and may take up to 30 seconds to fully stop and disappear from `squeue`.

# Run training scripts

We recommend using option A. We report options B and C for educational and debugging purposes.

## A) Run apptainer workload directly with slurm batch

Submit batch job to run script inside apptainer with GPU support and bind mount:

```
sbatch -p cuda --gpus 1 --mem=32g --cpus-per-task=16--wrap "cd /mnt/data/unisr-data/test_script_gpu/work/ && apptainer exec --nv --bind /mnt/scratch/unisr-data/data/ccta:/mnt/scratch/unisr-data/data/ccta monai.sif python train.py"
```

- If multiple GPUs are required, just ask for more resources with sbatch (`--gpus 2`) + obviously adapt the script for parallel computing.
- Slurm will automatically assign you the available workstation. If you wish to manually select it, add `-w workstation-ai-01` (or `-w workstation-ai-02`) just before `--wrap`.
- The above command should return the progressive identifier of the job, hereafter referred to as `{job_id}`. By default, Slurm creates a log in the current working directory, following the pattern `slurm-{job_id}.out`.

Monitor job output:

```
tail -f slurm-{job_id}.out
```

## B) Run apptainer workload from interactive slurm session

Start interactive slurm session with GPU resources:

```
srun -p interactive --gpus 1 --mem=32g --cpus-per-task=16 --pty bash
```

Run python script directly inside apptainer container with GPU support and bind mount:

```
apptainer exec --nv --bind /mnt/scratch/unisr-data/data/ccta:/mnt/scratch/unisr-data/data/ccta monai.sif python train.py
```

Exit slurm session:

```
exit
```

## C) Run python script from shell within interactive slurm session + interactive apptainer session

Start interactive slurm session with GPU resources:

```
srun -p interactive --gpus 1 --mem=32g --cpus-per-task=16 --pty bash
```

Launch interactive apptainer container with GPU support and bind mount

```
apptainer run --nv --bind /mnt/scratch/unisr-data/data/ccta:/mnt/scratch/unisr-data/data/ccta monai.sif
```

Run python script inside the container:

```
python train.py
```

Exit apptainer session:

```
exit
```

Exit slurm session:

```
exit
```