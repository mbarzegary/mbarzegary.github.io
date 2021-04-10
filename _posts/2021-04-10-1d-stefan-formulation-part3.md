---
layout: post
title: Solving Stefan (moving-boundary) formulation of a diffusion problem - Part 3 - FreeFEM (C++) implementation 
---

In the [previous post]({% post_url 2021-02-15-1d-stefan-formulation-part2 %}), we finished working on the implementation of a Newton solver to obtain the value of $$\alpha$$ in the following equation:

$$
\alpha=\frac{c_{0}-c_{\text {sat }}}{c_{\text {sol }}-c_{\text {sat }}} \sqrt{\frac{D}{\pi}} \frac{\exp \left(\frac{-\alpha^{2}}{D}\right)}{\operatorname{erfc}\left(\frac{-\alpha}{\sqrt{D}}\right)}
$$

Now, we want to follow the same approach, but this time in [FreeFEM](https://freefem.org/), which enables us to embed the implementation inside relevant applications. As described before, in order to solve the above equation using Newton's method, we need to know the derivative of the function as well. We used SymPy to calculate it, but then, the question is how to do that in a C-like language such as FreeFEM. The answer lies within one of the less-known features of SymPy: ability to print C functions. So, let's ask SymPy to write the C code for us. 

For simplicity, I put the necessary part of the code again here:

```python
from sympy import init_session
init_session(quiet=True)

a, D, C = symbols("alpha, D, C", positive=True)
func = a - C * sqrt(D/pi) * exp(-a**2/D) / erfc(-a/sqrt(D))
dfunc = diff(func, a)
```

So, we have the function and its derivitive in the `func` and `dfunc` variables. Time to magic, let's take advantage of SymPy printing APIs:

```python
from sympy.printing import print_ccode
print_ccode(func)
print_ccode(dfunc)
```

It prints the following output in C syntax, which can be directly copied into a C program.

```
-C*sqrt(D)*exp(-pow(alpha, 2)/D)/(sqrt(M_PI)*(2 - erfc(alpha/sqrt(D)))) + alpha

2*C*exp(-2*pow(alpha, 2)/D)/(M_PI*pow(2 - erfc(alpha/sqrt(D)), 2)) + 2*C*alpha*exp(-pow(alpha, 2)/D)/(sqrt(M_PI)*sqrt(D)*(2 - erfc(alpha/sqrt(D)))) + 1
```

Now, similar to [the developed Python code]({% post_url 2021-02-15-1d-stefan-formulation-part2 %}), we implement a simple FreeFEM code for the Newton's method. 

```cpp
func real F(real x, real C, real D)
{
  return -C*sqrt(D)*exp(-pow(x, 2)/D)/(sqrt(pi)*(2 - erfc(x/sqrt(D)))) + x;
}

func real dF(real x, real C, real D)
{
  return 2*C*exp(-2*pow(x, 2)/D)/(pi*pow(2 - erfc(x/sqrt(D)), 2)) + 2*C*x*exp(-pow(x, 2)/D)/(sqrt(pi)*sqrt(D)*(2 - erfc(x/sqrt(D)))) + 1;
}

func real dX(real x, real C, real D)
{
  return abs(0 - F(x, C, D));
}

func real newtons(real x0, real e, real C, real D)
{
  real delta = dX(x0, C, D);
  real x = x0;

  while (delta > e)
  {
    x = x - F(x, C, D) / dF(x, C, D);
    delta = dX(x, C, D);
    cout << "Delta is " << delta << endl;
  }

  cout << ">>>> Root is at " << x << endl;
  cout << ">>>> f(x) at root is " << F(x, C, D);
  return x;
}
```

And then, we can check it with some numerical values for the constants and 3 different initial points:

```cpp
real c0 = 0;
real cSol = 1735;
real cSat = 134;
real C = (c0 - cSat)/(cSol - cSat);
real D = 0.0001366;

real x0 = 0;
cout << newtons(x0, 1e-7, C, D) << endl;

x0 = 0.5;
cout << newtons(x0, 1e-7, C, D) << endl;

x0 = 1;
cout << newtons(x0, 1e-7, C, D) << endl;
```

which produces this output (using FreeFEM v4.7):

```
Delta is 3.70539e-07
Delta is 1.63268e-13
>>>> Root is at -0.000583359
>>>> f(x) at root is 1.63268e-13-0.000583359
Delta is 0.000551904
Delta is 3.70539e-07
Delta is 1.63268e-13
>>>> Root is at -0.000583359
>>>> f(x) at root is 1.63268e-13-0.000583359
Delta is 0.000551904
Delta is 3.70539e-07
Delta is 1.63268e-13
>>>> Root is at -0.000583359
>>>> f(x) at root is 1.63268e-13-0.000583359
```