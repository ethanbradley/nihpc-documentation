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

## Compilers

Kelvin-2 has a large set of compilers and libraries for those users who compile their own self-programmed applications.
We hardly recommend to avoid the system compilers, and to use those ones which are installed as modules. Modules for compilers as flagged in general as

    compilers/<name>/<version>

And libraries

    libs/<name>/<version>/<compiler>

where <compiler> states the compiler and version that it was compiled with, and in some cases, other libararies as dependencies.
If you are going to use a precompiled library, be sure to use for your application the same compiler and version that the particular library was compiled with.

### Compilers available on Kelvin-2

On Kelvin-2, for usual programming languages as C, C++ or Fortran, we recommend to use the GNU compiling suite. It is well tested and it is currently the fastest for AMD systems. It is also a universal compiling suite, so any code will compile with it.

    compilers/gcc/10.2.0
    compilers/gcc/10.3.0
    compilers/gcc/5.1.0
    compilers/gcc/6.4.0
    compilers/gcc/7.2.0
    compilers/gcc/9.3.0

Other compilers installed in Kelvin-2 are the AOCC, Clang (as part of library llvm), or Nvidia nvhp (only in GPU nodes).

    aoc-compiler/2.2.0
    aoc-compiler/3.0.0
    llvm/12.0.0/gcc-9.3.0
    nvhpc/22.7

In consistence with the general practice, jobs must not be run into the login nodes, and that includes compilation. 
The compilers check the hardware of the node where they are working, and they produce an executable adapted to the architecture of the node. 
Login nodes have different architecture than compute nodes, so an application compiled in the login nodes will likely not work in the compute nodes.

To compile, an interactive session should be opened in a compute node. 
All the compute nodes in Kelvin-2 have the same architecture, so in general it does not matter in which one of them the compilation is performed. 
High-memory nodes have more RAM memory, but the model of the RAM and the processors are the same as the standard nodes, 
so a program compiled in the standard nodes will work in the high-memory nodes.

One exception is to compile a program designed to work on the GPUs. 
In this case, the session must be allocated in the k2-gpu partition, and it has to be allocated a GPU resource where the application will be executed. 
For compatibility, we recommend to use the A100 GPUs to compile. Further details about how to compile for GPUs will be stated in the specific section.

    srun --pty --partition=k2-gpu --ntasks=1 --mem-per-cpu=4G --gres gpu:a100:1 bash

### Compile parallel applications

- OpenMP

 All the compilers have integrated the libraries to execute in parallel using the Open Multi Processor protocoll.
 In the case of the GNU compiler, the OpenMP protocoll is activated just adding the flag

     -fopemmp

 to the compilation command.

- MPI

 To compile using the Message Passing Interface protocoll, in Kelvin-2 it is necessary to load the modules for those libraries and change the compiling commands.
 The modules that activate the MPI libraries are flagged in Kelvin-2 as

     mpi/<name>/<version>/<compiler>

 where <compiler> points the compiler and version that the MPI libraries where compiled with.
 This is important for compatibility of the compilations, it should be used the same compiler for the application than the one that was used to compile the MPI libraries.

 The available MPI implementations on Kelvin-2 are

     mpi/mpich/3.0.4/gcc-4.8.5
     mpi/mpich/4.1.1/gcc-10.3.0
     mpi/mpich2/1.5/gcc-4.8.5
     mpi/mvapich/1.2.0-3635/gcc-4.8.5
     mpi/mvapich2/1.6/gcc-4.8.5
     mpi/openmpi/1.10.1/gcc-4.8.5
     mpi/openmpi/1.10.2/gcc-4.8.5
     mpi/openmpi/1.10.7/gcc-4.8.5
     mpi/openmpi/3.1.3/gcc-4.8.5
     mpi/openmpi/3.1.4/gcc-4.8.5
     mpi/openmpi/4.0.0/gcc-4.8.5
     mpi/openmpi/4.0.0/gcc-4.8.5+ucx-1.4.0
     mpi/openmpi/4.0.0/gcc-5.1.0
     mpi/openmpi/4.0.4/gcc-9.3.0+ucx-1.8.0
     mpi/openmpi/4.1.1/gcc-9.3.0

 We hardly recommend to use the OpenMPI compiling suite, it has been widely tested, and it works stable on Kelvin-2.
 The MPICh suite has not been so deeply tested, and it is not guaranteed that applications compiled with it will work on Kelvin-2.
 If you decide to use the MPICh suite, be sure to test your application before using it for production.

 To compile using MPI, the compiling commands should be changed.
 For example, to compile a C program with the compiler GNU 9.3.0, the command to be used is
 
     gcc <flags> -o my_executable.x my_C_program.c
 
 And if we want to compile it in parallel using the MPI version 4.1.1, the compiling command should be changed to

     mpicc <flags> -o my_executable.x my_MPI-C_program.c

 In general, the compiling commands should be changed

     gcc -> mpicc
     g++ -> mpicxx
     gfortran -> mpifortran

 These values are taken by default, and the command "mpicc" will use the GNU compiler and the MPI libraries.
 If you want to use a different compiler, you have to specify it in the environment variables

    OMPI_MPICC
    OMPI_MPICXX
    OMPI_FC

 For example, to compile using the MPI library v4.1.1 with the clang compiler included in the module llvm v12.0.0, the environment variables should be defined as

    export OMPI_MPICC=clang
    export OMPI_MPICXX=clang++
    export OMPI_FC=flang

 and compile using the commands "mpicc", "mpicxx", "mpifortran".

- OpenACC

 Open accelerator is a feature included in the Nvidia compiler.
 It can be activated adding to the compilation sequence the flag

     -acc

 This option implies an inteligent check-up of the hardware at the time of the execution.
 The compiler will use the available hardware resources at that time to optimize the acceleration of the marked regions of the code as "OpenACC",
 multiple-CPUs or GPUs will be allocated by the compiler for the execution, the user does not need to specify the resources at the time of the execution.

 OpenACC is a feature of the Nvidia compiler. On Kelvin-2, it will work only on the GPU nodes.
 If you want the compiler to take advantage of the GPU accelerators, you must allocate a GPU resource when you compile the code, preferably an A100 GPU.

 More information about OpenACC, including pdf user guides can be found in the web site </br>
 [https://www.openacc.org](https://www.openacc.org){target=_blank}

### Compile applications that use GPUs

