---
title: some key concepts in slurm
date: 2024-05-12
permalink: /posts/some-key-concepts-in-slurm
tags: 
    - slurm 
    - shell
    - Linux
---
Submitting a computational job by SLURM, there are some key parameters, which confused me a lot time, so, I investigate those concepts today and write down here.

The following is a typical submitting script:

```shell
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --nodes=2 # default 1
#SBATCH --ntasks=4   # assign total number of task, but there will be overlap by next parameter
#SBATCH --ntasks-per-node=4 # assign task running on per node, will reset --ntasks
#SBATCH --cpus-per-task=2 # assign cpu cores per task
#SBATCH --time=00:10:00


module load my_module
module load mpi


srun -n 4 mpi_executable arg1 arg2 &      # run task 1
srun -n 1 ./task_executable arg1 arg2 &   # run task 2
srun -n 1 ./task_executable arg3 arg4 &   # run task 3
srun -n 1 ./task_executable arg5 arg6 &   # run task 4
srun -n 1 ./task_executable arg7 arg8 &   # run task 5

# waiting all tasks over down
wait
```

Here, we apply two Nodes through `--nodes=2` (Actually, it is absolutely unnecessary that apply two Node, it is enough within one Node.), and assign the number of CPU cores and the number of tasks running on each node respectively.

Here, I requested two `Nodes` using `--nodes=2`, (in fact, it is unnecessary to run across nodes, running on a single node suffices, this is just an example). I specified that each `Node` would run 4 tasks using `--ntasks-per-node=4`, and further specified that each task would utilize 2 `CPU Cores` using `--cpus-per-task=2`.

And then, several tasks were submitted through  `srun` command.

Firstly, it is important to clarify that the entire script submits what is called a `job`. A `job` corresponds to one or multiple tasks, where a task is a process or multiple threads within a parallel task. Each task will utilize one or multiple CPU cores.

![[./Attachments/slurm-workflow.jpg]]

One `task` corresponds to one `Process`.

When requesting resources, the basic logic is to allocate `CPU cores` based on the number of tasks. By default each task is allocated one core (it is the CPU core that is determined, rather than a physical CPU). Each task corresponds to a `process`, where each process is used to execute different computational program or to use multiple processes in a parallel task (note the difference between parallel processed and parallel threads here).

It is important to note that if multiple tasks are submitted when requesting resources, and if multiple CPU cores are allocated according to the `--cpus-per-task` parameter (default 1), but if in reality only one task is submitted and it is a serial task, then in fact only one core will be actively working, while the remaining cores will be idle.


# Reference

[zhihu: Tutorial of SLURM](https://zhuanlan.zhihu.com/p/356415669)
