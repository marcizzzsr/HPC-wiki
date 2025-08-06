# Useful aliases

Here is a collection of useful aliases you could paste into your `~/.bashrc` file to make your life easier.

# How to add an alias
To add an alias just>
- open your `~/.bashrc` file with `nano` or `vim`
- scroll down to the last line
- (add a comment `## Custom aliases`, just to make more readable)
- insert the desired alias
- save and close the file
- source the new `~/.bashrc` with `source ~/.bashrc`

# `srun-i`
Lunch an interactive slurm container with minimum options (8 cores, 16GB RAM, no GPU)
```Shell
alias srun-i='srun -p interactive --mem=16 --cpus-per-task=8 --pty bash'
```