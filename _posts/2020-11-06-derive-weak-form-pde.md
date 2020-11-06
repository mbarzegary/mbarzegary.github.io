---
layout: post
title: Deriving weak formulation of partial differential equations
---

The finite element method doesn't need an introduction, but at the core of this magical method, in its mathematical nature, one challenging step makes it sometimes a bit difficult for newcomers to immediatley jump start and employ finite element to solve partial differential equations (PDEs) numerically. This challenging part is deriving the weak formulation of the PDE, which is indeed one of the very first steps a researcher should take to use the available PDE solvers (like [FreeFEM](https://freefem.org/), [FEniCS](https://fenicsproject.org/), and [deal.ii](https://www.dealii.org/)) to simulate a mathematical model.

Although deriving the weak form of a PDE is relatively simple, finding a good reference that demonstrates how to do it in action for the first time can be a bit difficult. This topic is well covered in most of the finite element books (the ones that discuss the mathematical aspects), but you need to go through a bunch of math to find the most essential steps. In this post, I try to explain this process by deriving the weak form of a reaction-diffusion PDE as an example. The equation we want to deal with is:

$$
\frac{\partial u}{\partial t}=\nabla \cdot (D   \nabla u)- s u
$$

in which, $$u=u(\mathbf{x},t)$$ is the state variable we want to find at each point of space and time. This is also called the strong form of the PDE. To obtain the finite element formulation, the weak form of the PDE is required. In order to get this, we define a space of test functions and then, multiply each term of the PDE by any arbitrary function as a member of this space. The test function space is

$$
\mathcal{V}=\left\{v(\mathbf{x}) | \mathbf{x} \in {\Omega}, v(\mathbf{x}) \in \mathcal{H}^{1}(\Omega), \text { and } v(\mathbf{x})=0 \text { on } \Gamma\right\}
$$

in which the $$\Omega$$ is the domain of interest, $$\Gamma$$ is the boundary of $$\Omega$$, and $$\mathcal{H}^{1}$$ denotes the [Sobolev space](https://en.wikipedia.org/wiki/Sobolev_space) of the domain $$\Omega$$, which is a space of functions whose derivatives are square-integrable functions in $$\Omega$$. The solution of the PDE belongs to a trial function space, which is similarly defined as

$$
\mathcal{S}_{t}=\left\{u(\mathbf{x}, t) | \mathbf{x} \in \Omega, t>0, u(\mathbf{x}, t) \in \mathcal{H}^{1}(\Omega), \text { and } \frac{\partial u}{\partial n}=0 \text { on } \Gamma\right\}.
$$

Then, we multiply each term of the PDE to an arbitrary function $$v \in \mathcal{V}$$:

$$
\frac{\partial u}{\partial t} v=\nabla \cdot (D  \nabla u) v- s u v.
$$

Integrating over the whole domain yields:

$$
\int_{\Omega} \frac{\partial u}{\partial t} v d \omega=\int_{\Omega} \nabla \cdot (D  \nabla u) v d \omega-\int_{\Omega} s u v d \omega.
$$

The diffusion term can be split using the [integration by parts](https://en.wikipedia.org/wiki/Integration_by_parts) technique:

$$
\int_{\Omega} \nabla \cdot (D  \nabla u) v d \omega = \int_{\Omega} \nabla \cdot[v(D  \nabla u)] d \omega-\int_{\Omega} (\nabla v) \cdot(D  \nabla u) d \omega
$$

in which the second term can be converted to a surface integral on the domain boundary by applying the [Green's divergence theory](https://en.wikipedia.org/wiki/Green%27s_theorem):

$$
\int_{\Omega} \nabla \cdot[v(D  \nabla u)] d \omega = \int_{\Gamma} D v \frac{\partial u}{\partial n} d \gamma.
$$

For the temporal term, we use the finite difference method and apply a first-order [backward Euler scheme](https://en.wikipedia.org/wiki/Backward_Euler_method) for discretization, which makes it possible to solve the PDE implicitly:

$$
\frac{\partial u}{\partial t} = \frac{u-u^{n}}{\Delta t}
$$

where $$u^n$$ denotes the value of the state variable in the previous time step (or initial condition for the first time step). Inserting all these into the integral form yields:

$$
\int_{\Omega} \frac{u-u^{n}}{\Delta t} v d \omega=\int_{\Gamma} D v  \frac{\partial u}{\partial n} d \gamma-\int_{\Omega} D  \nabla u \cdot \nabla v d \omega-\int_{\Omega} s u v d \omega.
$$

The surface integral is zero because there is a no-flux boundary condition on the boundary of the computational domain (defined in the trial function space). By reordering the equation, we get:

$$
\int_{\Omega} \frac{u}{\Delta t} v d \omega+\int_{\Omega} D \cdot \nabla \cdot u \nabla v d \omega+\int_{\Omega} s u v d \omega=\int_{\Omega} \frac{u^{n}}{\Delta t} v d \omega
$$

which is the weak form of the PDE and can be written as (by multiplying to $$\Delta t$$):

$$
\int_{\Omega} {u} v d \omega+\int_{\Omega} \Delta t D  \nabla u \cdot  \nabla v d \omega+\int_{\Omega} \Delta t s u v d \omega=\int_{\Omega} {u^{n}} v d \omega.
$$

So, the problem is finding a function $$u(t) \in \mathcal{S}_{t}$$ such that for all $$v \in \mathcal{V}$$ the above equation would be satisfied. Defining and solving this problem is simple and straightforward in a wide variety of available finite element PDE solvers such as FreeFEM.


