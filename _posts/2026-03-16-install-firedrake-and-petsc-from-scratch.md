---
layout: post
title: Installing Firedrake and PETSc from scratch&#58; letting PETSc handle its own dependencies
---

In the [previous post]({% post_url 2026-03-10-debug-misleading-mumps-error-in-petsc %}), I described a frustrating debugging journey where mixed BLAS libraries silently corrupted solver outputs after a Firedrake installation. I said a clean installation guide would follow, so here it is. This post documents the step-by-step procedure I ended up following to get a working Firedrake installation on a local Linux workstation, building PETSc from source with its own MPI and dependencies needed to run Firedrake.

The [official Firedrake installation guide](https://www.firedrakeproject.org/install.html) is well-written and covers the standard path nicely. On a supported Ubuntu system, most of the dependencies come from the system package manager, and the whole process is fairly smooth. But if you're on a non-standard distribution, or if your system has a messy state of libraries (multiple BLAS backends, multiple MPI implementations, and so on), the standard path can lead to subtle issues that are very hard to diagnose, as I described in the previous post. The approach I describe here is more self-contained: we let PETSc download and compile most of its dependencies from source, including MPI and BLAS, so we get a consistent build that doesn't depend on whatever happens to be installed on the system.

### Why let PETSc download its own MPI?

This is a question that comes up often, and the answer really depends on where you're running your code.

On an HPC cluster, you should almost always use the system MPI. The MPI implementation provided by the system administrators is typically tuned for the specific interconnect hardware (InfiniBand, Slingshot, etc.), and building your own MPI would mean losing all of that optimization. The system MPI is also the one that the job scheduler knows how to launch properly. Using anything else on a cluster is asking for trouble.

On a local workstation, though, the story is different. Most workstations ship with a generic OpenMPI installed via the package manager, and it's configured for the common case, not for your specific scientific computing stack. If you also happen to have another MPI implementation installed (maybe from a previous project, pulled in as a dependency of something else, or came along with something like NVIDA HPC SDK), things can get confusing fast. The dynamic linker may pick up the wrong `libmpi.so` at runtime, `mpicc` might point to a different MPI than the one PETSc was built with, and pip-installed packages like mpi4py can silently compile against the wrong implementation.

By passing `--download-mpich` to PETSc's configure, you get an MPI that lives entirely inside the PETSc directory tree. There's no ambiguity about which MPI is being used, and all the downstream packages (MUMPS, ScaLAPACK, pnetcdf, etc.) are guaranteed to be built against the same one. For local development and testing, this is a perfectly reasonable trade-off: you give up the system-level tuning (which doesn't matter much on a workstation anyway) in exchange for a build that's self-contained and reproducible.

### The challenges of using PETSc's own MPI

That said, having PETSc download its own MPICH does introduce a few complications that are worth knowing about.

The first issue is that packages like pnetcdf, which PETSc also downloads and compiles, may accidentally link against the system MPI instead of PETSc's MPICH during the build. This can manifest as `undefined reference` errors related to DSO (dynamic shared object) resolution in the PETSc build logs. The fix is to set a linker flag before running configure:

```bash
export LDFLAGS="-Wl,--copy-dt-needed-entries"
```

This tells the linker to consider the dependencies of shared libraries when resolving symbols, which prevents it from silently dropping the connection to the correct MPI. You then pass `LDFLAGS=$LDFLAGS` to PETSc's `./configure` so it picks up this flag.

The second issue shows up later, at the Firedrake/pip level. When you install Firedrake via pip, packages like mpi4py get compiled and linked. If the system's OpenMPI is on the default library search path and PETSc's MPICH is not, mpi4py will find and link against the wrong one. You'll then see symptoms like `RuntimeWarning: suspicious MPI execution environment` (because OpenMPI environment variables like `PMI_SIZE` confuse an MPICH-linked mpi4py) or `ImportError: undefined symbol: MPI_UNWEIGHTED` (because the runtime finds the wrong `libmpi.so`).

The fix is straightforward: make sure PETSc's lib and bin directories are at the **front** of `LD_LIBRARY_PATH` and `PATH`, not appended at the end:

```bash
export LD_LIBRARY_PATH=$PETSC_DIR/$PETSC_ARCH/lib:$LD_LIBRARY_PATH
export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
```

The ordering matters. If the system MPI directories appear first, the dynamic linker will find them before PETSc's, and you'll be back to the same problem.

### The installation procedure

Alright, with that context out of the way, here's the full procedure. I'll assume you're on a Linux system with Python 3.10 or later, a C/C++/Fortran compiler, git, curl, and the standard build tools installed. Everything below is run from a single working directory.

#### Step 1: Get the Firedrake configuration helper

Firedrake provides a utility script called `firedrake-configure` that helps determine the right PETSc version and configuration flags. Download it:

```bash
curl -O https://raw.githubusercontent.com/firedrakeproject/firedrake/release/scripts/firedrake-configure
```

This script doesn't install anything by itself; it just emits configuration options for the various steps.

#### Step 2: Clone PETSc

Clone the specific PETSc version that matches the current Firedrake release:

```bash
git clone --branch $(python3 firedrake-configure --show-petsc-version) https://gitlab.com/petsc/petsc.git
cd petsc
```

#### Step 3: Set up PETSc environment variables

These variables will be used throughout the rest of the installation. Set them once and make sure they're correct:

```bash
export PETSC_DIR=$(pwd)
export PETSC_ARCH=arch-firedrake-default
export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
```

#### Step 4: Configure PETSc

First, set the linker flag to avoid DSO resolution issues with MPICH (as discussed above):

```bash
export LDFLAGS="-Wl,--copy-dt-needed-entries"
```

Now run PETSc's configure. The flags below are essentially what `firedrake-configure --no-package-manager --show-petsc-configure-options` would give you, plus `--download-mpich` and `--download-openblas` to ensure we have a consistent MPI and BLAS:

```bash
./configure LDFLAGS=$LDFLAGS \
  --with-c2html=0 \
  --with-debugging=0 \
  --with-fortran-bindings=0 \
  --with-shared-libraries=1 \
  --with-strict-petscerrorcode \
  PETSC_ARCH=arch-firedrake-default \
  --COPTFLAGS='-O3 -march=native -mtune=native' \
  --CXXOPTFLAGS='-O3 -march=native -mtune=native' \
  --FOPTFLAGS='-O3 -march=native -mtune=native' \
  --download-bison \
  --download-fftw \
  --download-hdf5 \
  --download-hwloc \
  --download-metis \
  --download-mpich \
  --download-mumps \
  --download-netcdf \
  --download-pnetcdf \
  --download-ptscotch \
  --download-openblas \
  --download-scalapack \
  --download-suitesparse \
  --download-superlu_dist \
  --download-zlib \
  --download-hypre
```

A few notes on these flags. The `--with-debugging=0` combined with `-O3 -march=native -mtune=native` gives you an optimized build. If you need to debug PETSc-level issues, switch to `--with-debugging=1` and remove the optimization flags. The `--download-openblas` is what prevents the mixed BLAS nightmare I described in the previous post (PETSc builds its own OpenBLAS from source and links everything against it), so there's no chance of accidentally picking up the system's reference BLAS alongside a different OpenBLAS.

This step will take a while, as PETSc downloads and compiles all the listed packages.

#### Step 5: Build and verify PETSc

```bash
make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH all
make PETSC_DIR=$PETSC_DIR PETSC_ARCH=$PETSC_ARCH check
```

Do not skip `make check`. As I showed in the previous post, it takes a few seconds and can catch broken solver backends immediately, saving you from hours of debugging at the application level.

#### Step 6: Create a virtual environment

Go back to the parent directory and create a Python virtual environment:

```bash
cd ..
python3 -m venv venv-firedrake
. venv-firedrake/bin/activate
```

Purge the pip cache to make sure no stale binary wheels (potentially linked against the wrong libraries) sneak in:

```bash
pip cache purge
```

#### Step 7: Set the environment for Firedrake

Export the environment variables that Firedrake needs to find PETSc and its dependencies:

```bash
export $(python3 firedrake-configure --show-env)
export LD_LIBRARY_PATH=$PETSC_DIR/$PETSC_ARCH/lib:$LD_LIBRARY_PATH
export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
```

The first line sets `PETSC_DIR`, `PETSC_ARCH`, `CC=mpicc`, `CXX=mpicxx`, and `HDF5_MPI=ON`. The next two lines are critical: they ensure that PETSc's MPICH libraries and executables are found **before** the system ones. Without this (or with the wrong ordering), pip will compile mpi4py against whatever MPI the linker finds first, which is usually the system's OpenMPI. That's exactly the scenario I described earlier in this post, where you end up with `undefined symbol: MPI_UNWEIGHTED` errors or suspicious PMI warnings at runtime, because mpi4py was linked against a different MPI than the one PETSc was built with. Note that `firedrake-configure --show-env` assumes you're running this from the parent directory of the `petsc` folder, which is where we are if you've been following along.

#### Step 8: Pin setuptools

At the time of writing, there's a compatibility issue with recent setuptools versions that can break the build of some packages. Pin it:

```bash
echo 'setuptools<81' > constraints.txt
export PIP_CONSTRAINT=constraints.txt
```

#### Step 9: Install Firedrake

```
pip install --no-binary h5py 'firedrake[check,vtk,netgen]'
```

The `--no-binary h5py` is important: it forces h5py to be compiled from source against the HDF5 that PETSc built (which is MPI-aware), rather than using a pre-built binary wheel that may be linked against a different HDF5 or MPI.

The `[check,vtk,netgen]` part installs optional dependencies. The `check` group is needed to run the verification tests. Drop `vtk` or `netgen` if you don't need them (although `vtk` is quite essential if you would like to visualize your simulation results in ParaView).

#### Step 10: Verify the installation

```bash
firedrake-check
```

This runs a small set of unit tests that exercise the main functionality. If everything passes, you're good to go.

### Summary

The key idea behind this installation approach is to let PETSc manage as many of its own dependencies as possible: MPI, BLAS, HDF5, MUMPS, and all the rest. On a local workstation, this eliminates the class of problems where the dynamic linker silently picks up the wrong shared library from the system, which, as I documented in the previous post, can lead to spectacularly misleading error messages. The trade-off is a longer initial build time, but you only do this once (or at least not very often), and the result is a self-contained installation that's much easier to reason about when things go wrong.

If you're deploying on an HPC system, the approach should be different: use the system MPI and as many system-provided libraries as possible, since they're tuned for the hardware. The Firedrake project maintains a [wiki](https://github.com/firedrakeproject/firedrake/wiki/HPC-installation) with community-contributed instructions for various HPC systems, which is a good starting point for that scenario.
