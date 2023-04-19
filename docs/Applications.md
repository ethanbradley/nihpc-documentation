# Applications

Here we will provide information on some of the most popular centrally installed applications and software tools on Kelvin2.

## Matlab

MATLAB is a proprietary multi-paradigm programming language and numeric computing environment developed by MathWorks. MATLAB allows matrix manipulations, plotting of functions and data, implementation of algorithms, creation of user interfaces, and interfacing with programs written in other languages.

How to load the application:

        module load matlab/R2022a

If X11 forwarding is enabled, running the command matlab will open Matlab's GUI (it works in Mobaxterm). Otherwise, run Matlab in console mode with command:

        matlab -nosplash -nodisplay

However, please, DO NOT RUN MATLAB NOR ANY OTHER SOFTWARE ON LOGIN NODES. 

### Running Matlab interactively: srun + parfor (CPUs)

One of the two correct ways to run Matlab is to request a compute node (interactively) with command srun.

For example, request 1 compute node and 10 cores in "k2-hipri" partition:

        $ srun -p k2-hipri -N 1 -n 10 --mem=20G --time=1:00:00 --pty bash
        $ module load matlab/R2022a
        $ matlab -nosplash -nodisplay

Then, inside Matlab, run parpool for the "local" cluster requesting the same amount of workers as specified in srun:

        p = parpool('local', 10) % 10 cores requested in this example

More robust, the number of workers can be read and set automatically to launch the parallel pool:

        num_workers = str2double(getenv('SLURM_CPUS_ON_NODE'))
        p = parpool('local', num_workers)

Finally, you can run your parallel code, which must include "parfor", with Matlab environment. The parallel cluster can be released with the following commnand:

        delete(gcp('nocreate'))

### Running Matlab interactively: srun + GPUs

### Running Matlab in background: sbatch

## Python with Anaconda

## Ansys

For more than 50 years, Ansys software has enabled innovators across industries to push boundaries with the predictive power of simulation. From sustainable transportation and advanced semiconductors, to satellite systems and life-saving medical devices, the next great leaps in human advancement will be powered by Ansys.
[https://www.ansys.com](https://www.ansys.com)

Ansys is a licensed software. In order to use it, users have to be registered in the license server. If you are not already included in the license server, you must contact the person in charge of it:

QUB: [Marco Geron](mailto:m.geron@qub.ac.uk) </br>
Ulster: TBA

When the module is loaded, license parameters for QUB are loaded by default. Ulster users should change these license parameters in their batch script or interactive session. Contact the person in charge of the license for details.
User manual for Ansys is not publicly available, and only licensed users can access to it. To access to this material, register yourself in the Ansys web seite. </br>
[https://customercenter.ansys.com](https://customercenter.ansys.com)

### Load Ansys module

The latest installed version on Kelvin-2 is the 2023R1. To load Ansys:

    $ module load apps/ansys/2023.1

### Ansys Fluent batch script example

    #!/bin/bash

    #SBATCH --job-name=myfluentjob
    #SBATCH --output=myoutput.out
    #SBATCH --error=myerror.err
    #SBATCH --nodes=1
    #SBATCH --ntasks=32
    #SBATCH --partition=k2-hipri
    #SBATCH --mem=100G

    module load ansys/195

    ## QUB's license, already loaded with the module
    #export ANSYSLI_SERVERS=2325@143.117.212.118
    #export ANSYSLMD_LICENSE_FILE=1055@143.117.212.118

    ## Ulster's license
    #export ANSYSLI_SERVERS=2325@193.61.145.219
    #export ANSYSLMD_LICENSE_FILE=1055@193.61.145.219

    # Set architecture of the CPU (in this case amd64) and environment variables
    export FLUENT_ARCH=lnamd64
    export FL_TMPDIR=$SCRATCH/tmp

    # Create our hosts file 
    srun hostname -s | sort > hosts.$SLURM_JOB_ID.txt

    #Run Ansys Fluent.
    fluent 3ddp -g -t$SLURM_NTASKS -pinfiniband -mpi=openmpi -cnf=hosts.$SLURM_JOB_ID.txt -i my_fluent_input > my_fluent_output.res
