---
layout: post
title: Building OpenFOAM in a Conda virtual environment
---

In [previous post]({% post_url 2020-10-10-using-conda-as-a-build-environment %}), I explained why a Conda environment would be interesting for building software programs in an isolated box. In this post, as an example, I describe how it works if we want to build [OpenFOAM](https://openfoam.org/) in this manner. As I said, the main advantage of doing this is the possibility of building/installing software without the root privilege, a scenario that happens frequently while working with clusters and supercomputers. A root previlege is required for following the [official build procedure for OpenFOAM](https://openfoam.org/download/source/).

The following procedure is independent of the underlying Linux installation, and only a tool to download Miniconda and a text editor suffice (here I have used `wget` and `nano` for these purposes). Also, it's worth mentioning that all the paths are taken from my system, so throughout the tutorial, make sure to double-check all the paths/directories to make sure that they match your system configuration. The tutorial is prepared for `bash`, but the approach should be more or less the same for other shells and can be modified accordingly

Okay, the first step is to download and install Conda. I personally prefer [Miniconda](https://docs.conda.io/en/latest/miniconda.html) over a full [Anaconda](https://www.anaconda.com/) installation, which usually leaves you with a bunch of unused packages and a tremendous amount of disk usage. Miniconda provides you with a minimal installation of Conda, and you can use it to download the necessary packages only. So, let's download the installer and run it.

```bash
$ cd
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ bash Miniconda3-latest-Linux-x86_64.sh
```

Follow the onscreen instruction. In the last step, you may choose to auto-initialize the base environment every time you open your BASH session. Doing this saves you from doing some extra steps to add the `conda` executable path to the `PATH` environment variable, so it's better to let the installer add the appropriate statements to your profile's `.bashrc`. If not, execute this command after adding the bin folder to `PATH` and restarting your session:

```bash
$ conda init bash
```

So, now, Conda is fully under our control to follow our orders. Let's create a virtual environment. We call it "gcc" just because we want to use [GNU Compiler Collection](https://gcc.gnu.org/) to build things.

```bash
$ conda create -n gcc
```

By activating the environment, we isolate ourselves from the rest of the system. Let's do it.

```bash
$ conda activate gcc
```

Okay, it's time to install all the prerequisites in the virtual environment. It will be as easy as calling `conda install` command.

```bash
$ conda activate gcc
$ conda install gcc_linux-64
$ conda install gxx_linux-64
$ conda install gfortran_linux-64
$ conda install cmake
$ conda install openmpi
$ conda install flex
$ conda install bison
$ conda install boost
$ conda install curl
$ conda install git
```

Although this is something done automatically, let's double-check that the gcc bundled scripts are run to change the build environment variables (like `$CC`) to point to the compilers installed inside the virtual environment. 

```bash
$ $CONDA_PREFIX/etc/conda/activate.d/activate-gcc_linux-64.sh
$ $CONDA_PREFIX/etc/conda/activate.d/activate-gxx_linux-64.sh
$ $CONDA_PREFIX/etc/conda/activate.d/activate-gfortran_linux-64.sh
$ $CONDA_PREFIX/etc/conda/activate.d/activate-binutils_linux-64.sh
```

In a standard world, everything should be ready till now, and software build routines should look for the environment variables to find the appropriate compilers. For example, a standard build command in a makefile should be something like `$CC source.c` and not `gcc source.c`. But, we don't live in such a world, and it means that not every single piece of software follows this principle. Unfortunately, OpenFOAM is not an exception in this regard, and its build routine calls the compilers' executable directly. This creates an issue for our approach because it indeed calls the system compilers (if they are installed) and not the ones we have installed in our virtual environment. This is highly dangerous for our case, so let's create some soft links to ask BASH to compensate. 

```bash
$ cd $CONDA_PREFIX/bin/
$ ln -s x86_64-conda_cos6-linux-gnu-gcc gcc
$ ln -s x86_64-conda_cos6-linux-gnu-g++ g++
$ ln -s x86_64-conda_cos6-linux-gnu-c++ c++
$ ln -s x86_64-conda_cos6-linux-gnu-gfortran gfortran
$ ln -s x86_64-conda_cos6-linux-gnu-ld ld
$ ln -s x86_64-conda_cos6-linux-gnu-as as
$ ln -s x86_64-conda_cos6-linux-gnu-nm nm
$ ln -s x86_64-conda_cos6-linux-gnu-cpp cpp
$ ln -s x86_64-conda_cos6-linux-gnu-ld.bfd ld.bfd
$ ln -s x86_64-conda_cos6-linux-gnu-ld.gold ld.gold
$ ln -s x86_64-conda_cos6-linux-gnu-ar ar
cd -
```

Now, if you run `which gcc` or `which g++`, you should see it pointing to the virtual environment compilers. Yes? So, let's go on by cloning the source codes and do the actual build.

```bash
$ mkdir openfoam
$ cd openfoam/
$ git clone https://github.com/OpenFOAM/OpenFOAM-8.git
$ git clone https://github.com/OpenFOAM/ThirdParty-8.git
```
OpenFOAM comes with its own way of initializing things in BASH, and we need to follow its principle. No problem man, we will.

```bash
$ source OpenFOAM-8/etc/bashrc
$ cd ThirdParty-8/
```

Cool. It's time to start the build by typing `./Allwmake`, no? No, it's not. If you proceed, you will face lots of compiling/linking errors. It's because we have installed all the required libraries inside the Conda environment (and not the known system directories like `/usr/include` and `/usr/lib`). We should tell the compiler where to seek them. This is also something that should be solved in a standard way, but (not surprisingly) going for conventional things doesn't work for OpenFOAM. As a result, we edit the config files directly. A horrible idea, but something that usually works for such issues. Let's edit the "Scotch" build config to tell it where to find the "zlib" library. Run this command:
```bash
$ nano etc/wmakeFiles/scotch/Makefile.inc.i686_pc_linux2.shlib-OpenFOAM
```
and append these lines to its end:
```
CFLAGS  += -I /home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/include 
LDFLAGS += -L /home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/lib
```
Make sure to adapt the directories to something that matches your system paths. And now, we are ready to start building the third-party tools for OpenFOAM (which is literally just the Scotch library as we don't intend to build ParaView).

```bash
$ ./Allwmake
```

Then, we refresh the OpenFOAM variables and go to the main source directory.

```bash
$ wmRefresh
$ cd ../OpenFOAM-8/
```

We should perform some more modifications here (pay attention to the paths and adapt them before saving the files). Run:

```bash
$ nano wmake/rules/linux64Gcc/c
```
and add this to the end:
```
CFLAGS += -I /home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/include -I /home/mojtaba/miniconda3/pkgs/flex-2.6.4-ha10e3a4_1/include/
```
Do a similar thing for the C++ compiler flags:
```bash
$ nano wmake/rules/linux64Gcc/c++
```
and add:
```bash
c++FLAGS += -I /home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/include -I /home/mojtaba/miniconda3/pkgs/flex-2.6.4-ha10e3a4_1/include/
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/ThirdParty-8/platforms/linux64Gcc/gperftools-svn/lib 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/OpenFOAM-8/platforms/linux64GccDPInt32Opt/lib/openmpi-system 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/ThirdParty-8/platforms/linux64GccDPInt32/lib/openmpi-system 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/site/8/platforms/linux64GccDPInt32Opt/lib 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/OpenFOAM-8/platforms/linux64GccDPInt32Opt/lib 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/ThirdParty-8/platforms/linux64GccDPInt32/lib 
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/openfoam/OpenFOAM-8/platforms/linux64GccDPInt32Opt/lib/dummy
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/lib
c++FLAGS += -Wl,-rpath-link,/home/mojtaba/miniconda3/envs/gcc/lib
```
And the last one, which is a little bit tricky (because `make` is sensitive to indentation and you should pay attention to not ruin the structure of the file), is to help the linker find "zlib" again. Run this:

```bash
$ nano src/OpenFOAM/Make/options
```
and modify the `LIB_LIBS` variable to look like this (it's safer to gather them all in one line to avoid mistakes):
```
LIB_LIBS = $(FOAM_LIBBIN)/libOSspecific.o -L$(FOAM_LIBBIN)/dummy -lPstream -L /home/mojtaba/miniconda3/pkgs/zlib-1.2.11-h7b6447c_3/lib -lz
```

Okay, so we are all set to continue. Let's run the build command:

```bash
$ ./Allwmake -j 8
```

You should adjust the number of CPU cores you want to employ (which implies the number of concurrent compile commands running at the same time). The more, the better. I had 8 free cores, so I have entered 8 here. It may take up to several hours if you run it without the `-j` flag, but for example, with 8 parallel cores, it took less than one and a half hours to finish.

If it succeeds (which should be the case), we can go on to test if it really works by copying one of the OpenFOAM tutorials and running it.

```bash
$ cd ..
$ mkdir tests
$ cd tests/
$ cp $FOAM_TUTORIALS/incompressible/simpleFoam/pitzDaily . -r
$ cd pitzDaily/
$ blockMesh
$ simpleFoam
```

You should see the OpenFOAM simulation running. And that's it. Relatively straightforward, no? To use it next time you open a BASH session, you should activate the virtual environment again (`conda activate gcc`) and run the initialization script (`source OpenFOAM-8/etc/bashrc`), and there you go.