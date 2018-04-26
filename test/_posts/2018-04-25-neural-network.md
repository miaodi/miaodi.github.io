---
title:  "Neural Network and Back Propagation Algorithm"
search: false
categories: 
  - Jekyll
last_modified_at: 2018-02-19T08:05:34-05:00
use_math: true
---

## Intro
Neural networks are typically organized in layers. Layers are made up of a number of interconnected nodes which contain an activation function $h_\theta(z)$. Patterns $\mathbf{x}$ are presented to the network via the input layer, which communicates to one or more hidden layers where the actual processing is done via a system of weighted connections $\mathbf{\Theta}$.  The hidden layers then link to an output layer where the predict answer $\mathbf{y}^{pred}$ is output as shown in the graphic below.

<figure style="width: 450px" class="align-center">
  <img src="{{ site.url }}{{ site.baseurl }}/assets/images/tikz41.png" alt="">
  <figcaption>An artificial neural network with 3 hidden layers.</figcaption>
</figure> 

## Math Model

Here, we use the sigmoid function as activation function:

$$h(z)=\frac{1}{1+e^{-z}},$$

which has a convenient derivative of:

$$h'=h(z)(1-h(z)).$$

We assume there are L layers of neural networks. We label nodes of layer $i$ as $\mathbf{a}^{(i)}$ and call them "activation units". The output of the $i^{th}$ layer is the input of the $i+1^{th}$ layer. In the input layer,

 $$\mathbf{a}^{(0)}:=\mathbf{x};$$

 and in the output layer,

 $$\mathbf{a}^{(L)}:=\mathbf{y}^{pred}.$$

 $\mathbf{a}^{(i)}$ can be solved in a forward propagation manner, as:

  $$\begin{align*}
  z&=\theta^{(i)}\cdot\mathbf{a}^{(i-1)}\\
  \mathbf{a}^{i}&=h(z)
  \end{align*}.$$

Given the trainning set $\\{ (\mathbf{x}_1, \mathbf{y}_1), (\mathbf{x}_2, \mathbf{y}_2), \cdots, (\mathbf{x}_m, \mathbf{y}_m)\\}$, the cost function for this neural network can be written as:

$$
J=-\frac{1}{m}\sum_{t=1}^{m}(\mathbf{y}_t^{T}\cdot\log(\mathbf{y}_t^{pred})+(1-\mathbf{y}_t)^{T}\cdot\log(1-\mathbf{y}_t^{pred}))
$$

