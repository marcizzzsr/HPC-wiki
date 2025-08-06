# Jupyter
We recommend using the Jupyter server only for interactive development; please run the actual training with the commands listed above.

The entry point is a bash script that handles ports, binds, and mounts. We have pre-built entry points at `/mnt/data/unisr-data/jupyter_entrypoint`. The pre-built entry points are linked to the pre-built Apptainer images (`.sif`) located at `/mnt/data/unisr-data/apptainer_images/sif`.

```bash
cd /mnt/data/unisr-data/jupyter_entrypoint
tree
#.
#.
#└── jupyter_monai_1_5_0_custom.sh
```
To start the Jupyter server, run:
```
sh /mnt/data/unisr-data/jupyter_entrypoint/jupyter_monai_1_5_0_custom.sh
```
The terminal will prompt you to select the source directory to bind (accessible in Jupyter):
```
Select source directory for datasync:
1) /mnt/data/unisr-data/jupyter_datasync (HDD storage)
2) /mnt/beegfs/scratch/unisr-data/jupyter_datasync (SSD storage)
```
Finally, copy the address (the first one located after `Jupyter Server 2.14.2 is running at:`) and paste it into a browser.

# Helpers and basic commands

Check slurm queue:

```
squeue
```

Delete slurm job

```
scancel {job_id}
```

Check directory size:

```
du -sh ccta
```

Diff:

```
diff --color train_1.py train_2.py
```

Dependencies of a pip library:

```bash
curl https://pypi.org/pypi/feature-engine/1.8.3/json | jq '.info.requires_dist'
```