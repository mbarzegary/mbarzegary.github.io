---
layout: post
title: Solving Stefan (moving-boundary) formulation of a diffusion problem using numerical and symbolic computing - Part 1
---

Moving-boundary problems are a subset of the general concept of boundary-value problems which not only require the solution of the underlying partial differential equation (PDE), but also the determination of the boundary of the domain (or sub-domains) as part of the solution.  Moving-boundary problems are usually referred to as Stefan problems and can be used to model a plethora of phenomena ranging from phase separation and multiphase flows in materials engineering to bone development and tumor growth in biology. 

Diffusion systems are the mathematical models in which the change of state variables occurs via spreading of components. These systems are described by a set of parabolic PDEs and can model a large number of different systems in science and engineering. Combining the diffusion systems with moving-boundary problems provides a way to study the systems in which the diffusion leads to the change of domain geometry. Such systems have great importance in various real-world scenarios in chemistry and chemical engineering as well as environmental and life sciences.

In this post, I will describe the simplest possible formulation of such a system. The focus is on the Stefan problem, and the contribution of the diffusion part is simplified in a 1D formulation of a diffusion-controlled moving boundary test-case. In this example, we want to find the velocity of the moving interface in a solid-liquid diffusion system in which the solid particles diffuse in the liquid and the interface moves as a result of mass loss. Such a system resembles corrosion or degradation phenomena. For a 1D case, the position of the diffusion interface can be determined by:

$$
s(t)=s_{0}+2 \alpha \sqrt{t},
$$

where $$s(t)$$ and $$s_0$$ are the current and initial interface positions, respectively, and $$\alpha$$ is obtained through solving:

$$
\alpha=\frac{c_{0}-c_{\text {sat }}}{c_{\text {sol }}-c_{\text {sat }}} \sqrt{\frac{D}{\pi}} \frac{\exp \left(\frac{-\alpha^{2}}{D}\right)}{\operatorname{erfc}\left(\frac{-\alpha}{\sqrt{D}}\right)}
$$

where $$c_{\text {sol}}$$ is the concentration in the solid bulk (i.e. materials density), and $$c_{\text {sat}}$$ is the concentration at which the material is released to the medium. $$c_{0}$$ represents the initial concentration of the solid ions in the liquid, which is usually zero for most corrosion cases. Erfc is the [complementary error function](https://en.wikipedia.org/wiki/Error_function) and is defined as (it is usually available as a math function in most of the  numerical and symbolic computing libraries):

$$
\text { erfc } z=1-\operatorname{erf} z = 1 - \frac{2}{\sqrt{\pi}} \int_{0}^{z} e^{-t^{2}} d t
$$

To simplify the notation, let's encapsulate the concentration fraction:

$$
C=\frac{c_{0}-c_{\text {sat }}}{c_{\text {sol }}-c_{\text {sat }}} 
$$

So, the equation would be:

$$
\alpha=C \sqrt{\frac{D}{\pi}} \frac{\exp \left(\frac{-\alpha^{2}}{D}\right)}{\operatorname{erfc}\left(\frac{-\alpha}{\sqrt{D}}\right)}
$$

Obviously, this is not a simple equation to obtain $$\alpha$$ directly, but first, before going to use a numerical technique to solve it, we try to use a symbolic computation to see if it is capable of handling such an equation. For this purpose, we use [SymPy](https://www.sympy.org/en/index.html), a powerful Python package for symbolic mathematics. 

We should first import the package and initialize an interactive session:

```python
from sympy import init_session
init_session(quiet=True)
```

Then, we define the symbols:

```python
a, D, C = symbols("alpha, D, C")
```

It was pretty simple, wasn't it. So, let's go for defining the main equation and printing it to be sure that it's defined as expected:

```python
eq = a - C * sqrt(D/pi) * exp(-a**2/D) / erfc(-a/sqrt(D))
Eq(eq, 0)
```

which results to this output (which will be written in the LaTeX format thanks to what ``init_session`` has already prepared for us):

$$
\frac{C \sqrt{D} e^{- \frac{\alpha^{2}}{D}}}{\sqrt{\pi} \left(2 - \operatorname{erfc}{\left(\frac{\alpha}{\sqrt{D}} \right)}\right)} + \alpha = 0
$$

Perfect, let's call the solver to see what happens:

```python
solve(eq, a)
```

It produces the following error:

```
NotImplementedError       Traceback (most recent call last)
<ipython-input-3-644d539985dd> in <module>
----> 1 solve(eq, a)
```

Oops, apparently, the equation was so complex for our symbolic solver to understand, and it means nothing but we should switch to a numerical implementation. This is what I will explain in the next post.