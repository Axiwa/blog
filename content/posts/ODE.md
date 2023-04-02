---
title: Neural ODE
mathjax: true
categories: Project
---

## Dynamic system & Machine learning

When I was reading the paper [Neural Ordinary Differential Equations](https://arxiv.org/abs/1806.07366v4), I was confused of the adjoint method, so I turned to [A Proposal on Machine Learning via Dynamical Systems](https://doi.org/10.1007/s40304-017-0103-z). It is clearer about what we should do. We begin from the side of dynamical system:

Consider the differential equation in

$\frac{dx}{dt} = f(u(t),x), x(0)=x_0, u(t)\in U, t\in [0,T]$

We want to find the function $u(t)$, defined in $[0,T]$ that minimizes/maximizes 

$J(u) = \Psi(x(T))+\int_0^T L(x,u)dt$

If we fix a time horizon T, there is a mapping:

$x_0 \rightarrow x(T,x_0)$

If we consider supervised learning and for each data point $x_0^i$ there is a label $y^i$ associated with it, our question is whether the flow map of a given dynamical system can describe the model properly and to choose $u(t)$ such that a loss function of $x(T, x_0)$ and $y(x_0)$ could be minimized ($\mu(x)$ is the distribution). 

$\int(y(x)-g(x(T)))^2d\mu(x)+\int_0^T L(x,u)dt$ 
$= \sum_i(y_i-x_i(T))^2+\int_0^T L(x,u)dt$

We consider the [ResNet](https://www.geeksforgeeks.org/residual-networks-resnet-deep-learning/):

$x_{t+1} = x_{t}+f(u(t), x_t)$ 

When we use artificial time (adaptively choosing $\Delta t$ -> layers) and there are plenty of layers, then this is the discrete form of our differential equation.

$\frac{dx}{dt} = f(u(t),x), x(0)=x_0, u(t)\in U, t\in [0,T]$


Then we can regard $\Psi$ as the loss function and the integral (running cost) as the regularization term, and we could find the similarity between the deep neural network and dynamic systems.


## Adjoint method & optimization
Why is there adjustment to adj_y? <https://github.com/rtqichen/torchdiffeq/issues/59#issue-452634924>

ODEint problem: why do we need interpolation? <https://github.com/rtqichen/torchdiffeq/issues/114>

torch.autograd.grad???


Code 	Math 	Description
func_eval 	f(t1,y(t1),θ)f(t_1, y(t_1), \theta)f(t1​,y(t1​),θ) 	(arg) The output whose derivative you want.
y 	y(t1)y(t_1)y(t1​) 	(arg) The state w.r.t to which you want output’s derivative.
adj_y 	λ(t1)⊺\lambda(t_1)^\intercalλ(t1​)⊺ 	(arg) The free choice adjoint vector/lagrange multiplier.
vjp_y 	λ(t1)⊺∂f∂y(t1)\lambda(t_1)^\intercal \frac{\partial f}{\partial y(t_1)}λ(t1​)⊺∂y(t1​)∂f​ 	(ret) The vector-Jacobian product.


## Optimal control

What we need to do is to optimize $u(t)$ to minimize the $J$. This is a classic optimal control problem, which could be solved when solving the *Hamiltonian system*. This is the **Pontryagin's maximum principle**. 

If we consider a Lagrangian multiplier vector $\lambda$, and construct the *Hamiltonian* H:

$H(x(t),u(t),\lambda(t),t) = \lambda^T(t)f(x(t),u(t))+L(x(t),u(t))$

Pontryagin's minimum principle states that the optimal state trajectory $x^*$, optimal control $u^*$, and corresponding Lagrange multiplier vector $\lambda^*$ must minimize the Hamiltonian H so that:

(1) $H(x^*(t), u^*,\lambda^*,t) \leq H(x^*(t),u,\lambda^*,t)$

(if there is no constraint on $u(t)$, (1) becomes $H_u = 0$)

(2) $\Psi_T(x(T))+H(T) = 0$

(3) $-\dot{\lambda}^{T}(t) = H_x(x^*(t),u^*(t),\lambda(t),t)$

(4) $\lambda^{T}(T) = \Psi_x(x(T))$

The meaning of the notation can be found in <https://en.wikipedia.org/wiki/Pontryagin%27s_maximum_principle>, and a simplified proof could be found on <https://www.youtube.com/watch?v=i3Ca509ws-c> and <https://people.kth.se/~aaurell/Courses/SF3971_VTHT15/SMP-intro.pdf>. But the clearest proof in Chinese for me is <最优控制理论与系统>定理3-6

From the beginning I thought the adjoint sensitive method is the related to this but it seems not much...so I just give some links here from this viewpoint.

[Maximum Principle Based Algorithms for Deep Learning](https://arxiv.org/abs/1710.09513)

[Layer-Parallel Training of Deep Residual Neural Networks](https://arxiv.org/abs/1812.04352)

