---
title: What is an Environment Variable and Slurm how to handle it.
date: 2024-05-14
permalink: /posts/environment-variable
tags: 
    - slurm 
    - shell
    - Linux
---

It is not a complex concept of environment variables that every coder will face to in their first coding experience 🤔, but I still have some confusing point while I start utilizing HPC for computation. So, I write a post to comb this concept. And during the process of writing, I got more clear about it, that is joyful!!

### What is an Environment Variable?

An **environment variable** is a dynamic variable in an operating system environment that can affect the way running processes behave. Environment variables are used to store system-wide values that can be used by applications and scripts to configure their behavior and pass configuration data to them. Examples include `PATH`, which specifies the directories the shell should search for executables, and `HOME`, which specifies the path to a user’s home directory.

### How to Check All Environment Variables in a Linux System

There are several ways to view all environment variables in a Linux system:

#### 1. Using the `printenv` Command

The `printenv` command displays the values of the environment variables.

- To display all environment variables: 
	- `printenv`
- To display the value of a specific environment variable, say `HOME`:
	- `printenv HOME`
	-  `echo $HOME`
	
#### 2. Using the `env` Command

The `env` command can also be used to display environment variables.
- To display all environment variables:
	`env`


When I utilize a high performance cluster (HPC) for computing task, I have a confusion about the environment variables in different `node`. Before we get the certain of such question, we should understand the architecture of a typical cluster.

### Architecture and Relationship Between Node and OS in a Cluster

In a computing cluster, the architecture typically consists of multiple nodes (individual computers) networked together to perform distributed computing tasks. Here’s a basic overview of the components:

1. **Cluster Nodes**:
   - **Head Node / Master Node**: The central node where users log in, submit jobs, and manage the cluster.
   - **Compute Nodes / Worker Nodes**: Nodes where the computational tasks are executed. These nodes are managed by the head node.

2. **Operating System (OS)**:
   - Each node in the cluster runs its own instance of an operating system, usually a Linux distribution. The OS on each node manages hardware resources and provides an environment for executing tasks.

3. **Resource Manager / Job Scheduler**:
   - Slurm (Simple Linux Utility for Resource Management) is a common job scheduler used to manage job submissions, queue jobs, allocate resources, and monitor job execution across the nodes.

### Environment Variables in a Cluster

Environment variables in a cluster can vary based on how they are configured and propagated. Here’s how they typically work:

1. **User-Specific Environment Variables**:
   - When you log into the head node, your shell initialization files (e.g., `.bashrc`, `.bash_profile`) set up your environment variables on head/master/admin node.

2. **Node Environment Variables**:
   - Each compute node has its own set of environment variables defined by system-wide initialization scripts (e.g., `/etc/profile`, `/etc/bash.bashrc`) and any scripts specific to the node.

3. **Job Submission and Environment Propagation**:
   - When you submit a job using Slurm, the environment variables from your current session on the head node are captured and can be propagated to the compute nodes where the job runs.

### How Slurm Handles Environment Variables

1. **User-Specific Environment Variables**:
   - Each user on a system typically has their own set of environment variables that are configured in their shell initialization files (e.g., `.bashrc`, `.bash_profile`, `.profile` for bash users).
   - When you log into the cluster, these files are sourced, setting up your environment variables specific to your user session.

2. **Submitting Jobs with Slurm**:
   - When you submit a job to Slurm using the `sbatch` or `srun` commands, the environment variables from your current session are captured and passed to the job's environment by default.
   - This means that any environment variables you have set in your session (before submitting the job) will be available in the job's environment.

### Ensuring Consistent Environment Variables Across Nodes

To ensure that each node has the same environment variables for your job, you can use several methods:

1. **Propagate Environment Variables with Slurm**:
   - By default, Slurm propagates the environment variables from your submission session to the compute nodes. This can be controlled and specified using the `--export` option.
   - Example:
     ```sh
     sbatch --export=ALL my_job_script.sh
     ```
   - `--export=ALL` ensures that all current environment variables are exported to the job’s environment.

2. **Set Environment Variables in Job Script**:
   - You can explicitly set environment variables within your job script to ensure they are consistent across all nodes.
   - Example:
 ```sh
     #!/bin/bash
     #SBATCH --job-name=my_job
     #SBATCH --output=output.txt
	 
     export MY_VARIABLE=value
     echo "MY_VARIABLE is $MY_VARIABLE"
 ```

3. **Use a Common Initialization Script**:
   - You can create a common initialization script that sets the required environment variables and source it in your job script.
   - Example:
```sh
     #!/bin/bash
     #SBATCH --job-name=my_job
     #SBATCH --output=output.txt
	 
     source /path/to/common_env.sh
     echo "MY_VARIABLE is $MY_VARIABLE"
 ```

4. **Cluster-Wide Configuration**:
   - The system administrators can configure environment variables cluster-wide by placing scripts in directories like `/etc/profile.d/` that are sourced for every user session on all nodes.

### The typical issue caused by environment variables.

Once I have submitted a Python script for downloading HCP data(Human Connectome Project). I fund this job failed because of the net. In the cluster, one compute node usually can't connect to the out internet, that means the compute node can't access the AWS dataset. This issue can be solved by exporting a series of environment variables:

```shell
export http_proxy  = the ip address of master node such as 12.12.12.12
export HTTP_PROXY  = the ip address of master node
export https_proxy = the ip address of master node
export HTTPS_PROXY = the ip address of master node
export ftp_proxy   = the ip address of master node
export FTP_PROXY   = the ip address of master node
export all_proxy   = the ip address of master node
export ALL_ROXY    = the ip address of master node
```

Well down. The compute node will connect the web through the master node that can connect the outer web directly. 


### Summary

- Each user has their own set of environment variables configured in their shell initialization files.
- When submitting jobs with Slurm, the environment variables from the user’s current session are captured and passed to the job.
- You can manage environment variables for Slurm jobs using job scripts, the `--export` option, or sourcing environment variable files.
- Each user's environment in Slurm jobs can be different based on their individual settings and configurations.
- **Node and OS Relationship**: Each node in a cluster runs its own instance of an OS. The head node manages job submissions and resource allocation.
- **Environment Variables**: By default, the environment variables from your session on the head node are propagated to the compute nodes when you submit a job.
- **Verification**: Use job scripts to print and verify environment variables on compute nodes to ensure they are set as expected.


---
# References
