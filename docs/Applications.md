# Applications

Here we will provide information on some of the most popular centrally installed applications and software tools on Kelvin2.

## Matlab

MATLAB is a proprietary multi-paradigm programming language and numeric computing environment developed by MathWorks. MATLAB allows matrix manipulations, plotting of functions and data, implementation of algorithms, creation of user interfaces, and interfacing with programs written in other languages.

How to load the application:

        module load matlab/R2022a

If X11 forwarding is enabled, running the command matlab will open Matlab's GUI (it works in Mobaxterm). Otherwise, run Matlab in console mode with command:

        matlab -nosplash -nodisplay

However, please, do not run Matlab on login nodes. 

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