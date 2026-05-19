[[_TOC_]]

# Useful aliases

Here is a collection of useful aliases you could paste into your `~/.bashrc` file to make your life easier.

## How to add an alias
To add an alias just>
- open your `~/.bashrc` file with `nano` or `vim`
- scroll down to the last line
- (add a comment `## Custom aliases`, just to make more readable)
- insert the desired alias
- save and close the file
- source the new `~/.bashrc` with `source ~/.bashrc`

### `srun-i`
Lunch an interactive slurm container with minimum options (8 cores, 16GB RAM, no GPU)
```bash
alias srun-i='srun -p interactive --mem=16 --cpus-per-task=8 --pty bash'
```
### `srun-gpu`
Lunch an interactive slurm container with minimum options (8 cores, 16GB RAM, single GPU)
```bash
alias srun-gpu='srun -p interactive --gres=gpu:1 --mem=8 --cpus-per-task=4 --pty bash'
```

# Color coded prompts
It can be tricky to understand where you are actually writing shell commands into: am I on login node? or maybe in an Apptainer container? A straightforward method to understand where you are is to change the color of the prompt depending on the context. Add the following lines in your `.bashrc` file:
```bash
if [ "$color_prompt" = yes ]; then
        if [[ -n "$SLURM_JOB_ID" ]]; then
                PS1='\[\e[94;43;1m\]SLURM\[\e[0m\] ${debian_chroot:+($debian_chroot)}\[\e[94;1m\]\u\[\e[97m\]@\[\e[94m\]\h\[\e[0;97m\]:\[\e[92;1m\]\w\[\e[0;97m\]\$\[\e[0m\] '
        else
                PS1='\[\e[94;47;1m\]Login\[\e[0m\] ${debian_chroot:+($debian_chroot)}\[\e[94;1m\]\u\[\e[97m\]@\[\e[94m\]\h\[\e[0;97m\]:\[\e[92;1m\]\w\[\e[0;97m\]\$\[\e[0m\] '
        fi
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
export APPTAINERENV_PS1='\[\e[97;41m\]Apptainer\[\e[0m\] ${debian_chroot:+($debian_chroot)}\[\e[94;1m\]\u\[\e[97m\]@\[\e[94m\]\h\[\e[0;97m\]:\[\e[92;1m\]\w\[\e[0;97m\]\$\[\e[0m\] '
unset color_prompt force_color_prompt

```

This will render you prompt something like this:
![image.png](/.attachments/image-6b549b53-393f-4055-b02b-c2c5264908ff.png)

(if you are a customization geek, you can style yours [here](https://bash-prompt-generator.org/))