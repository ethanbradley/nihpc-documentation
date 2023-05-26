## NI-HPC System Diagram


![Image title](assets/HPC_workflow.png)

## Job Handling

### Slurm

Jobs on the cluster are under the control of a scheduling system, [Slurm](https://slurm.schedmd.com/documentation.html).<br />

Jobs are scheduled according to the currently available resources and the resources that are requested. Information on how to request resources for your job are outlined [here](Modules & Jobscripts.md).<br />

Jobs are not necessarily run in the order in which they are submitted.
Jobs needing a large number of cores, memory or walltime will have to queue until the requested resources become available in. The system will run smaller jobs, that can fit in available gaps, until all of the resources that have been requested for the larger job become available.<br />
<ins>Always run jobs with the specific number of resources needed.<ins>
<br />
<br />

### Submitting a Job

There are 2 classes of jobs that can be ran on Kelvin2.

- Non-interactive - `sbatch`
- Interactive - `srun`

#### sbatch
Jobs are submitted via a job-script
To learn more about writing a jobscript see [here](Modules & Jobscripts.md).<br />

Once you have created your jobscript you then submit it using the `sbatch` command and its name :

    sbatch my_jobscript.sh

Once your job is submitted you will recieve a unique `JOBID`.


#### srun

srun is an interactive job - allows users to run interactive applications directly on a compute node.<br />

To start an interactive job :

    srun --pty /bin/bash

- Users should specify resources required to run.<br />
- Input data is the shell session or application started.<br />
- Output data is shown on screen or can be specified to write elsewhere.<br />
<br />


### Queue status
Once you have submitted your jobs using `sbatch` or `srun`, you can then view your queue status to see how your jobs are doing along with further information:

    squeue -u <username>

Example `squeue` output :

    JOBID PARTITION NAME       USER  ST TIME NODES NODELIST(REASON)
     11    all       mpiJob    user1 PD 0:00 2     node101, node102
     2     all       serialJob user1 R  0:02 1     node101

#### Job states

Once you have executed the `squeue` command take note of the current state (ST) :

- Running jobs (R) - your job is currently running in the compute.
- Queuing jobs (PD) - your job is waiting for resources to become available to run
- Failed jobs (F)- your job submission has failed and should be delted from the queue.

#### Deleting jobs

If you need to delete a job from the current queue you can use the `scancel` command with the unique `JOBID` of the job.

    scancel 8

Users can delete their own jobs only.

## Advanced. Submitting jobs with Slurm, good practice.

### What is SLURM?
Slurm is the application used by Kelvin-2 and other computing clusters to organise the job queues. Slurm distributes the jobs according to the resources requested by the users. Slurm also takes in account the computation resources that each user has already used, and it gives higher priority to users with the least used resources, looking for a fair distribution of those computation resources.

### SLURMs fair use policy
As a difference with other HPC facilities, such as Archer or Archer2, users of Kelvin-2 do not have a time-limit quota. In principle, a user of Kelvin-2 can use infinite computing time, as long as resources are available. Nevertheless, the Slurm queue system will pull down in the priority list the users who most time have consumed.<br />
As a general rule for submitting jobs, try to allocate only the necessary resources for your job, CPUs, GPUs, wall time, memory. If you allocate more resources your job will have to queue longer, until these resources are released. In addition, you can block necessary resources for other users, disturbing them, and in a community machine like Kelvin-2, without time quotas, cooperation, trust and good praxis between the users is essential for the proper working of the facility.<br />
A list of the most common Slurm commands can be found in the document<br />
https://gitlab.qub.ac.uk/qub_hpc/kelvin_training/-/blob/master/slurm-cheat-sheet.txt

### Core versus Node level scheduling
In terms of job scheduling, there is a big difference between Kelvin-2 and other HPCs, for example Archer-2. In Archer-2, where the users have quotas enabled, they are charged for the usage of full nodes and not for CPUs. If the user requires less resources than the available within the node, he will be charged for the full node anyway. In addition, the allocated nodes will be for exclusive use of that user, and no other user will have access to them while the job is running. In that sense, the user is suggested to allocate the least number of nodes and to collapse the resources, CPUs and memory, of the requested nodes.<br />
Kelvin-2 works in a very different way, its nodes have a larger number of CPUs, 128 per node, and it allows several users in a node simultaneously as long as there are resources available. Even more, the Slurm scheduling system tries to spread the jobs evenly among the nodes, trying to minimize the computation charge in an individual node. In consequence, when requesting resources to allocate for a job, it is not a good idea to collapse the resources of the nodes, memory and CPUs. Doing so, probably those resources in a node will be never available, and the job will have to queue for ever. In stead of that, try always to spread the resources evenly among a number of nodes, and also give freedom to Slurm to distribute the resources, for example allocating the total number of CPUs with --ntasks, not restricting the number of CPUs per node with --ntasks-per-node, and equivalent with the memory, allocate per CPU with --mem-per-cpu, and not per node --mem.<br />
In Kelvin-2, multi-node jobs are preferred to single-node multiple-core ones (smp). If your application does not work in multiple nodes, and you are forced to run it in a single node, probably your queue time will be slightly longer.

### Dos and donts, in order to optimize the usage, and minimize the queue time.
  - Never run jobs in the login nodes. That will seriously disturb other users who are logged in. Login nodes are never to run jobs.
  - Allocate interactive sessions with srun if you need to run a job interactively, never use the login nodes.
  - Don't run jobs in the login nodes. Yes, we already said it, but it is convenient to repeat it, really don't.
  - Request the necessary resources for your job. Use the job analysis tool sacct to check if you allocated more memory than the necessary for futures jobs with the same application.
  - Don't allocate more resources than necessary, that will increase the queue time, and will waste resources of the machine that could be used by other users.
  - Try to spread the allocation among several nodes, 20 CPUs and 100 Gb of memory is a reasonable amount to be allocated in a single node.
  - Don't allocate a big number of CPUs or a large amount of memory in a single node.
  - Give freedom to Slurm to distribute the resources among the nodes, use the flags --ntasks and --mem-per-cpu to allocate resources per CPU preferably than per node.
  - Don't restrict the resources per node, do not use the flags --ntasks-per-node nor --mem.
  - Never allocate more resources than available in the nodes. Review the training material for information about the resources per node in each partition.
  - Allocate always the correct partition, double check the resources you require, in particular the wall time, and fit it in the particular partition. Check the partition table in the training material.
  - Specify an error output different to the standard output. In case the job crashes, this output has useful information about why the job failed, so how it can be fix.
  - Activate email notifications.

### Example of a Slurm batch script:

    #!/bin/bash
    # Job Name and Files
    #SBATCH --job-name=<job-name>
    #Output and error
    #SBATCH --output=%x.%j.out
    #SBATCH --error=%x.%j.err
    #Initial working directory:
    #SBATCH --chdir ./
    #Notification and type
    #SBATCH --mail-type=BEGIN,END,FAIL,REQUEUE
    #SBATCH --mail-user=<myemail>@qub.ac.uk
    # Wall clock limit:
    #SBATCH --time=240:00:00
    #SBATCH --no-requeue
    # MPI options
    #SBATCH --partition=k2-lowpri
    #Number of nodes and MPI tasks per node:
    #SBATCH --ntasks=500
    #SBATCH --mem-per-cpu=10G

    # Load Necessary modules
    module load compilers/gcc/9.3.0
    module load mpi/openmpi/4.0.4/gcc-9.3.0
    module load libs/blas/3.8.0/gcc-9.3.0
    module load libs/lapack/3.9.0/gcc-9.3.0

    # Set environment variables
    export PATH=$HOME/bin:$PATH

    # Run parallel program
    mpirun -np 500 <myapplication>

# Advanced. Working with GPUs on Kelvin-2.

## Available GPU hardware
In Bio and Kelvin-2 clusters, they are available the following GPU hardware resources.

| Nodes        | GPU type                 | GRES type | Total | Partition          |
|:------------ |:-------------------------|----------:|------:|:-------------------|
|gpu01         |1 x Quadro K4200          |k4200      |  1    |gpu bio-gpu         |
|gpu02         |4 x Tesla V100 SXM2 32GB  |v100       |  4    |bio-gpu             |
|gpu103-110    |4 x Tesla V100 PCIe 32GB  |v100       | 32    |k2-gpu k2-epsrc-gpu |
|gpu111-113    |4 x Tesla A100 SXM4 80GB  |a100       | 12    |k2-gpu k2-epsrc-gpu |
|gpu114        |28 x CI slices of A100    |1g.10gb    | 28    |k2-gpu k2-epsrc-gpu |

## GPU resource allocation
To allocate GPU resources, you need to specify it with the Slurm flag

    #SBATCH --gres 

and the specific resource, for example:

- Tesla V100 PCIe 32 Gb, full GPU only: `--gres gpu:v100:1`
- Tesla A100 SXM4 80GB, full GPU: `--gres gpu:a100:1`
- CI slices of the A100 in gpu114: `--gres gpu:1g.10gb:1`

## Compiling applications for GPU.
The compilation should be done in the latest hardware, so one of the A100 GPU nodes. The drivers to be uploaded are the Nividia-CUDA version 11.7. These drivers are compatible with older hardware, and they will make the application to work in all the GPUs installed in Kelvin-2.

    module load libs/nvidia-cuda/11.7.0/bin

GPU compilers from Nvidia HPC SDK toolkit can be found in the modules

    nvhpc/22.7
    nvhpc/22.7-byo
    nvhpc/22.7-nompi

See the section about [compiling applications for GPU](Applications.md#compiling-applications-that-use-gpus) for more specific details.
