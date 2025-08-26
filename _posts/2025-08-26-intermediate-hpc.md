---
date: 2025-08-26
title: Intermediate HPC 
categories:
  - HPC at Yale
description: intermediate HPC resources
type: Document
---

## Tmux
Tmux is a terminal multiplexer that is useful when you wish to hold and view multiple windows or terminals, all accessed and controlled from a single terminal. 
To use Tmux, you run tmux on login node (not compute node), and "salloc" in tmux. 

### Getting started
To being a tmux session named myproject, type:
```
tmux new -s myproject
```
One main benefit of using Tmux is that it keeps your terminal session alive even if you disconnect from the server. Any running processes (script, applications, interactive jobs) are preserved when you disconnect from the server (detach) and you will pick up exactly where you left off when you reattach later. 

### Detach and (re-)Attach
To detach, type:
1. Ctrl + b: signal to tmux that following keystroke is **meant for it** and not the session you are working in
2. d: detach

To reattach:
1. ```tmux attach -t myproject```

To exit:
1. Ctrl + d

### Using tmux on clusters
Using tmux on the cluster allows you to create interactive allocations that you can detach from. Normally, if you get an interactive allocation (e.g. salloc) then disconnect from the cluster, for example by putting your laptop to sleep, your allocation will be terminated and your job killed. Using tmux, you can detach gracefully and tmux will maintain your allocation. Here is how to do this correctly:
1. Enter the cluster of your choice
2. Open a terminal and start tmux
3. Inside tmux session, submit an interactive job with salloc. See Slurm commands [here](https://docs.ycrc.yale.edu/clusters-at-yale/job-scheduling/#interactive-jobs)
4. Inside job allocation (compute node), start your application (e.g. matlab or R)
5. Detach from tmux by Ctrl + b, then d
6. Later, on the same login node, reattach by running tmux attach

### Operating multiple windows and panes
Tmux allows you to create, toggle between and manipulate panes and windows in your session. A window is the whole screen that tmux displays to you. Panes are subdivisions in the current window, where each runs an independent terminal.

Example:
```
[ms725@grace1 ~]$ tmux new -s analysis
# I am in my tmux session now
[ms725@grace1 ~]$ salloc
[ms725@c14n02 ~]$ ./my_fancy_analysis.sh
```

To monitor CPU and memory utilization:
To split window, type: 
1. Ctrl + b
2. %

Go to the compute note you are working on by running "top":
1. ssh c14n02
2. ```top```
The compute node is obtained from your prompt: e.g. ms725@c14n02.
To switch back and forth between panes: type: Ctrl + b then o.

## Bigmem and GPU nodes
Only request GPU or bigmem nodes if you need them and your program will use them. Similarly, only request multiple CPU and or nodes if you program can use them. Mermoy on compute node is shared between jobs and 5GB is allocated to each CPU. The limit is strictly enforced so jobs exceeding limit are killed. You can request custom memory per node or CPU with sbatch or salloc using ```--mem=6g``` or ```--mem-per-cpu=6g``` (best if memory use scales with CPUs). Other options include Bigmem and GPU nodes.
### Bigmem nodes
These are nodes with 512GB-4TB of RAM, depending on cluster. They are available via bigmem **partition** and reserved for applications with large memory needs. 
To use:
```
sbatch -p bigmem --mem=1500g script.R
```
### GPU nodes
GPU nodes have conventional CPUs with multiple cores, and 1-4 GPUs.
To use GPU
1. request nodes from partition with gpu nodes like "gpu", "gpu_devel", "scavenge"
  Note: Scavenging allows users to:
  - Access unused nodes from other partitions.
  - Run jobs at lower priority, which may be preempted (killed) if higher-priority jobs need the resources.
  - Submit opportunistic workloads, such as: Short jobs, Checkpointable jobs, and Job arrays or dSQ tasks
2. request number of GPUs
3. have an application that use GPUs
4. and specify CPUs
Example:
```
salloc -p gpu -c 8 --gpus=1080ti:4 --mem=64g --time=02:00:00
sbatch -p gpu -c 8 --gpus=1080ti:4 --mem=64g --time=02:00:00 batch.sh
```

## Job Monitoring
1. To check if job is running efficiently and using allocated resources, type ```squeue``` to identify node where job is running. 
2. ```ssh ``` to node and run ```top``` ot ```htop```
3. To check for GPUs: use ```nvidia-smi``` to check utilization

Alternatively, you can check manually using:
1. ```seff jobid```; Enter jobid
2. ```seff-array jobid```; Enter jobid, for array jobs
3. ```sacct -j jobid```; Enter jobid, for more information

## Running Parallel Jobs
Programs can use multiple CPUs on one node. Parallelism is especially useful when you have many data files and want to process each one simultaneously. Another useful example is when you have a large set of parameters and want to evaluate each parameter. 

## Containers for Reproducibility


















