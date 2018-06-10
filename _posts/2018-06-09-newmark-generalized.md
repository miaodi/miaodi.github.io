---
title:  "Camparison of Newmark-beta method and Generalized-alpha method"
search: false
categories: 
  - Finite element method
last_modified_at: 2018-06-09T08:05:34-05:00
use_math: true
comments: true
---

## Intro

Recently, I was working on solving some phase-field based fracture problems, where I need to do time marchings to let the fracture propagate in time domain. Taking this opportunity, I reviewed a bunch of numerical methods for ODEs. Different methods have different accuracies and are focused on different type of problems. Although Runge-Kutta method and Backward Euler method are well known to numerical people in their accuracy and stability,  in finite element field, people like to use Newmark-$\beta$ method and Generalized-$\alpha$ method. The reason is because these two methods allows to solve second order ODEs directly without introducing any state variables. Hence, I reviewed the performances of of these two methods in detail. In case I forget their properties later on, I'd like to document my study here.

## Problem

In general, a dynamic finite element problem can be written as

$$\mathbf{M}\ddot{\mathbf{d}}+\mathbf{C}\dot{\mathbf{d}}+\mathbf{K}{\mathbf{d}}=\mathbf{f}$$

where $\mathbf{M}$, $\mathbf{C}$, $\mathbf{K}$, are mass, damping and stiffness matrix, respectively. $\mathbf{d}$ is the displacement vector and $\mathbf{f}$ is the external load vector.

Now, the problem is: Given the displacement, speed and acceleration at certain time $(t_n)$, how to find their values at the next time step $(t_{n+1}:=t_n+\delta{t})$. The two methods we are about to discuss are designed to tackle this issue.

## Newmark-$\beta$ method

The Newmark-$\beta$ method was proposed in 1959 and is widely used in solve $2^{nd}$ order dynamic problems nowadays. The scheme is given as:

$$
\begin{align*}
  \mathbf{M}{\mathbf{a}}_{n+1}+\mathbf{C}{\mathbf{v}}_{n+1}+\mathbf{K}{\mathbf{d}_{n+1}}&=\mathbf{f}_{n+1}\\
  \mathbf{d}_{n+1}&=\mathbf{d}_{n}+\delta{t}\mathbf{v}_{n}+\frac{\delta{t}^2}{2}\left({\left(1-2\beta\right)\mathbf{a}_{n}+2\beta{\mathbf{a}_{n+1}}}\right)\\
  \mathbf{v}_{n+1}&=\mathbf{v}_{n}+\delta{t}\left({\left(1-\gamma\right)\mathbf{a}_{n}+\gamma{\mathbf{a}_{n+1}}}\right)
\end{align*}
$$

The primary requirement of numerical algorithms is that they should provide adequate good approximations of exact solution. The definition of 'adequate good' contains two aspects: stability and accuracy, and the accuracy of a numerical method heavily depends on its stability. From these two properties, we can determine the values of $\gamma$ and $\beta$.

### Stability

Stability means, for a given disturbance in the solution for a certain moment, its influence should eventually fades out. To demonstrate this conception, we consider a dampingless singule-degree-of-freedom (SDOF) problem (Indeed, any dampingless system can be decomposed into SDOFs by spectral decomposition.):

$$
\begin{align*}
  a_{n+1}+\omega^2d_{n+1}&=0\\
  {d}_{n+1}&={d}_{n}+\delta{t}{v}_{n}+\frac{\delta{t}^2}{2}\left({\left(1-2\beta\right)a_{n}+2\beta{a_{n+1}}}\right)\\
  {v}_{n+1}&={v}_{n}+\delta{t}\left({\left(1-\gamma\right){a}_{n}+\gamma{a_{n+1}}}\right)
\end{align*}
$$

Once we have the above expression, we can solve $$\mathbf{X}_{n+1}:=\left[\delta{t}^2{a_{n+1}}, \delta{t}{v_{n+1}, {d_{n+1}}}\right]^T$$ in terms of $\mathbf{X}_{n}$ as


$$
\begin{align*}
  \mathbf{X}_{n+1}=\mathbf{A}\mathbf{X}_{n},
\end{align*}
$$

with

$$
\begin{align*}
  \mathbf{A}=
  \begin{bmatrix}
 \frac{(2 \beta -1) \Omega ^2}{2 \beta  \Omega ^2+2} & -\frac{\Omega ^2}{\beta  \Omega ^2+1} & -\frac{\Omega ^2}{\beta  \Omega ^2+1} \\
 \frac{2 \beta  \Omega ^2-\gamma  \left(\Omega ^2+2\right)+2}{2 \beta  \Omega ^2+2} & \frac{\beta  \Omega ^2-\gamma  \Omega ^2+1}{\beta  \Omega ^2+1} & -\frac{\gamma  \Omega ^2}{\beta  \Omega ^2+1} \\
 \frac{1-2 \beta }{2 \beta  \Omega ^2+2} & \frac{1}{\beta  \Omega ^2+1} & \frac{1}{\beta  \Omega ^2+1} 
  \end{bmatrix}
\end{align*}
$$

where $\Omega=\omega\delta{t}$. The matrix $\mathbf{A}$ is called amplification matrix. The above formulation allows us to do time stepping by matrix multiplications as $$\mathbf{X}_{n+N}=\mathbf{A}^N\mathbf{X}_{n}$$. The stability requires that all eigenvalues of the amplification matrix should have magnitude that less than 1 so that any disturbance (in terms of $d$, $v$ or $a$) will be completely damped out. By solving the characteristic equation of $\mathbf{A}$, we can obtain three eigenvalues. By applying the magnitude constraint on these three eigenvalues, we can deduce the range of $\gamma$, $\beta$ and $\Omega$ (Indeed, what we really care about is the time step size $\delta{t}$.) that we are allowed to choose. You can use any symbolic math tool to solve this problem, the results are

* $2\beta\geq\gamma\geq\frac{1}{2}$ for all $\Omega\geq{}0$. In other words, it is stable for any choice of the time step size $\delta{t}$. This is also called unconditionally stable.
* $\gamma\geq\frac{1}{2}$ && $\beta\leq\frac{\gamma}{2}$ && $\Omega\leq{\frac{2}{\gamma-2\beta}}$. This is also called conditionally stable, because there is additional constraint on the time step size.

Because of the constraint on $\Omega$, sometimes we need to find the highest frequency $\omega$ in the finite element linear system to identify the largest feasible time step.

### Accuracy

A numerical method is accurate if $\|\|u-u^h\|\|\rightarrow{}0$ as $\delta{t}\rightarrow {}0$. However, accurate methods can differ in their efficiency, that is the approximted result can converge to analytical solution in different speeds. In this section, we will discuss the convergence rate of Newmark-$\beta$ in detail.

Since there are three fields ($d$, $v$ and $a$) involved in the original formulation, for the sake of simplification, we should try to eliminate some of them. Here we consider a multi-step system

$$
\begin{align*}
  \begin{cases}
    \mathbf{X}_{n+1}&=\mathbf{A}\mathbf{X}_{n}\\
    \mathbf{X}_{n+2}&=\mathbf{A}\mathbf{X}_{n+1}\\
    \mathbf{X}_{n+3}&=\mathbf{A}\mathbf{X}_{n+2}.
  \end{cases}
\end{align*}
$$

From the above linear system, we can eliminate all speed and accelerations terms, resulting in a pure displacement equation as

$$
\begin{equation*}
  A_0d_{n}+A_1d_{n+1}+A_2d_{n+2}+A_2d_{n+3}=0.
\end{equation*}
$$

The expression of coefficients $A_i$ are quit clumsy, and the derivation procedure is tedious, but again you can use symbolic math tools to do it very quickly. We end up wth the following equation

$$
\begin{equation*}
  A_0d_{n}+A_1d_{n+1}+A_2d_{n+2}+A_2d_{n+3}=0.
\end{equation*}
$$

By replacing numerical solution $d_{n+i}$ with exact solution $d(t_{n+i})$ and doing Taylor expansion on each displacement terms *w.r.t* $d(t_{n})$, we obtain the truncation error

$$
\begin{equation*}
  \tau= -\frac{\Omega \left(4 \dddot{d}(t_n)+(2 \gamma +3) \omega^2 \dot{d}(t_n)\right)}{2\omega}.
\end{equation*}
$$

Hence, in general, Newmark-$\beta$ is a first order accurate method, but if we can let

$$
\begin{equation*}
  4=2\gamma+3 \rightarrow \gamma=\frac{1}{2},
\end{equation*}
$$

the first order truncation error can be completely eliminated. As a result, we improve Newmark-$\beta$ to second order accuracy.
