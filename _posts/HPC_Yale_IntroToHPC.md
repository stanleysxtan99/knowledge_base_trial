---
date: 25_August_2025
title: Introduction to HPC at Yale
categories: HPC at Yale
description:
type: Document
---

# Introduction to HPC at Yale
## Why use a HPC?
- Personal machine is too slow and/or does not have enough storage
- Get access to GPUs
- Have a large number of jobs to run
- Access to many software programs
- Access to databases and Cloud storage
- Support from YCRC

## HPC at Yale
- Yale currently offers several HPC: Grace, McCleary, Milgram, Bouchet

## How do I access HPC at Yale
- YaleSecure, Ethernet (on campus), Yale VPN (if off campus)
- Command line ssh (linux or macOS), Open OnDemand (web-based login), Graphical ssh tool (MobaXterm Windows)
- First and third method use ssh and require ssh key; access via login nodes but computational work has to happen on compute or interactive nodes 

## How do I navigate Open OnDemand
- Log into the [Web Portal](https://docs.ycrc.yale.edu/clusters-at-yale/access/ood/#:~:text=Open%20OnDemand%20is%20available%20on%20each%20cluster%20using,to%20access%20the%20web%20portal%20on%20the%20cluster.) and click on the OOD site link of the cluster you use
![Alt text](./images/yserc-introtohpc0.png)
![Alt text](./images/yserc-introtohpc1.png)
![Alt text](./images/yserc-introtohpc2.png)
![Alt text](./images/yserc-introtohpc3.png)
- "Interactive Apps" allows you to choose and use a specific software (e.g. R Studio) in an Interactive Session
  - Note that sessions **do not end** when website is closed; Can close website and reopen on any computer and start where you left off
  - Sessions end when requested time runs out or users end the session by clicking Delete
- "Clusters" to open a terminal
- "Utilities" to check status of nodes, breakdown of CPU hours used by members of group, quota of size usage et cetera

# Basic Linux for HPC Environments
Linux is an open-source operating system widely used in high-performance computing (HPC) environments for its command-line interface. In an HPC terminal, users navigate the file system using commands.

## Prompt
Upon opening a terminal successfully, you should see a prompt that looks something like this: "[scr93@login2.grace fungi]$". "scr93" would be the netID; "login2" would be the current node; "grace" would be the cluster name (could be "mccleary" or others depending on the cluster you use); and "fungi" refers to the name of the current directory. 

If you see "~" for the name of the current directory, it means you are in your *home* directory. The home directory in Linux is your personal starting point in the file system, typically located at /home/<netid>, where <netid> is your username. In an HPC environment, it is where you land upon login and where scripts, final results, and links to other key directories are stored for easy access. 

## Useful commands
- "ls" to list all files and directories available in your current location
  - In your home directory, you will also see palmer_scratch and project
  - Both palmer_scratch and project are directories; palmer_scratch directory is a temporary 60 day storage and has the most space, it is great for temporary storage of large data files and bases for analysis; project directory is for long-term storage and most work should be completed here.
- "getquota" to see total storage available in key directories
- "cd" and "pwd"; "cd" lets you navigate to different directories and "pwd" prints path of current location
- "nano" creates and edit existing files. Example: nano test.txt. To exit nano editor, ctrl x to exit. 
- "mkdir" to create a directory. Example: "mkdir new_directory" to create a new directory called new_directory
- "cp" and "mv"; "cp" to copy a file or directory from one location to another - copied file will be found in both original and new location; "mv" moves file or directory from one location to another. Example: cp /path/to/file/filename /path/to/destination.
  - Consider using "rsync" which provides progress updates.
  - OOD file upload and download has a 15GB limit. Consider using
    - "wget" for downloads directly from link
    - "Rclone" for moving data from **cloud storage** such as Box, Dropbox, AWS S3 or Google Cloud Storage
    - Globus
- "cat" pritns file contents to terminal; great for short files
- "less" opens file in terminal; great for larger files, has search function
- "rm" to remove or delete object such as files and directories. rm -r DIRECTORY to delete directory and all of its sub-directories and files
- "ssh transfer" to go onto transfer node, as transfer node is more suited for transfer of files (optimized for different tasks)
- "salloc" to go into compute node for most tasks
    - Flags:
      - -p or --partition : to specify partition, devel by default
      - -t or --time : DD-HH-MM-SS or DD-HH, 1 hour by default
      - --mem-per-cpu : memory limit by CPU core, 5GB per CPU core by default
      - -c or --cpus-per-task: number of CPU cores per task, by default 1, specify for multithreaded

## Running Jobs On Cluster
There are two kinds of jobs: interactive versus batch. Interactive jobs are great for development, debugging, or interactive environments. All OOD sessions are interactive jobs and users are limited to **4** interactive sessions simultaneously. Batch jobs are non-interactive, users can run many batch jobs simultaneously, and users can choose to receive emails when jobs starts/ends. 

Job flow: Submit -> Job Pending -> Slurm allocates resources -> Job Starts -> Job Completed/Failed/Timed Out/Cancelled -> Allocated Resources are released

General Purpose Partitions: 
- devel default for interactive jobs (< 6 hours)
- day default for batch jobs (< 24 hours) 
- week long jobs (do not submit < 24 hour jobs to week)

### Loading Modules
Some of the common software installed by YCRC are available using module. Load a module to use the software:
- "module load miniconda" to load default version of miniconda
- "module load R/4.4.1-foss-2022b" to load specific version of R software
- "module list" to see loaded modules
- "module spider R" to see list of R modules varying by version
- "module reset" unloads all modules
 
### Common Slurm Commands for batch Jobs
- "sbatch": submit a batch job, job ID is given to you
- "squeue --me": list all pending and running jobs
- "scancel JOBID": cancel queued job or kill running job
- "nano slurm_script.sh": to create a slurm script
- "sbatch slurm_script.sh": to submit job

### Sample batch script
---
```
#!/bin/bash

#SBATCH -J example_job
#SBATCH -p day
#SBATCH -t 12:00:00 
#SBATCH --mail-type=ALL
#SBATCH --cpus-per-task=1

module reset
module load miniconda
module load R
conda activate my_env

python test_python.py
Rscript my_script.R
```
---

### Monitoring jobs with jobstats
Benefits to monitoring jobs include: optimizing resource utilization, detects problem and helps troubleshooting, and requesting smaller resources reduces wait times for job starts. There are 2 ways to go about doing this:
1. Go to User Portal and select on JobID to view jobstats
2. Go to terminal, when job is finished, run command: "jobstats JOBID". Check output file: "cat slurm-JOBID.out" to view jobstats.

## Slurm Job Arrays
Array is useful when you have many identical independent jobs to run. Example: running 1000 image files that correspond to individual samples. In such cases, using Dead Simple Queue is useful. See [here](https://docs.ycrc.yale.edu/clusters-at-yale/job-scheduling/dsq/); note that all jobs will have the same job ID. 



