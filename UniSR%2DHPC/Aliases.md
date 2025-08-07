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
```bash
alias srun-i='srun -p interactive --mem=16 --cpus-per-task=8 --pty bash'
```

# `apptainer`
Set a better command prompt for apptainer sessions, substitutes the standard apptainer command:
```bash
alias apptainer='APPTAINERENV_PS1="\[\e[38;5;129m\]\u@\h-Apptainer\[\e[0m\]:\[\e[38;5;33m\]\W\[\e[0m\]\$ " apptainer'
```
(style yours [here](https://bash-prompt-generator.org/) if you are a customization geek, than paste it after     `APPTAINERENV_PS1=`)