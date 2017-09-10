While there are plenty of resources to understand deep learning, there is not one that explains the tricks to make daily interaction with neural nets easier. This is my collections of tricks. Suggestions and pull requets are very welcome :). The discussion assumes you use Python.

The tricks include the easiest way to do something based on my experience, not complete explanations of all features available.

**Speed Profiling**

There are two main profiling tools, cProfiler and line_profiler. The first helps you find time-consuming functions and the second what lines within the functions take the longest.

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
  
Memory Profiling