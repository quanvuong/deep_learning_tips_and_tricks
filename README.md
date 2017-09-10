While there are plenty of resources to understand deep learning, there is not one that explains the tricks to make daily interaction with neural nets easier. This is my collections of tricks. Suggestions and pull requets are very welcome :). The discussion assumes you use Python.

The tricks include the easiest way to do something based on my experience, not complete explanations of all features available.

I'm currently an AI researcher at NYU Shanghai.

**Profiling Speed**

There are two main profiling tools, cProfiler and [line_profiler](https://github.com/rkern/line_profiler). The first helps you find time-consuming functions and the second what lines within the functions take the longest.

cProfiler comes with Python. To install line_profiler, do: 

`pip install line_profiler`

To use cProfiler, do: 

`python -m cProfile -o cProfile_results script_to_profile.py` 

This command runs your script and outputs the profiling results to file `cProfile_results`. I use Pycharm to view the profiling result file. You can profile directly from within Pycharm, but I don't recommend it because Pycharm doesn't free memory after the script finishes.

Using Pycharm to view the profiling results, we have something along [this line](https://i.imgur.com/Fr4RIps.png?1).

There are two main things to look for, Time and Own Time. Time indicates the time spent within a function. Own Time also indicates time spent within the function (time spent within functions called by that function are not taken into account). You want to optimize functions who either have high Time or Own Time or both.

After you have identify a function to optimize, let's use line_profiler to determine what's taking our function so long to run. To run line_profiler, add the decorator `@profile` to the function ([example](https://i.imgur.com/mOQ6h5q.png)), and do:

`kernprof -l -v script_that_calls_the_function.py`

This command runs your script and outputs the profiling result to your terminal. A typical output looks like [this](https://i.imgur.com/bAEfjcU.png). You can how see long each line within your function takes to run, and optimize accordingly.
  
**Profiling Memory Usage**

If you suspect your script is leaking memory, use [memory_profiler](https://github.com/fabianp/memory_profiler).

To install memory_profiler, do: 

`pip install line_profiler`

To check if your script is leaking memory, do:

`mprof run script_to_profile.py`

This command runs your script and records memory usage into a file. To view the memory usage, do:
 
`mprof plot`

This command shows memory usage over time. In this [example](https://i.imgur.com/5TuHdct.png), we can see that memory usage increases linearly with time, which indicates memory leak.

**SLURM - Job Scheduler for HPC**

If you're running your code on High Performance Cluster, you're probably using [SLURM](https://slurm.schedmd.com/). SLURM seems intimidating at first, but it's a real time saver. It allows you to specify the settings with which your job should be run in (how much memory, gpu). It queues your jobs when requested resources are not available and send you email notifications when your jobs finish.

To use SLURM, you write a text file, which includes all the information needed to run your job and submit the file to SLURM.

A typical SLURM job submission file looks like:

```#!/bin/sh -l
#SBATCH --job-name=name_of_job
#SBATCH --output=output.slurm  # stdout is redirected to this file, i.e. your print statement will output to this file.

#SBATCH --time=24:00:00  # maximum time your script will run for. The format is DD-HH:MM:SS.
#SBATCH --cpus-per-task=1  # How many CPUs ?
#SBATCH --mem=20000mb  # How much memory ?
#SBATCH --mail-type=BEGIN,END,FAIL,REQUEUE,STAGE_OUT  # What end to send email notification for. 
#SBATCH --mail-user=email@address.com  # Where to send email notification.
#SBATCH --gres=gpu:k80:1  # To request GPU, include this lien ? You can specify kind and number of GPU.

Command to run your script goes here. 
I typically activate my conda environment. cd to the directory containing my Python script and run it.
```

To submit the job to SLURM, do:

`sbatch text_file_name`

You'll want to add `output.slurm` to your `.gitignore` file.

A really handy feature of SLURM is [job array](https://slurm.schedmd.com/job_array.html). It allows you to run identical version of your script with different environment variables. This is super useful for parameter tuning or average results over seeds.

To use job array, add this to your slurm text file:

`#SBATCH --array=1-10`

Adding this will run your script 10 times in parallel. The environment variable `SLURM_ARRAY_TASK_ID` in each script will have different values (from 1 to 10 in this case). You can then set different parameter setting for each parallel run based on this environment variable.

Also remember to change `#SBATCH --output=output.slurm` to `#SBATCH --output=%a_output.slurm`. This redirect the stdout of each parallel run to different file. `%a` will be replaced by the corresponding `SLURM_ARRAY_TASK_ID` for each run.

Another useful trick is to add the following line into your `~/.bashrc`.

`alias q='echo ; squeue --user=qhv200 --format="%.16i %.9P %.110j %.8M %.10T %R"; echo;'`

You can then view your submitted jobs, their status and most relevant info by issuing `q` in the terminal. [Example](https://i.imgur.com/MKgDK0M.png).