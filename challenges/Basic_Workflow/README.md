# Basic Workflow to Run a Job on Ascent/Summit

The basic workflow for running programs on HPC systems is 1) set up your programming environment - i.e., the software you need, 2) compile the code - i.e., turn the human-readable programming language into machine code, 3) request access to one or more compute nodes, and 4) launch your executable on the compute node(s) you were allocated. In this challenge, you will perform these basic steps to see how it works.

For this challenge you will compile and run a vector addition program written in C. It takes two vectors (A and B), adds them element-wise, and writes the results to vector C:

```c
// Add vectors (C = A + B)
for(int i=0; i<N; i++)
{
    C[i] = A[i] + B[i];
}
```

## Step 1: Setting Up Your Programming Environment
Many software packages and scientific libraries are pre-installed on Ascent for users to take advantage of. Several packages are loaded by default when a user logs in to the system and additional packages can be loaded using environment modules. To see which packages are currently loaded in your environment, run the following command:

```
$ module list
``` 

> NOTE: The `$` in the command above represents the "command prompt" for the bash shell and is not part of the command that needs to be executed.

For this example program, we will use the PGI compiler. To use the PGI compiler, load the PGI module by issuing the following command:

```
$ module load pgi
```

## Step 2: Compile the Code

Now that you've set up your programming environment for the code used in this challenge, you can go ahead and compile the code. First, make sure you're in the `Basic_Workflow/` directory:

```
$ cd ~/hands-on-with-summit/challenges/Basic_Workflow
```

> NOTE: The path above assumes you cloned the repo in your `/ccsopen/home/username` directory.

Then compile the code. To do so, you'll use the provided `Makefile`, which is a file containing the set of commands to automate the compilation process. To use the `Makefile`, issue the following command:

```
$ make
```

Based on the commands contained in the `Makefile`, an executable named `run` will be created.

## Steps 3-4: Request Access to Compute Nodes and Run the Program

In order to run the executable on Ascent's compute nodes, you need to request access to a compute node and then launch the job on the node. The request and launch can be performed using the single batch script, `submit.lsf`. If you open this script, you will see several lines starting with `#BSUB`, which are the commands that request a compute node and define your job (i.e., give me 1 compute node for 10 minutes, charge project `PROJID` for the time, and name the job and output file `add_vec_cpu`). You will also see a `jsrun` command within the script, which launches the executable (`run`) on the compute node you were given. 

The flags given to `jsrun` define the resources (i.e., cpu cores, gpus) available to your program and the processes/threads you want to run on those resources (for more information on using the `jsrun` job launcher, please see challenge [jsrun\_Job\_Launcher](../jsrun_Job_Launcher)).

To submit and run the job, issue the following command:

```
$ bsub submit.lsf
```

### Writing data to the filesystem

Your program might also need to generate data as part of its function. Your `/ccsopen/home/username` directory is not an appropriate location to store this data, as the filesystem is not designed for the kinds of workloads an HPC program usually need to perform when reading and writing data. A more appropriate location is in `/gpfs/wolf` which is a parallel filesystem designed for heavy HPC workloads. Specifically you should be using either your scratch directory `/gpfs/wolf/trn019/<username>` (where `<username>` should be replaced by your actual username) or if you're working in a team you should create a unique directory in the project shared directory `/gpfs/wolf/trn019/proj-shared/`. In the vector addition code, you will see that it is opening a file named `output.txt` which will be created in the current directory. The program will be writing a lot of data in this file, so we need to make sure that we switch to a location in `/gpfs/wolf`. We do this by using the `cd` command in the `submit.lsf` script to make sure that the program starts running from a location in `/gpfs/wolf`. The `programdir` variable is used to store the current directory path before switching to the `/gpfs/wolf` location, so that it can find the `run` executable.

## Monitoring Your Job

Now that the job has been submitted, you can monitor its progress. Is it running yet? Has it finished? To find out, you can issue the command 

```
$ jobstat -u USERNAME
```

where `USERNAME` is your username. This will show you the state of your job to determine if it's running, eligible (waiting to run), or blocked. When you no longer see your job listed with this command, you can assume it has finished (or crashed). Once it has finished, you can see the output from the job in the file named `add_vec_cpu.JOBID`, where `JOBID` is the unique ID given to you job when you submitted it. 
List your directoy to verify that `add_vec_cpu.JOBID` is present by doing: 
```
$ ls
```
You can confirm that it gave the correct results by looking for `__SUCCESS__` in the output file. To see the contents of the file, do: 

```
cat add_vec_cpu.JOBID
```

