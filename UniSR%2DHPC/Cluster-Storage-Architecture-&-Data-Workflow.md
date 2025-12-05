# Quick Reference

Effective use of the cluster requires understanding the storage hierarchy. The cluster uses a tiered storage approach ranging from slow, high-capacity archival spaces to ultra-fast, parallelized file systems for computation.



| Path | Type | Speed | Capacity | Scope | Best Use Case |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`/mnt/data`** | Network Storage | 🐢 Slow | **Huge** | **Shared** (All Nodes) | Initial upload, cold storage, archiving results. |
| **`/mnt/beegfs/scratch`** | BeeGFS (SSD) | 🚀 **Fastest** | **Medium** | **Shared** (All Nodes) | **Active training**, high-speed I/O. | 
| **`/mnt/beegfs/data`** | BeeGFS (HDD) | 🐇 Fast | **Huge** | **Shared** (All Nodes) | Large datasets requiring high throughput. |
| **`/mnt/scratch`** | Local Disk (SSD) | 🐅 Fast | **Tiny** | **Local** (Single Node) | Temp caching *during* a job. |


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

## A. SSD Tier (The "Hot" Zone)
**Path:** `/mnt/beegfs/scratch/unisr-data`

* **Hardware:** Enterprise NVMe/SSDs.
* **Performance:** Extremely high IOPS (Input/Output Operations Per Second) and low latency.
* **Usage:** This is where your **active training data** should live. If you are training a deep learning model with thousands of small files (e.g., images), place them here.

## B. HDD Tier (The "Warm" Zone)
**Path:** `/mnt/beegfs/data/unisr-data`

* **Hardware:** High-speed HDDs.
* **Performance:** Excellent sequential throughput, good for large files.
* **Usage:** Ideal for large datasets that are too big for the SSD tier or for checkpointing large model states.

---

# Node-Local Storage
**Path:** `/mnt/scratch`

This storage is physically attached to the compute node you are running on.

* **Scope:** **NOT SHARED.** Data on `node-1:/mnt/scratch` is invisible to `node-5`.
* **Visibility:** Only accessible when you have an active allocation via SLURM.
* **Subpath:** `/mnt/scratch/unisr-data` (Shared directories structure).
* **Usage:** Use this for temporary scratch files that do not need to be saved after the job finishes. It eliminates network overhead entirely.

---

# Recommended Workflow

To maximize performance and keep the cluster organized, follow this lifecycle for your data:

1.  **Ingest (Login Node):**
    Upload your raw data or download datasets directly to the slow storage with `scp` or `wget` commands. Example:
    `scp -r ./my_dataset user@cluster:/mnt/data/unisr-data/datasets/`

2.  **Stage (Preparation):**
    Before submitting your SLURM job, copy the dataset to the high-speed BeeGFS SSD tier.
    `cp -r /mnt/data/unisr-data/datasets/my_dataset /mnt/beegfs/scratch/unisr-data/my_project/`

3.  **Compute (Training):**
    Point your training scripts to read from `/mnt/beegfs/scratch`.
    *Optional:* If your job requires extremely low latency temporary files, write them to `/mnt/scratch` (local) during the job.

4.  **Result:**
    Write your logs and model checkpoints to `/mnt/beegfs/scratch`. Once training is complete, move final results back to `/mnt/data` for long-term storage or download.

---

# Notes
- Even though BeeGFS is accessible from the login node it's quite slow and hangs even for simple operations. As of now, it's better to **first hop on a node** via SLURM and then access it.
- Be picky with what you move to fast storage as it's a tiny space compared to slow storage. If you don't need something, just move it back to slow storage.