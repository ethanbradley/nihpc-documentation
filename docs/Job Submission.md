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
Once you have submitted your jobs using `sbatch` or `srun` you can then view your queue status to see how your jobs are doing along with further information:

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

Users can delete their own jobs only