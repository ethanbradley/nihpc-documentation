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

### ***Running Matlab with the use of GPUs***

First, get a GPU node and launch vnc server from the node, for example:

``` bash
srun -p k2-gpu -N 1 -n 4 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
vncserver
```

Then, similarly as done in the previous section, (1) open a local terminal and launch a forward tunnel to the GPU node; (2) use a VNC application to connect to the tunnel; (3) load and launch the Matlab software.

To test the use of the GPU device within Matlab code, copy and paste the script file `matlab_gpu_example.m` below.

``` matlab title="matlab_gpu_example.m" linenums="1"
% The Mandelbrot algorithm iterates over a grid of real and imaginary parts.
% The following code defines the number of iterations, grid size, and grid limits.
maxIterations = 500;
gridSize = 1000;
xlim = [-0.748766713922161, -0.748766707771757];
ylim = [ 0.123640844894862,  0.123640851045266];

% Use the gpuArray function to transfer data to the GPU and create a gpuArray object.
x = gpuArray.linspace(xlim(1), xlim(2), gridSize);
y = gpuArray.linspace(ylim(1), ylim(2), gridSize);

% Many Matlab functions support gpuArrays and run directly on the GPU (e.g., meshgrid)
% The function "ones", below, create an array of ones directly on the GPU
[xGrid,yGrid] = meshgrid(x,y);
z0 = xGrid + 1i*yGrid;
count = ones(size(z0), 'gpuArray');

% The code below implements the Mandelbrot algorithm, fully running on the GPU
z = z0;
for n = 0:maxIterations
    z = z.*z + z0;
    inside = abs(z) <= 2;
    count = count + inside;
end
count = log(count);

% Plot the results
imagesc(x, y, count);
colormap([jet(); flipud(jet()); 0 0 0]);
axis off;
```

The following results should appear when you run this script in the GUI that is running on the cluster's GPU, as per below. As shown in the Matlab command window, the output of calling the function `whos` reveal the many variables (gpuArray objects) that are still allocated on the GPU.

![Image title](assets/cluster_gpu_matlab.png){ width="1000" }

### ***Running Matlab in background: sbatch***

Finally, the most convenient way to use the cluster resources may be to perform Matlab calculations in background, using `sbatch` functionality instead of `srun`. This is critical, mainly for analysis where calculations may take several hours or days. To demonstrate this, simply copy and paste the following "sbatch" script, called `gpu_example.sh`, as an example which relies on the same script prepared above to calculate the Mandelbrot solution.

``` bash title="gpu_example.sh" linenums="1"
#!/bin/bash

#SBATCH --job-name=test
#SBATCH -N 1
#SBATCH -n 4
#SBATCH --mem=20G
#SBATCH --time=00:10:00
#SBATCH --partition=k2-gpu
#SBATCH --gres=gpu:1g.10gb:1
#SBATCH --output=test_%j.log

module load matlab/R2022a

# Ulster University (UU) users must use UU's licence by declaring
# (removing comments) these environmet variables:
#export MLM_LICENSE_FILE=27000@193.61.190.229
#export LM_LICENSE_FILE=27000@193.61.190.229

matlab -nosplash -nodisplay -r "matlab_gpu_example; saveas(gcf, 'Mandelbrot'); exit;"
```
In the code above, line #19, Matlab is called in "nosplash" and "nodisplay" mode to execute the `matlab_gpu_example.m` script. Besides, note that the code will run in background, i.e., without display, therefore it must use some Matlab function like `saveas`, as shown in the code, which saves the graphical output. As a result, the Matlab file that must be listed now in the working directory, `Mandelbrot.fig`, should contain the visual results. Finally, to run this "sbatch" script, run the following command in your terminal.

``` bash
sbatch gpu_example.sh
```

## **Python with Anaconda**

## **Ansys**

For more than 50 years, Ansys software has enabled innovators across industries to push boundaries with the predictive power of simulation. From sustainable transportation and advanced semiconductors, to satellite systems and life-saving medical devices, the next great leaps in human advancement will be powered by Ansys.
[https://www.ansys.com](https://www.ansys.com)

Ansys is a licensed software. In order to use it, users have to be registered in the license server. If you are not already included in the license server, you must contact the person in charge of it:

QUB: TBA </br>
Ulster: TBA

When the module is loaded, license parameters for QUB are loaded by default. Ulster users should change these license parameters in their batch script or interactive session. Contact the person in charge of the license for details.
User manual for Ansys is not publicly available, and only licensed users can access to it. To access to this material, register yourself in the Ansys web seite. </br>
[https://customercenter.ansys.com](https://customercenter.ansys.com){target=_blank}

### ***Load Ansys module***

The latest installed version on Kelvin-2 is the 2023R1. To load Ansys:

    $ module load apps/ansys/2023.1

### ***Ansys Fluent batch script example***

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

## **Paraview**

A recommended method to run Paraview on Kelvin-2, it is to start a PV server on Kelvin-2, and to use your local machine to visualize the graphical interface. In this case, you need an installation of Paraview in your local machine as well.

With the following method, Paraview can be run in parallel, and use the local machine to work with the graphical interface.
It is necessary to set a tunnel through the Kelvin-2 login node.

As example, we are going to explain how to set the tunnel and connect with a computer outside the QUB campus. If the local machine is insithe the QUB campus, you have to remove the specification of the port `-p`, the identification private-key file `-i`, and to change the IP name to `kelvin2.qub.ac.uk`.

<ins>Step 1</ins>. Run pvserver in a computing node of Kelvin-2.

Open the session in Kelvin-2 as usual, and then open an interactive session. `<N>` will be the number of cores you require for your parallel job on Paraview.

    $ srun --pty --partition=k2-hipri --ntasks=<N> --mem-per-cpu=2G /bin/bash

Then, you are transferred to a computing node.

Now load the necessary modules, for paraview, and for OpenMPI.

    [<uid>@nodeNNN [kelvin2] ~]$ module load apps/paraview/5.8.1/bin
    [<uid>@nodeNNN [kelvin2] ~]$ module load mpi/openmpi/4.0.4/gcc-9.3.0+ucx-1.8.0

Run the server with the command

    [<uid>@nodeNNN [kelvin2] ~]$ mpirun -np <N> pvserver --force-offscreen-rendering --server-port=<YYYY>

Where `<uid>` is your Kelvin-2 username, `<N>` is the number of MPI tasks you want to run Paraview, and `<YYYY>` is a port number your choice.

<ins>Step 2</ins>. Create the tunnel to the port `<YYYY>`, through the port `<XXXX>` of your local machine.

In a different shell on your local machine, open the tunnel with the command:

    (local)$ ssh -L <XXXX>:nodeNNN:<YYYY> -p 55890 -i ~/.ssh/my-kelvin-key <uid>@login.kelvin.alces.network

Where `<uid>` is your Kelvin-2 username, `<YYYY>` is the port number you let open in the former step, and `<XXXX>` is a different port number your choice on your local machine.
`nodeNNN` is the computing node that the `srun` command directed you in the former step.

<ins>Step 3</ins>. Open paraview in your local machine.

Inside the application click:<br>
File - Connect<br>
- Give a name to the session, e. g. "remote machine"<br>
- In "Server Type" choose: Client / Server<br>
- host: localhost<br>
- Port: `<XXXX>`  (from step 2)<br>

Configure<br>
- Startup Type: Manual<br>
- Press Save<br>

Connect<br>
Once you created the client the first time, you can recover it the next time you open Paraview, you do not need to create it again, and you can connect directly in the step 3.

![Paraview Frontend](assets/Paraview_fronted.png)
