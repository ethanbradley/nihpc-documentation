The NI-HPC centre hosts the Kelvin-2 research cluster.
Kelvin2 is ran on a Linux (Centos 7) Operating system.
Below we will list the resources available.

## Compute

- 84 x 128 core Dell PowerEdge R6525 compute nodes with AMD EPYC 7702 dual 64-Core Processors (786GB RAM).
- 4 High memory nodes (2TB RAM).
- 32 x NVIDIA Tesla v100 GPUs.
- 16 x NVIDIA Tesla A100 GPUs
- 2PB of lustre parallel file system for scratch storage.

## Storage

There are 4 pools of storage on Kelvin2 :

### <ins>Home<ins>
        
    /users/"username"
 - Default place that users login to.
- 50GB/100K file quota.
- No automated file deletion - therefore good to store smaller/compressed longer term data.

### <ins>Scratch<ins>
        
    /mnt/scratch2/users/"username"
 - Large, shared storage area
 - No Quota
 - Temporary data solution - once per month a purge will delete any data that hasnt been accessed in the previous 3 months.
 - Mainly used for storing data that is neccassary to running jobs and storing their output.
 - Once results are outputed, permanent data should be moved to longer term storage ( Home/McClayRDS ).

### <ins>Temp<ins>

    /tmp

- Local scratch disk on nodes
- Data will be automatically deleted when session ends.

### <ins>McClayRDS<ins>

    /mnt/autofs/mcclayrds-projects
    
- QUB only
- Available only from  login and data movement nodes
- Group quotas enabled
- Replicated to secondary site
- To apply for storage on McClayRDS, contact the Research Data Management team [here](https://libguides.qub.ac.uk/ResearchDataManagement/Introduction)
