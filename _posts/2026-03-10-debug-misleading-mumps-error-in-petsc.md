---
layout: post
title: Debugging a misleading MUMPS error in PETSc&#58; it was not about memory
---

I came across a very frustrating issue recently while setting up a fresh [Firedrake](https://www.firedrakeproject.org/) installation on my workstation, and since the debugging process turned out to be quite instructive (and the root cause was something that can bite anyone doing computational science on Linux), I decided to document it here. So, if you've ever faced `FACTOR_OUTMEMORY` from MUMPS and thought "but I have plenty of RAM!", this post might be useful for you.

### The problem

I have a conjugate heat transfer code built with Firedrake, which assembles a nonlinear system and solves it with Newton's method using a direct (LU) solver via [MUMPS](https://mumps-solver.org/). The code had been running reliably for a while on an older Firedrake/PETSc installation, and it ran perfectly on my colleague's machine with the latest version. So, I did a fresh Firedrake install on my machine, ran the same code, and got this:

```
0 SNES Function norm 5.268268276857e+00
  Linear firedrake_1_ solve did not converge due to DIVERGED_PC_FAILED iterations 0
                 PC failed due to FACTOR_OUTMEMORY
  Nonlinear firedrake_1_ solve did not converge due to DIVERGED_LINEAR_SOLVE iterations 0
```

The solver didn't even get through a single linear solve. MUMPS reported it ran out of memory during LU factorization. The thing is, my machine has plenty of RAM (about 128GB plus over 200GB of swap space), the problem size is moderate, and this exact code works elsewhere. So, something was clearly wrong with my installation.

### The debugging journey

#### Trying to throw more memory at MUMPS

My first instinct was the standard advice for `FACTOR_OUTMEMORY`: increase the memory relaxation parameter. MUMPS has a parameter called `icntl_14` that tells it to allocate a larger percentage above its initial memory estimate:

```python
solver_parameters = {
    "ksp_type": "preonly",
    "pc_type": "lu",
    "pc_factor_mat_solver_type": "mumps",
    "mat_mumps_icntl_14": 200,
    "mat_mumps_icntl_23": 4000,
}
```

No effect. Same error. So, I thought maybe MUMPS specifically is broken in my build.

#### Trying alternative solvers

Since PETSc can use different direct solver backends, I switched to [SuperLU_DIST](https://github.com/xiaoyeli/superlu_dist), a completely independent direct solver with a different codebase:

```
0 SNES Function norm 5.268268276857e+00
double free or corruption (out)
double free or corruption (out)
```

Memory corruption! Two different solver packages, two different failure modes. That was the moment I started suspecting the problem was deeper than any individual solver.

#### Verifying MUMPS on a simple problem

To check whether MUMPS was completely broken or not, I ran a simple Poisson problem, a 50×50 mesh, Laplacian with Dirichlet BCs, solved with MUMPS on 4 MPI processes:

```python
from firedrake import *

mesh = UnitSquareMesh(50, 50)
V = FunctionSpace(mesh, "CG", 1)
u = TrialFunction(V)
v = TestFunction(V)

a = inner(grad(u), grad(v)) * dx
L = Constant(1.0) * v * dx
bc = DirichletBC(V, 0.0, "on_boundary")

u_sol = Function(V)
solve(a == L, u_sol, bcs=bc, solver_parameters={
    "ksp_type": "preonly",
    "pc_type": "lu",
    "pc_factor_mat_solver_type": "mumps",
})
print("Solve succeeded, norm =", norm(u_sol))
```

It worked perfectly, using 12 MB total. So, MUMPS wasn't completely broken; it could factor small matrices just fine. The issue was specific to my actual problem.

#### Running serial to get a better error

I then tried running the actual code on a single MPI process instead of four, and got a different error:

```
0 SNES Function norm 5.268268276857e+00
  Linear firedrake_1_ solve did not converge due to DIVERGED_PC_FAILED iterations 0
                 PC failed due to FACTOR_NUMERIC_ZEROPIVOT
```

Interesting! In parallel, MUMPS was reporting `OUTMEMORY`, but in serial it reported `ZEROPIVOT`. It turns out MUMPS sometimes conflates these errors in parallel mode, which is one of the reasons `FACTOR_OUTMEMORY` is such a frustrating and misleading message to debug.

#### The breakthrough: `make check`

After installing PETSc, there's a validation step that I think many people skip:

```bash
make PETSC_DIR=/path/to/petsc PETSC_ARCH=arch-firedrake-default check
```

I ran it and got something very revealing:

```
C/C++ example src/snes/tutorials/ex19 run successfully with MUMPS
C/C++ example src/snes/tutorials/ex19 run successfully with HYPRE

Possible problem with ex19 running with SuiteSparse, diffs above
Possible problem with ex19 running with SuperLU_DIST, diffs above
```

MUMPS and HYPRE passed, but [SuiteSparse](https://people.engr.tamu.edu/davis/suitesparse.html) and SuperLU_DIST both failed, reporting zero SNES iterations instead of converging in two iterations. The solvers weren't crashing; they were silently returning without doing any work, or returning garbage that the nonlinear solver interpreted as already converged. This was a clear signal. Two out of four solver backends were broken at the PETSc level, on the standard test problem. The build itself was defective.

### The root cause: mixed BLAS libraries

All of PETSc's direct solvers ultimately depend on [BLAS](https://www.netlib.org/blas/) and [LAPACK](https://www.netlib.org/lapack/) for the dense linear algebra operations inside the factorization. If the BLAS layer is broken or inconsistent, every solver built on top of it will misbehave, but they'll each misbehave differently, and that's what makes this so confusing to diagnose from the top.

I checked what BLAS libraries PETSc was actually linking against at runtime:

```bash
ldd /path/to/petsc/arch-firedrake-default/lib/libpetsc.so | grep -i blas
```

```
libblas.so.3 => /lib/x86_64-linux-gnu/libblas.so.3 (0x00007f6728182000)
libopenblas.so.0 => /lib/x86_64-linux-gnu/libopenblas.so.0 (0x00007f6722db0000)
```

And there it was. PETSc was linking to **two different BLAS implementations simultaneously**: the system's generic reference BLAS (`libblas.so.3`) and [OpenBLAS](http://www.openmathlib.org/OpenBLAS/) (`libopenblas.so.0`). These are different libraries with potentially different threading models, memory layouts, and internal conventions. When both are loaded into the same process, different parts of the computation can end up calling different implementations, leading to subtle data corruption.

This is the kind of bug that doesn't cause a clean crash. It corrupts numerical results just enough that some solvers produce garbage, others hit unexpected zero pivots, and others allocate the wrong amount of memory because the matrix data they're working with has already been silently corrupted. And the reason this only affected my machine was the state of my system's BLAS alternatives. On Ubuntu and Debian, `/lib/x86_64-linux-gnu/libblas.so.3` is managed by the `update-alternatives` system and can point to different backends. My system had both reference BLAS and OpenBLAS installed, and PETSc's configure picked up both through different dependency chains. My colleague's machine either had a cleaner BLAS setup or PETSc resolved to a single library.

### The fix

The solution is to ensure PETSc uses exactly one BLAS implementation, and the most reliable way to do that is to let PETSc download and compile its own. By passing `--download-openblas`, PETSc builds OpenBLAS from source with the same compiler and flags it uses for everything else. MUMPS, SuperLU_DIST, SuiteSparse, and ScaLAPACK all link against this single, consistent BLAS. No mixing, no conflicts.

After reinstalling, you can verify the fix by checking the linked libraries again:

```bash
ldd /path/to/petsc/arch-firedrake-default/lib/libpetsc.so | grep -i blas
```

You should see a single OpenBLAS library pointing to PETSc's own build directory, not to `/lib/x86_64-linux-gnu/`. And of course, run `make check` again to confirm all solver backends pass.

### Lessons learned

I want to wrap up with a few takeaways that I think can be useful for anyone in the field of scientific computing who may run into similar issues:

**`FACTOR_OUTMEMORY` often isn't about memory.** MUMPS uses this error code as a catch-all for various factorization failures, especially in parallel. If you've verified that you have enough RAM and the problem isn't absurdly large, look deeper. In my case, the actual problem had nothing to do with memory allocation at all.

**When multiple solver backends fail, look below them.** If MUMPS, SuperLU, and SuiteSparse all break on the same problem, the bug almost certainly isn't in any of those packages. It's in the shared infrastructure underneath such as BLAS, LAPACK, MPI, or the compiler runtime. The individual error messages from each solver will be different and individually misleading; the pattern of widespread failure is the real clue.

**Always run `make check` after building PETSc.** It takes a few seconds and tests the critical solver backends against known-good reference outputs. It can catch the failures right away and save quite a lot of time debugging at the application level.

**Mixed BLAS libraries are a silent killer.** On modern Linux distributions, it's easy to end up with multiple BLAS implementations installed (reference BLAS, OpenBLAS, MKL, ATLAS). The `update-alternatives` system is supposed to manage this, but when a build system like PETSc's configure resolves dependencies through multiple paths, it can pull in more than one. The resulting corruption is numerical, not structural, meaning that your code runs, your matrices assemble, your solvers launch, but the numbers that come back are wrong. Letting PETSc build its own BLAS with `--download-openblas` eliminates this entire class of problems.

To be honest, this was something that took me a while to figure out, but the debugging process itself was a good reminder that in scientific computing, the error you see is rarely the error you have. In my case, the actual failure chain was something like: mixed BLAS → corrupted dense operations → MUMPS interprets corruption as out-of-memory → PETSc reports `DIVERGED_PC_FAILED`. Four levels of abstraction between the root cause and the error message I was staring at. Yes, it can get that nightmarish (of course not as bad as the nostalgic [DLL hell](https://en.wikipedia.org/wiki/DLL_Hell) in Windows, but close enough!).

In the next post, I will explain how to have a fresh and clean installation of PETSc and Firedrake from scratch, documenting the procedure step by step to hopefully save you from running into issues like this one.
