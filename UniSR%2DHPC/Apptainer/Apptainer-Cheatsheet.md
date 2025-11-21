This is a simple Apptainer cheatsheet that gathers hands-on knowledge that we gained by using it. For further and in-depth details, please refer to the official [Apptainer API guide](https://apptainer.org/docs/user/main/index.html).

[[_TOC_]]

---

# Images
https://apptainer.org/docs/user/main/definition_files.html
https://apptainer.org/docs/user/main/build_a_container.html
todo

---

# Sandbox
https://apptainer.org/docs/user/main/build_a_container.html#sandbox
todo

---

# Binds
https://apptainer.org/docs/user/main/bind_paths_and_mounts.html

Apptainer's default behavior differs from Docker. Once a container is launched, the following directories are automatically mounted into it:
- `$HOME`
- `$PWD` 
- `/tmp`
- `/var/tmp`
- Other directories depending on the admin's configuration (e.g., `/scratch`, `/data`, etc.)

To disable these automatic binds, you can use the `--no-mount` flag followed by the directories you want to exclude (e.g., `home,cwd`). See [here](#simulating-docker-behavior) for further details.

## How to bind a directory
To explicitly bind a directory into the container, use the `--bind` or `-B` flag:

```Shell
$ apptainer shell --bind <local path>:<container path> my_container.sif
```

The container path does not need to already exist inside the image. For example:

```Shell
$ apptainer shell --bind /home/usr/project/uc-1/data:/mnt/data my_container.sif
```

Apptainer will create the `/mnt/data` directory and any missing parent directories inside the container. All data from `/home/usr/project/uc-1/data` will then be accessible inside the container at that path.

## Simulating Docker behavior
https://apptainer.org/docs/user/main/docker_and_oci.html#docker-like-compat-flag
To simulate Docker-style isolated container execution—i.e., prevent any automatic bind mounts and thus avoid any impact on the local system unless explicitly specified—you can use the `--containall` (`-c`) or `--compat` flag. This flag creates temporary directories for `$HOME`, `/tmp`, and `/var/tmp` which will be automatically deleted once the container is closed.

```Shell
$ apptainer shell --containall my_container.sif
```
The resulting container will behave similarly to:

```Shell
$ docker run -it my_container /bin/bash
```

---
# Overlays
https://apptainer.org/docs/user/main/persistent_overlays.html

Apptainer uses overlays to allow the temporary extension of an image at runtime. An overlay can be seen as a layer that gets added to the image filesystem and can be either _read-only_ or _writable_.

A persistent overlay is a directory or file system image that “sits on top” of your immutable SIF container. When you install new software or create and modify files in non-binded directories the overlay will store the changes.

**Apptainer only allows one writable overlay per container, hence specifying multiple writable overlay while running an image will result in an error. Conversely, more than one read-only overlays can be specified at runtime.**

## How to use
You first need to create an overlay by specifying its path and size. For example, to create a 1 GiB overlay image:
```shell
apptainer overlay create --size 1024 /tmp/ext3_overlay.img
```

If you want to avoid allocating the full disk space size associated with the overlay, and allow it to grow as you save files to it, just add the `--sparse` keyword:
```shell
apptainer overlay create --sparse --size 1024 /tmp/ext3_overlay.img
```

After that, you can use your overlay in writable mode with any of the [run commands](#run-commands) or in read-only mode just by specifying the path with `<path-to-overlay>:ro`, for example:
```shell
apptainer shell --overlay <path-to-overlay> <path-to-image.sif>
```

---

# Permissions
Apptainer containers inherit user permissions from the host system. This means that any action taken inside the container (file creation, modification, deletion) will follow the same permissions as if performed directly on the host system. **The user inside the container is the same user who invoked the command outside the container, and therefore maintains the same access rights to mounted directories and files.**

---

# Run commands
## `apptainer exec`
This command overrides any script specified in the `%runscript` section of the `.def` file. Therefore, its syntax expects a specific command to be executed inside the image.

If you're looking to open a shell inside your image, use `apptainer shell` instead.
## `apptainer run`
This command directly executes the content of the `%runscript` section in the `.def` file of the Apptainer image. Therefore, it does not expect any additional command after the execution options.
## `apptainer shell`
This command is virtually equivalent to running apptainer exec with the `/bin/bash` command: it opens an interactive shell inside the specified image and ignores the commands defined in the `%runscript`.

---

# Images
**OUTDATED SECTION, check https://github.com/AI-UniSR/apptainer-image-registry**

You can find pre-built images at `/mnt/data/unisr-data/apptainer_images/`; please avoid building new ones unless strictly necessary.

In `/mnt/data/unisr-data/apptainer_images/sif`, you’ll find a list of curated `.sif` files, which are pre-built images. In `/mnt/data/unisr-data/apptainer_images/def`, there is a subfolder for each build. Within each subfolder, you’ll find the `.def` file and any additional requirements that were used to create the corresponding `.sif` file.

```bash
cd /mnt/data/unisr-data/apptainer_images
tree
#.
#├── def
#│   └── monai_1_5_0_custom
#│       ├── monai_1_5_0_custom.def
#│       ├── pip_freeze.txt
#│       └── requirements.txt
#└── sif
#    └── monai_1_5_0_custom.sif
```

## Build `.sif` from `.def`
**This option should be avoided unless strictly necessary!** Please check the [Containers best practices](/UniSR%2DHPC/Apptainer/Containers-best-practices) guidelines before proceeding.

First be sure to **start an interactive slurm session**:

```bash
srun -p interactive --mem=16 --cpus-per-task=8 --pty bash
```

Build `.sif` from `.def`:

```
apptainer build --ignore-fakeroot-command <path-to-sif> <path-to-def>
```

Freeze the Python environment of a `.sif` (recommended for facilitating reporting):

```
cd /mnt/data/unisr-data/apptainer_images/sif/

apptainer exec monai_1_5_0_custom.sif pip freeze > /mnt/data/unisr-data/apptainer_images/def/monai_1_5_0_custom/pip_freeze.txt
```

## Docker support
https://apptainer.org/docs/user/main/docker_and_oci.html

If possible, we advise to build images using Docker (granted you have it installed on your local machine), due to more accessible resources online. Apptainer allows to build images directly from docker ones.

If you find an image of interest in [Docker Hub](https://hub.docker.com), you can transform it into a sif euqivalent using:
```
apptainer pull <image-name>.sif docker://user/image:tag
```

Or, if you want to extend it, start your `.def` file with:
```
Bootstrap: docker
From: user/image:tag
```