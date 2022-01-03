---
layout: post
title: Use and import Python packages and modules inside ParaView (PvPython)
---

I came across a couple of scenarios in which I needed to call a couple of packages inside [PvPython](https://www.paraview.org/Wiki/ParaView/Python_Scripting), the Python client of [ParaView](https://www.paraview.org/), which allows us to automate ParaView tasks (you may take a look at [this automation example]() to see how PvPython works). After trying various techniques in Windows and Linux, my conclusion is that the best solution to this problem is the one suggested [here](https://discourse.paraview.org/t/how-can-i-install-and-import-other-modules-inside-pvpython/3067). But, since this solution is not elaborated and may be difficult to follow, I implement it in this post the way I did it for myself.

Let's first clarify the problem. ParaView comes with its own Python intrepreter, in which you can easily access ParaView Python package. To give it a try, open a terminal and type `pvpython` and then `from paraview.simple import *`. This doesn't give you an error, but if you execute the statement in your local installation of Python, you will face the famous `ModuleNotFoundError`. Now, the problem we want to discuss is the opposite of this one: we cannot use installed packages (like pandas or matplotlib) in PvPython, a scenario that can be very helpful in different applications.

You may think to yourself that the solution is quite obvious and simple: create a Python virtual environment (for example, using [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html) or [venv](https://docs.python.org/3/tutorial/venv.html)) and execute PvPtyhon inside it. I should say that your solution is more or less correct, but it needs a major rearrangement. When you run Python inside a virtual environment, you run the interpreter that is installed inside the virtual environment. So, it means if you run PvPython inside a virtual environment, it cannot see and use the installed packages because it is NOT the interpreter the virtual environment is configured for. So, as I said, the solution is a little bit different from the initial idea: we need to activate a virtual environment while we are inside Python (in this case, inside PvPython). By doing this, the virtual environment launch script will configure the path (I mean `PYTHONPATH`) to point to the installed packages of the environment, and we will be able to use them inside our PvPython automation script.

I think the description is a little bit confusing, no? Let's do that in action and see how it works. We may go on with the virtual environment manager of our choice, but we should keep in mind that it should be easy to activate inside a Python script. As far as I got after a couple of searches, `conda` and `venv` environments are difficult to activate programmatically, so we use `virtualenv` since using it, all we need to do is call an activation script. 

Let's assume that the script we want to run is as simple as this:

```python
import pandas as pd
from paraview.simple import *
```

which means that we are going to use the pandas package in a ParaView script (this was actually the use case for which I started to seek a solution). If you run this script using a normal Python executable, you will face the `ModuleNotFoundError: No module named 'paraview'`, while running it using PvPython results in `ModuleNotFoundError: No module named 'pandas'`. So, it's time for magic. First, we should install [virtualenv](https://virtualenv.pypa.io/en/latest/):

```bash
$ python -m pip install --user virtualenv
```
Then, we need a virtual environment. We can create it in the same location as our script for easier access (in this case, inside a directory called `venv`)

```bash
$ python -m virtualenv venv
```

Now, it's time to activate the environment and install the required packages inside it. We can activate the environment by executing

```bash
$ source ./venv/bin/activate
```
in Linux and 
```cmd
> .\venv\Scripts\activate
```
in Windows. As I said before, we want to use pandas in this example, so let's install it (make sure that the environment is activated):

```bash
$ pip install pandas
```

We are done with the environment, so we can deactivate it and come back to the normal shell. The last step is to modify the script to activate the environment when the it runs. To generalize the code, we pass the path of the environment as an argument in the command line (named `virtual-env`) so that the script can work with different environments if needed. The script should be modified like this:


```python
import sys
if '--virtual-env' in sys.argv:
  virtualEnvPath = sys.argv[sys.argv.index('--virtual-env') + 1]
  # Linux
  virtualEnv = virtualEnvPath + '/bin/activate_this.py'
  # Windows
  # virtualEnv = virtualEnvPath + '/Scripts/activate_this.py'
  if sys.version_info.major < 3:
    execfile(virtualEnv, dict(__file__=virtualEnv))
  else:
    exec(open(virtualEnv).read(), {'__file__': virtualEnv})

# The main script starts from here
import pandas as pd
from paraview.simple import *
```

What the added code does is simple. It looks for the path of the environment and runs the activation script, meaning that the environment gets activated while the script is being interpreted by Python. Make sure to adapt the code if you want to use it under Windows. The script can be executed like this (assuming the script is named `my_script.py` and the environment was created according to the above instruction in a directory called `venv`):

```bash
$ pvpython my_script.py --virtual-env venv
```

You should be able to run the script without any error, meaning that PvPython can see and use the pandas package installed in our virtual environment.