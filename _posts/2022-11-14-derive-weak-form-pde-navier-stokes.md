---
layout: post
title: Weak (variational) formulation of Navier-Stokes equation
---
Computational Fluid Dynamics (CFD) is the field of studying the dynamics of fluid flow using mathematical and computational methods. The fluid flow is usually expressed in the form of Navier-Stokes or Stokes equations, on which appropriate numerical schemes are applied, and the derived system of equations is solved using computers, resulting in the prediction of flow patterns and secondary entities like the shear stress.

The concept of the weak formulation needed for solving partial differential equations (PDEs) numerically using the finite element method was already discussed [here]({% post_url 2020-11-06-derive-weak-form-pde %}). In this post, we have a look at how to derive the weak form of the Navier-Stokes equations, which can be used in available open-source PDE solvers (like [FreeFEM](https://freefem.org/), [FEniCS](https://fenicsproject.org/), and [deal.ii](https://www.dealii.org/)) to simulate fluid flow in any desired domain. 

In its general form, the Navier-Stokes equations describing the flow of an incompressible fluid with constant density $$\rho$$ in the domain $$\Omega \subset \mathbb{R}^{d}$$ (with $$d$$ being the dimension, so 2 or 3) can be written as :

$$
\left\{ {\begin{array}{*{20}{l}}
\displaystyle  {\frac{\partial \mathbf{u}}{\partial t} - {\nabla\cdot}[\nu(\nabla {\mathbf{u}} + \nabla {\mathbf{u}^T})] + ({\mathbf{u}}.\nabla ){\mathbf{u}} + \nabla {\mathbf{p}} = {\mathbf{f}},\quad x \in \Omega ,t > 0,} \\
\displaystyle  {\nabla\cdot{\mathbf{u}} = 0,\quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad x \in \Omega ,t > 0,}
\end{array}} \right.
$$

in which $$\mathbf{u}$$ is the fluid velocity, $$\mathbf{p}$$ is the pressure (which is actually pressure divided by the density), $$\nu = \frac{\mu}{\rho}$$ is the kinematic viscosity (with $$\mu$$ being the dynamic viscosity), and
$$\mathbf{f}$$ is a force term. The equations are conservation of linear momentum and conservation of mass (also called continuity equation), respectively. When $$\nu$$ is constant, the diffusion term can be simplified as:

$$
\text{div} [\nu(\nabla {\bf u}+\nabla {\bf u}^{T})] =\nu (\Delta {\bf u} + \nabla \text{div} {\bf u})=\nu \Delta {\bf u},
$$

which turns the general form into the following:

$$
\left\{ {\begin{array}{*{20}{l}}
\displaystyle  {\frac{\partial \mathbf{u}}{\partial t} - \nu\Delta{\mathbf{u}} + \left( {\mathbf{u} \cdot \nabla } \right) {\mathbf{u}} + \nabla p = {\mathbf{f}},\quad x \in \Omega ,t > 0,} \\
 \displaystyle {\nabla\cdot{\mathbf{u}} = 0,\quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad \quad x \in \Omega ,t > 0,}
\end{array}} \right.
$$

This equation satisfies the incompressibility condition $$\nabla\cdot\mathbf{u}=0$$ and needs proper initial and boundary conditions to be well-posed. The initial condition can be defined as:

$$
{\bf u}({\bf x},0)={\bf u}_{0}({\bf x})\qquad \forall{\bf x}\ \epsilon\ {\bf \Omega,}
$$

where $${\bf u}_{0}$$ is a divergence-free velocity field. Various types of boundary conditions can be applied. For example, if $$\partial \Omega$$ is the boundary of $$\Omega$$, it can be split into 3 distinct boundaries $$\partial \Omega=\Gamma_{1} \cup \Gamma_{2} \cup \Gamma_{3}$$ each of which with a different type. On $$\Gamma_{1}$$, the inlet can be defined as a Dirichlet boundary condition for the velocity for a given velocity profile $${\bf g}$$:

$$
{\bf u} = {\bf g} \quad \text{on } \Gamma_1
$$

On $$\Gamma_2$$, a wall boundary no-slip condition can be considered:

$$
{\bf u} = 0 \quad \text{on } \Gamma_2
$$

On $$\Gamma_3$$, for the outlet condition, a homogeneous Neumann condition on velocity and a zero pressure condition can be defined like:

$$
\frac{\partial {\bf u}}{\partial n} = 0, \quad \mathbf{p} = 0, \quad \text{on } \Gamma_3
$$

with $$n$$ being the normal direction on the boundary $$\partial \Omega$$. Broadly speaking, these boundaries can be grouped into 2 sets:   $$\Gamma_{D} = \Gamma_{1} \cup \Gamma_{2}$$ and $$\Gamma_{N} = \Gamma_{3}$$ for boundaries with Dirichlet and Neumann conditions, respectively.

The Navier-Stokes equations can be written componentwise for individual components of the flow vector field in the Cartesian coordinates. Denoting $$u_i, i=1,\ldots,d$$ (with $$d=2$$ in 2D and $$d=3$$ in 3D), the equation can be presented as:

$$
\left\{ {\begin{array}{*{20}{l}}
\displaystyle  {\frac{\partial {u_i}}{\partial t} - \nu \Delta {u_i} + \mathop \sum \limits_{j = 1}^d {u_j}\frac{\partial {u_i}}{\partial {x_j}} + \frac{\partial p}{\partial {x_i}} = {f_i},\qquad i = 1, \ldots ,d,} \\
\displaystyle  {\mathop \sum \limits_{j = 1}^d \frac{\partial {u_j}}{\partial {x_j}} = 0.}
\end{array}} \right.
$$

For deriving the weak formulation, the first equation of the Navier-Stokes is multiplied by a test function $$v$$ defined on a proper function space V in which the test functions vanish on the Dirichlet boundary:

$$
V = [{\bf H}^{1}_{\Gamma_{D}}(\Omega)]^{d} = \lbrace{\bf V} \in [{\bf H}^{1}(\Omega)]^{d} : {\bf v}|\Gamma_{D} = {\bf 0}\rbrace.
$$

yielding to:

$$
{\mathop{\int}_{\Omega}} {\partial {\bf u} \over \partial t}.{\bf v}\ d\omega- {\mathop{\int}_{\Omega}}\nu\triangle{\bf u.v}d\omega+ {\mathop{\int}_{\Omega}}[({\bf u.\nabla){\bf u].{\bf v}}}d\omega+ {\mathop{\int}_{\Omega}}\nabla p.{\bf v}d\omega= {\mathop{\int}_{\Omega}}{\bf f. v}d\omega.
$$

Applying Green's divergence theory results in:

$$
-\int_{\Omega} \nu \Delta \mathbf{u} \cdot \mathbf{v} d \omega=\int_{\Omega} \nu \nabla \mathbf{u} \cdot \nabla \mathbf{v} d \omega-\int_{\partial \Omega} \nu \frac{\partial \mathbf{u}}{\partial \mathbf{n}} \cdot \mathbf{v} d \gamma
$$

and

$$
\int_{\Omega} \nabla p \cdot \mathbf{v} d \omega=-\int_{\Omega} p \nabla\cdot \mathbf{v} d \omega+\int_{\partial \Omega} p \mathbf{v} \cdot \mathbf{n} d \gamma
$$

Substituting these two equation into the first equation yields to:
$$
\begin{array}{r}
\displaystyle\int_{\Omega} \frac{\partial \mathbf{u}}{\partial t} \cdot \mathbf{v} d \omega+\int_{\Omega} \nu \nabla \mathbf{u} \cdot \nabla \mathbf{v} d \omega+\int_{\Omega}[(\mathbf{u} \cdot \nabla) \mathbf{u}] \cdot \mathbf{v} d \omega-\int_{\Omega} p \nabla\cdot \mathbf{v} d \omega \\
\displaystyle=\int_{\Omega} \mathbf{f} \cdot \mathbf{v} d \omega+\int_{\partial \Omega}\left(\nu \frac{\partial \mathbf{u}}{\partial \mathbf{n}}-p \mathbf{n}\right) \cdot \mathbf{v} d \gamma \quad \forall \mathbf{v} \in V .
\end{array}
$$

The last term of this equation is expressed in accordance to the defined Neumann boundary condition, which vanishes on $$\Gamma_3$$ due to the defined condition. Moreover, this term vanishes on the Dirichlet boundaries due to the properties of the function space $$V$$.

Similarly, the second equation of the Navier-Stokes is multiplied by a test function $$q$$ belonging to the function space $$Q$$, called the pressure space:

$$
Q = {\bf L}^2_0(\Omega) = \lbrace p \in L^2(\Omega) : {\mathop{\int}_{\Omega}} p \ d\omega = 0\rbrace,
$$

resulting in:

$$
{\mathop{\int}_{\Omega}} q \nabla\cdot{\bf u}\ d\omega = 0 \qquad \forall q \in Q.
$$

The last 2 equations are so called weak (variational) forms of the Navier-Stokes equations.