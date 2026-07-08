# Useful custom commands
This page is dedicated to useful shell custom commands that can make your life easier while using the cluster. You can use any of these scripts or create one of your own, the procedure to make them executable is the same!

[[_TOC_]]

# How to create a user-level command
This procedure allows you to turn any shell script into a user-level command.
You can create your own custom commands that can be run from anywhere on the cluster without needing to type the full path. We will do this by setting up a personal binary directory (`~/.local/bin`).

## 1. Set Up Your Directory and `$PATH`

First, create a hidden `bin` folder in your home directory and tell your shell to look there for commands. Run the following lines in your terminal:

```bash
# Create the directory (no error if it already exists)
mkdir -p ~/.local/bin

# Add the directory to your PATH in .bashrc
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

# Apply the changes to your current session
source ~/.bashrc

```

## 2. Create Your Script

Create a new file inside your new directory. You can name it whatever you want your command to be (e.g., `mycommand`). Notice we drop the `.sh` extension so it feels like a native command.

```bash
nano ~/.local/bin/mycommand

```

Add your code. **You must include a shebang (`#!/bin/bash`)** on the very first line so the system knows how to run it:

```bash
#!/bin/bash
echo "Hello from my custom command!"

```

*(Save and exit your text editor).*

## 3. Make It Executable

For the system to run the file as a command, you must grant it execute permissions:

```bash
chmod +x ~/.local/bin/mycommand

```

## 4. Run Your Command

You are all set! You can now call your script by its filename from anywhere in the cluster:

```bash
mycommand

```

# slog
`slog` is a simple shell script that wraps the known `sbatch` command in order to automatically tail to the output log of the job, saving you the hassle to find the correct log belonging to the job and manually launch `tail -f` to view its output.

Create a file named `slog.sh` and paste the following:
```bash
#!/bin/bash

# Submit the job
SUBMIT_OUTPUT=$(sbatch "$@")
echo "$SUBMIT_OUTPUT"

# 2. Extract Job ID
JOB_ID=$(echo "$SUBMIT_OUTPUT" | grep -oP '\d+')

if [ -z "$JOB_ID" ]; then
    echo "Submission failed."
    exit 1
fi

# 3. Retrieve the actual log path from Slurm
# We loop briefly because 'scontrol' might take a split second to register the path
LOG_FILE=""
while [ -z "$LOG_FILE" ] || [ "$LOG_FILE" == "(null)" ]; do
    sleep 0.5
    LOG_FILE=$(scontrol show job "$JOB_ID" | grep "StdOut=" | cut -d= -f2)
done

echo "Monitoring log: $LOG_FILE"

# 4. Wait for the file to actually appear on disk before tailing
until [ -f "$LOG_FILE" ]; do
    sleep 1
done

tail -f "$LOG_FILE"
```

Now make it executable with `chmod +x ~/.local/bin/slog.sh`.
> Keep in mind that this is just a streaming of the output log! If you want to kill the job you have to manually do it using `scancel`! Closing the stream with Ctrl+C will only stop the stream but not the job from running!