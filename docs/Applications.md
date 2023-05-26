# Applications

Here we will provide information on some of the most popular centrally installed applications and software tools on Kelvin2.

## **Matlab**

MATLAB is a proprietary multi-paradigm programming language and numeric computing environment developed by MathWorks. MATLAB allows matrix manipulations, plotting of functions and data, implementation of algorithms, creation of user interfaces, and interfacing with programs written in other languages.

How to load the application:

``` bash
module load matlab/R2022a
```

If X11 forwarding is enabled, running the command `matlab` will open the Matlab's GUI (it works in Mobaxterm). Otherwise, launch Matlab in console mode as follows:

``` bash
matlab -nosplash -nodisplay
```

However, please, **==DO NOT RUN MATLAB NOR ANY OTHER SOFTWARE ON LOGIN NODES==**. 

### ***Running Matlab interactively: srun + parfor (CPUs)***

One of the two correct ways to run Matlab is to request a compute node (interactively) with the `srun` command.

For example, request 1 compute node and 10 cores in "k2-hipri" partition:

``` bash
srun -p k2-hipri -N 1 -n 10 --mem=10G --time=1:00:00 --pty bash
module load matlab/R2022a
matlab -nosplash -nodisplay
```

Then, inside Matlab, notice that calling the function `feature` (line #1 below) must show that exactly 10 CPU cores are available, corresponding to the number of cores allocated above with the `srun` command. Next, the codes launches the parallel pool (`parpool`) for the "local" cluster requesting the same amount of workers as the cores allocated in this example.

``` matlab linenums="1"
feature('numcores')
p = parpool('local', 10) % 10 cores requested in this example
```

More robust, the number of workers can be read and set automatically to launch the parallel pool:

``` matlab
num_workers = str2double(getenv('SLURM_CPUS_ON_NODE'))
p = parpool('local', num_workers)
```

Finally, you can run your Matlab parallel code, which must include a "parfor" loop. For example, the following code illustrates the use of `parfor` and Monte Carlo simulation to calculate an approximate value for \( \pi \), using the formula (line #7):

$$
\pi \approx \lim_{N_{MC} \to \infty} {4 \sum_{n=1}^{N_{MC}} I(x_i^2 + y_i^2 < 1.0) \over N_{MC}}; x_i, y_i \sim \mathcal{U}(0,1),
$$

where the symbol \( \mathcal{U}(0,1) \) represents the random uniform distribution for the indicated interval and \( I(boolean) \) is an indicator function.

``` matlab title="Matlab code for parfor demonstration" linenums="1"
N = 1e6;
out = zeros(1, num_workers);
parfor i = 1:num_workers
	xy = rand(N,2);
	out(i) = sum(sum(xy.^2,2)<=1);
end
mypi = 4*sum(out)/(N*num_workers)
```

At the end of the parallel computations, the allocated parpool within Matlab can be released with the following command:

``` matlab
delete(gcp('nocreate'))
```

### ***Running Matlab interactively: srun + GPUs***

### ***Running Matlab with GUI directly on a compute node***

First, get a compute node and launch vnc server from the node, for example:

``` bash
srun -p k2-hipri -N 1 -n 6 --mem=10G --time=1:00:00 --pty bash
vncserver
```

In this case, let us assume that the output of `vncserver` is

*New 'node117.pri.kelvin2.alces.network:1 (jsan)' desktop is node117.pri.kelvin2.alces.network:1*

*Starting applications specified in /users/jsan/.vnc/xstartup*<br>
*Log file is /users/jsan/.vnc/node117.pri.kelvin2.alces.network:1.log*

Then, open a local terminal and launch a forward tunnel to the compute node by following these steps:

1. Go to the directory which contains the kelvin key in your PC/laptop
``` bash	
cd /drives/c/Users/jsan/.ssh
```
2. Create the tunnel (in this example illustrated with the command below, all input sent via port *5903* on your local host is being forwarded via port *5901* to the compute node *"node117.pri.kelvin2.alces.network"*. If the `vncserver` output above were *"node117.pri.kelvin2.alces.network:7"*, then the port number will be *5907* instead of *5901*. Clearly, users must replace the username *"jsan"* and the key's filename by the corresponding information for their accounts)
``` bash	
ssh -L 5903:node117.pri.kelvin2.alces.network:5901 -p 55890 -i ./kelvin-key jsan@login.kelvin.alces.network
```
3. Connect to the tunnel using your installed VNC application (the example shown in the figure below uses TurboVNC in Windows OS. More details or troubleshooting can be found in the VNC session)
![Image title](assets/TurboVNC.jpg)
4. Once in the opened terminal for the connected compute node, launch the matlab application:
```bash
module load matlab/R2022a
matlab
```

This time matlab GUI will be opened as shown in the figure below. For a better experience, use the "full screen" button in the VNC toolbar, and use the combination keys ++ctrl+alt+shift++ + F to escape from the full screen mode. Note also that using the instruction `feature('numcores')` inside the Matlab's GUI session shows correctly the number of allocated CPU cores in the compute node (CPU cores equal to 6 in this example).

![Image title](assets/TurboVNC_Matlab.jpg){ width="800" }

### ***Running Matlab in background: sbatch***

Finally, the most convenient way is to run Matlab in background, particularly for analysis where calculations take days. Several examples are discussed next.

## **Python with Anaconda**

## Python with Anaconda

## Ansys

For more than 50 years, Ansys software has enabled innovators across industries to push boundaries with the predictive power of simulation. From sustainable transportation and advanced semiconductors, to satellite systems and life-saving medical devices, the next great leaps in human advancement will be powered by Ansys.
[https://www.ansys.com](https://www.ansys.com)

Ansys is a licensed software. In order to use it, users have to be registered in the license server. If you are not already included in the license server, you must contact the person in charge of it:

QUB: [Marco Geron](mailto:m.geron@qub.ac.uk) </br>
Ulster: TBA

When the module is loaded, license parameters for QUB are loaded by default. Ulster users should change these license parameters in their batch script or interactive session. Contact the person in charge of the license for details.
User manual for Ansys is not publicly available, and only licensed users can access to it. To access to this material, register yourself in the Ansys web seite. </br>
[https://customercenter.ansys.com](https://customercenter.ansys.com){target=_blank}

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

    module load apps/ansys/2023.1

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
We hardly recommend avoiding the system compilers, and to use those ones which are installed as modules. Modules for compilers as flagged in general as

    compilers/<name>/<version>

And libraries

    libs/<name>/<version>/<compiler>

where <compiler> states the compiler and version that it was compiled with, and in some cases, other libraries as dependencies.
If you are going to use a precompiled library, be sure to use for your application the same compiler and version that the particular library was compiled with.

### Compilers available on Kelvin-2

On Kelvin-2, for usual programming languages as C, C++, or Fortran, we recommend using the GNU compiling suite. It is well tested, and it is currently the fastest for AMD systems. It is also a universal compiling suite, so any code will compile with it.

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
In this case, the session must be allocated in the k2-gpu partition, and it must be allocated a GPU resource where the application will be executed. 
For compatibility, we recommend using the A100 GPUs to compile. Further details about how to compile for GPUs will be stated in the specific section.

    srun --pty --partition=k2-gpu --ntasks=1 --mem-per-cpu=4G --gres gpu:a100:1 bash

### Compiling parallel applications

- OpenMP

 All the compilers have integrated the libraries to execute in parallel using the Open Multi Processor protocol.
 In the case of the GNU compiler, the OpenMP protocol is activated just adding the flag

     -fopemmp

 to the compilation command.

- MPI

 To compile using the Message Passing Interface protocol, in Kelvin-2 it is necessary to load the modules for those libraries and change the compiling commands.
 The modules that activate the MPI libraries are flagged in Kelvin-2 as

     mpi/<name>/<version>/<compiler>

 where <compiler> points the compiler and version that the MPI libraries were compiled with.
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

 We hardly recommend using the OpenMPI compiling suite, it has been widely tested, and it works stable on Kelvin-2.
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

### Compiling applications that use GPUs

To compile a program designed to work on the Graphical Processing Units of Kelvin-2, it is essential that it is compiled in a GPU node, so the queue "k2-gpu" must be allocated.
For backwards compatibility, the program must be compiled in the latest model of GPU present in the machine, in our case, the Nvidia A100 GPUs.
So, when allocating the interactive session to carry out the compilation, the resource A100 should be allocated with the flag

    --gres gpu:a100:1

See the specific [section](Job Submission.md#advanced-working-with-gpus-on-kelvin-2) to work with GPUs on Kelvin-2 for more information about how to allocate the resources.

During the last years, the popularity of the specific GPU-focused programming languages, such as CUDA, has decreased.
This is due to the publicly-available libraries have become more and more complete, and practically any mathematical operation that can benefit of the acceleration advantages of a GPU is present on those libraries.
Some examples are "cublas" for linear-algebra operations, and "cufft" for Fourier transforms.
Most of the libraries designed to work in the GPUs of Kelvin-2 can be found in the module for the Nvidia CUDA drivers:

    libs/nvidia-cuda/11.0.3/bin
    libs/nvidia-cuda/11.7.0/bin

These libraries can be including in the executables, as usual adding the flag "-l" to the compilation command, for example

    gcc <flags> -lcublas -lcufft -o my_executable.x my_GPU_program.c

Nevertheless, if you require a very specific operation, that is not present in the available libraries, and it is necessary to use CUDA, the Nvidia compiler can be used.
Currently, the Nvidia compiler is the only one installed on Kelvin-2 that can compile CUDA code

    nvhpc/22.7
    nvhpc/22.7-byo
    nvhpc/22.7-nompi

This compiler can recognise the sections of the code in an intelligent way, so there is no need of more flags to point that it includes CUDA routines.
To compile with Nvidia compiler, just use its usual commands

    nvc <flags> -o my_executable.x my_CUDA_C_program.c
    nvc++ <flags> -o my_executable.x my_CUDA_C++_program.cxx
    nvfortran <flags> -o my_executable.x my_CUDA_Fortran_program.for

