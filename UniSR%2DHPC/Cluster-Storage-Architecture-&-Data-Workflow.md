# Quick Reference

Effective use of the cluster requires understanding the storage hierarchy. The cluster uses a tiered storage approach ranging from slow, high-capacity archival spaces to ultra-fast, parallelized file systems for computation.



| Path | Type | Speed | Capacity | Scope | Best Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`/mnt/data`** | Network Storage | 🐢 Slow | **Huge** | **Shared** (All Nodes) | Initial upload, cold storage, archiving results. |
| **`/mnt/beegfs/nvme`** | BeeGFS (NVMe) | 🚀 **Fastest** | **~6TB** | **Shared** (All Nodes) | **Active training**, high-speed I/O for experiments. | 
| **`/mnt/beegfs/hdd`** | BeeGFS (HDD) | 🐇 Slow | **Huge** | **Shared** (All Nodes) | Large datasets, long-term storage. |


[[_TOC_]]

---

# The Landing Zone: `/mnt/data`
**Path:** `/mnt/data/unisr-data`

This is the standard entry point for the cluster. It is hosted on standard network storage. While it is visible to all nodes, it lacks the throughput required for heavy computational tasks.

* **Characteristics:** Shared, Slow, High Capacity.
* **Permissions:** Users have read/write permissions over everything here.
* **Content:** Datasets, common scripts, Apptainer/Singularity images.
* **Usage:** Use this as the target for your `scp` or `wget` commands when bringing data into the cluster.

> ⚠️ **Warning:** Do not run heavy training loops reading directly from here. It will bottleneck the network and slow down your training significantly.

---

# High-Performance Shared Storage (BeeGFS)

For high-performance computing, we utilize **BeeGFS**. Unlike standard storage (NFS), BeeGFS is a parallel file system. It stripes data chunks across multiple servers, allowing your read/write operations to utilize the combined bandwidth of multiple storage targets simultaneously.

## A. NVMe Tier (The "Hot" Zone)
**Path:** `/mnt/beegfs/nvme`

* **Hardware:** Enterprise NVMe/SSDs.
* **Capacity:** Approximately 6TB.
* **Performance:** Extremely high IOPS (Input/Output Operations Per Second) and low latency.
* **Usage:** This is where your **active experiment data** should live. If you are training a deep learning model with thousands of small files (e.g., images), place them here. Store only the data needed for running your current experiments.

## B. HDD Tier
**Path:** `/mnt/beegfs/hdd`

* **Hardware:** HDDs.
* **Performance:** Slower storage, good for large files and archival.
* **Usage:** Ideal for large datasets, long-term storage, or data that doesn't require high-speed access during computation.

---

# Recommended Workflow

To maximize performance and keep the cluster organized, follow this lifecycle for your data:

1.  **Ingest (Login Node):**
    Upload your raw data or download datasets directly to the landing zone with `scp` or `wget` commands. Example:
    `scp -r ./my_dataset user@cluster:/mnt/data/unisr-data/datasets/`

2.  **Stage (Preparation):**
    Before submitting your SLURM job, copy the dataset to the high-speed BeeGFS NVMe tier.
    `cp -r /mnt/data/unisr-data/datasets/my_dataset /mnt/beegfs/nvme/datasets/my_dataset`

3.  **Compute (Training):**
    Point your training scripts to read from `/mnt/beegfs/nvme` for optimal performance.

4.  **Result:**
    Write your logs and model checkpoints to `/mnt/beegfs/nvme` during training. Once training is complete, move final results back to `/mnt/data` for long-term storage or download. If needed, intermediate results can be stored in `/mnt/beegfs/hdd`.

5.  **Cleanup:**
    Be mindful of the ~6TB limit on `/mnt/beegfs/nvme`. Remove experiment data once completed and move important results to `/mnt/data` or `/mnt/beegfs/hdd` for archival.

---

# Notes
- Even though BeeGFS is accessible from the login node it's quite slow and hangs even for simple operations. As of now, it's better to **first hop on a node** via SLURM and then access it.
- Be mindful of the `/mnt/beegfs/nvme` capacity (~6TB). Only keep data there that you actively need for experiments. Move completed or inactive data to `/mnt/beegfs/hdd` or `/mnt/data`.