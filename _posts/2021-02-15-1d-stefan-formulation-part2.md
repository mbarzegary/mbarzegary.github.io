---
layout: post
title: Solving Stefan (moving-boundary) formulation of a diffusion problem using numerical and symbolic computing - Part 2
---

In the [previous post]({% post_url 2020-12-28-1d-stefan-formulation-part1 %}), I quickly demonstrated the formulation of a moving-boundary problem for a 1D case, but in the end, we reached a problem related to the complexity of the derived equation. The complexity led to `NotImplementedError`, which means that the symbolic solver is not capable of solving that. As an alternative solution, in this post, we switch to a numerical solver to obtain the value of $$\alpha$$ through the following equation (see previous post for further explanation of this):

$$
\alpha=\frac{c_{0}-c_{\text {sat }}}{c_{\text {sol }}-c_{\text {sat }}} \sqrt{\frac{D}{\pi}} \frac{\exp \left(\frac{-\alpha^{2}}{D}\right)}{\operatorname{erfc}\left(\frac{-\alpha}{\sqrt{D}}\right)}
$$

To this end, we use [Newton method](https://en.wikipedia.org/wiki/Newton%27s_method), an iterative method to find the root of an equation. We reformulate the above equation and use the Newton method to find the value of the root, which will be $$\alpha$$ in this case.

Newton method works based on evaulation of the following iterative equation:

$$
x_{n+1}=x_{n}-\frac{f\left(x_{n}\right)}{f^{\prime}\left(x_{n}\right)}
$$

We start by guessing the initial value of $$x_{n}$$, and then successively find a new value of $$x_{n+1}$$ and replace it with $$x_{n}$$ again. The desired function in our case can be defined based on the above expression for $$\alpha$$:

$$
f(\alpha) = \alpha - C \sqrt{\frac{D}{\pi}} \frac{\exp \left(\frac{-\alpha^{2}}{D}\right)}{\operatorname{erfc}\left(\frac{-\alpha}{\sqrt{D}}\right)}
$$

with $$C$$ being:

$$
C=\frac{c_{0}-c_{\text {sat }}}{c_{\text {sol }}-c_{\text {sat }}} 
$$

Sympy can be used to easily calculate the derivative of $$f(\alpha)$$. Similar to the previous post, we define appropriate symbols in Sympy, but this time, we also assign some numerical values to them for facilitating further evaluations.

```python
from sympy import init_session
init_session(quiet=True)

D_value = 0.00075
c_0 = 0
c_sol = 1735
c_sat = 134
C_value = (c_0 - c_sat)/(c_sol - c_sat)

a, D, C = symbols("alpha, D, C", positive=True)
func = a - C * sqrt(D/pi) * exp(-a**2/D) / erfc(-a/sqrt(D))
Eq(func, 0)
```
which produces the following output: 

$$
\frac{C \sqrt{D} e^{- \frac{\alpha^{2}}{D}}}{\sqrt{\pi} \left(2 - \operatorname{erfc}{\left(\frac{\alpha}{\sqrt{D}} \right)}\right)} + \alpha = 0
$$

Now, we can simply calculate the derivitive of $$f(\alpha)$$:

```python
dfunc = diff(func, a)
dfunc
```

which results to:

$$
\frac{2 C e^{- \frac{2 \alpha^{2}}{D}}}{\pi \left(2 - \operatorname{erfc}{\left(\frac{\alpha}{\sqrt{D}} \right)}\right)^{2}} + \frac{2 C \alpha e^{- \frac{\alpha^{2}}{D}}}{\sqrt{\pi} \sqrt{D} \left(2 - \operatorname{erfc}{\left(\frac{\alpha}{\sqrt{D}} \right)}\right)} + 1
$$

Although this looks a bit complicated, we don't care about the complexity as Sympy will take care of the evaluation of this expression for obtaining $$f^{\prime}\left(\alpha\right)$$ in each iteration. Let's define some functions for the corresponding terms in the Newton method equation:

```python
func = func.subs([(D, D_value), (C, C_value)])
dfunc = dfunc.subs([(D, D_value), (C, C_value)])

def F(x):
    return func.subs(a, x).evalf()
 
def dF(x):
    return dfunc.subs(a, x).evalf()
```

And then, a pretyy simple declaration of the iterative Newton method:

```python
def dx(f, x):
    return abs(0-f(x))

def newtons_method(f, df, x0, e, verbose=False):
    delta = dx(f, x0)
    while delta > e:
        x0 = x0 - f(x0)/df(x0)
        delta = dx(f, x0)
        if verbose:
            print ('Delta is ', delta)

            
    print ('>>>>> Root is at ', x0)
    print ('>>>>> f(x) at root is ', f(x0))
    return x0
```

In this definition, the code stops iterating when the error drops below a certain threshold (passed by `e` to the function). 

Okay, everything is ready so far. We can start the iterations by calling `newtons_method(F, dF, x0, 1e-7, True)`, in which `x0` is the initial point to start with and the error threshold is $$10^{-7}$$. But, a better approach can be running the process for various start points and see if they all reach the same result or not:

```python
x0s = [0, .5, 1]
for x0 in x0s:
    newtons_method(F, dF, x0, 1e-7, True)
```

which generates this output, indicating that the value of $$\alpha$$ is $$-0.001366$$ with the selected values for the chemical parameters:

```
Delta is  8.68237762054906e-7
Delta is  3.82565589437956e-13
>>>>> Root is at  -0.00136691381199428
>>>>> f(x) at root is  3.82565589437956e-13
Delta is  0.00129321032032748
Delta is  8.68237762054906e-7
Delta is  3.82565589437956e-13
>>>>> Root is at  -0.00136691381199428
>>>>> f(x) at root is  3.82565589437956e-13
Delta is  0.00129321032032748
Delta is  8.68237762054906e-7
Delta is  3.82565589437956e-13
>>>>> Root is at  -0.00136691381199428
>>>>> f(x) at root is  3.82565589437956e-13
```

As you can see, the selected initial points all have converged to the same value, but of course the number of iterations is different to reach the root.

