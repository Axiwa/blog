---
title: Hair Rendering
categories: CG
date: 2023-04-02
katex: true
markup: pandoc
---
画头发好好玩！在 Mitsuba3 中实现毛发的渲染

<!-- ![](../../../images/hair/1_m.png) -->
<img src="../../images/hair/fourhair.png" alt="four color" width="500" height="200">

<!--more-->

## Introduction

Realistic rendering for hair is an interesting topic. Classic [Kajiya-Kay](https://dl.acm.org/doi/10.1145/74334.74361) model is still used in modern games, but it could not simulate the translucent features. [Marschner](https://dl.acm.org/doi/10.1145/882262.882345) model is another widely used hair shading model, which approximated the hair as a transparent circular cylinder, considering tilted cuticle scales. They proposed three modes of reflection and transmission: R for first reflection across the cylinder interface, TT for transmission through the hair, and TRT for internal reflection: 

<!-- ![](../../../images/hair/1_m.png) -->
<img src="../../images/hair/TRT.png" alt="four color" style="width: 60%; height: auto;">

The distribution of scattering of hair is composed on these three lobes, and each lobe can be divided into longitudinal scattering (M) and azimuthal scattering (N). However, the scattering inside the hair can be more than three times. [d'Eon et al.](https://dl.acm.org/doi/10.1111/j.1467-8659.2011.01976.x) proposed an energy conserving model. Based on Marschner model, they consider more order of interaction on the surface in addition to R/TT/TRT. [Yan's double cylinder model](https://dl.acm.org/doi/10.1145/2816795.2818080) considers the micro structure of the hair, where there is an outer cylinder cortex and an inner cylinder medulla, often absent from human hair fibers but plays more important role in animal fur. [d'Eon et al.](https://dl.acm.org/doi/10.1145/2614106.2614161) showed a non-separable fiber-scattering lobes which can explain behaviors seen in forward scattering. Huang et al. proposed the [microfacet](https://onlinelibrary.wiley.com/doi/full/10.1111/cgf.14588) model for hair shading and its importance sampling, which can better simulate the phneomenon under forward scattering.

Kajiya and Kay, Marschner et al. and d'Eon et al. are far-field model, where we assume the microscopic fiber's diameter is so small that the shading we see macroscopically at a point on the surface of this fiber is actually the sum of the light from a circumference of points on the cylinder. Therefore, when the hair fiber covers more than several pixels, the fiber still looks flat. They usually require [integration technique](https://dl.acm.org/doi/10.1111/j.1467-8659.2011.01976.x) along the azimuthal section. [Zinke et al.](https://dl.acm.org/doi/10.1109/TVCG.2007.43) first derived a near field shading model for dielectric fibers. The microfacet hair model is also based on near-field model.

<img src="../../images/hair/near_vs_far.png" alt="four color" style="width: 80%; height: auto;">

The model we use for implementation is [Chiang et al.](https://dl.acm.org/doi/10.1145/2775280.2792559), a near-field model. Based on d'Eon et al., they take the position information in azimuthal direction into consideration. The model also considers the scattering behavior of an infinite number of high-order lobes and separates longitudinal and azimuthal scattering. 

This work can be summarized as follows.

- We look into the mathematical details of Chiang et al. model and analyze the implementation methods in [PBRT3](https://pbr-book.org/3ed-2018/contents) and Mitsuba3.

- We conduct numerical tests to verify the correctness of the scattering function and importance sampling.

- By comparing different scenes rendered by our implementation and other render / hair model, we analyze the difference among them and showcase our hair scattering model works properly with the existing curve model.

## Scattering model

### Notation
Following the convention of [Marschner et al.](https://dl.acm.org/doi/10.1145/882262.882345), we use a spherical coordinate system where axis stays aligned with the fiber's tangent vector. The axis is like this:

<img src="../../images/hair/coord.png" alt="coord" style="width: 60%; height: auto;">

We define $w-v$ plane as the normal plane. $\theta$ measures the angle between the ray and normal plane, which ranges from $-\pi/2$ to $\pi/2$ and $\pi/2$ corresponds to a direction $u$ increases. The azimuthal angle $\phi$ is the angle between the projected ray on the normal plane and $v$ axis, which ranges from $0$ to $2\pi$. We use $p$ to denote the number of path segments the ray travels inside the hair before back to outside, which also describe the scattering events at the boundary: $p=0$ corresponds to R, $p=1$ corresponds to TT, $p=2$ corresponds to TRT, and so forth. We also consider the tilt at the mesosurface, denoting the tile scale with $\alpha$.

### Factored lobe

We denote hair BSDF as a function of incident ray and outgoing ray $f(\omega_i, \omega_o)$. Since we assume the fiber to be circular with radial symmetry, so the azimuthal scattering function is only parameterized by the relative azimuth $\phi_d = \phi_o - \phi_i$. The fiber reflectance is fully parameterized by three angles: $f(\theta_i, \theta_o, \phi_d)$, which can be decomposed into a sum of separate lobes ([Marschner et al.](https://dl.acm.org/doi/10.1145/882262.882345)) corresponding to the $p$-th internal interaction with the fiber.

$$
    f\left(\theta_i, \theta_o, \phi\right)=\sum_{p=0}^{\infty} f_p\left(\theta_i, \theta_o, \phi\right)
$$

$f_p(\theta_i, \theta_o, \phi_d)$ can be further decomposed into longitudinal scattering function $M_p(\theta_i, \theta_o)$, attenuation function $A_p(h)$ and azimuthal scattering function $N_p(\phi_d)$. $h$ is the offset along the curve width where the ray intersects with the "flat ribbon" of the curve. $h=\pm 1$ means the grazing angle at the edge of the circle, and $h=0$ means the ray hits right in the center of a curve's width.

### Longitudinal scattering

In Marschner et al. model, taking (1) the roughness $\beta_m$ along the longitude, and (2) the cuticle scale into account, they use a normalized Gaussian function with standard deviation: $M_p(\theta_h) = g(\beta_h; \theta_h-\alpha_p)$ as distribution of $M_p$. In d'Eon(2011), they chose another energy-conserving distribution function:

$$
    M_p\left(v, \theta_i, \theta_o\right)=\frac{\operatorname{csch}(1 / v)}{2 v} e^{\frac{\sin \left(-\theta_i\right) \sin \theta_o}{v}} I_0\left[\frac{\cos \left(-\theta_i\right) \cos \theta_o}{v}\right]
$$

where $v$ is the roughness variance and $I_0 (x)$ is the modified Bessel function of the first kind. In d'Eon(2013) they added numerical stablizing way for low roughness condition, and we use it in our implementation. More concretely, for different $p$, we have different variance $v[p]$ calculated from $\beta_m$ and lower order $v$, and with known $\theta_i$ and $\theta_o$, we can calculate $M_p$.

### Azimuthal Scattering

For far-field models, they need to calculate the integral over the fiber width and take the attenuation function along each path into account. For example, in d'Eon(2011), they have:

$$
    N_p(\phi)=\frac{1}{2} \int_{-1}^1 A(p, h) D(\phi_d-\Phi(p, h)) d h
$$

where $\Phi$ is the exact azimuth predicted by perfectly smooth fiber cylinder surface

$$
    \phi(p, h)=2 p \gamma_t-2 \gamma_i+p \pi
$$


D is the Gaussian detector for surface roughness:

$$
    D(\phi)=\sum_{k=-\infty}^{\infty} g(\beta ; \phi_d-2 \pi k)
$$

However, calculating this integral is usually slow and may not capture the peaks. Chiang et al. proposed to sample $N_p$ using Monte Carlo method and replace the wrapped Gaussian with the logistic distribution

$$
\begin{aligned}
    & l(x, s)=\frac{\mathrm{e}^{-x / s}}{s\left(1+\mathrm{e}^{-x / s}\right)^2} \\
    & \int l(x, s) \mathrm{d} x=\frac{1}{1+\mathrm{e}^{-x / s}} \\
    & N_p(\phi, h)=A_p(h) \frac{l(\phi_d-\Phi(p, h), s)}{\int_{-\pi}^{\pi} l\left(x^{\prime}, s\right) \mathrm{d} x^{\prime}}
\end{aligned}
$$

With analytically integratable logistic distribution, $N_p$ can easily describe azimuthal angular distribution and can be perfectly sampled.

### Absorption

The $A_p$ term describes how much energy the incident light loses after travelling through the hair. It is affected by (1) Fresnel reflection and (2) Transimission in the hair. For refraction we first need to calculate the modified index because of the longitudinal angle in the normal plane

$$
    \eta^{\prime}=\frac{\sqrt{\eta^2-\sin ^2 \theta_{\mathrm{i}}}}{\cos \theta_{\mathrm{i}}}
$$

Then we can get the refracted angle $\gamma_t$ "inside the hair" using [Snell's law](Langenbucher2018). With the modified refractive index $\eta^{\prime}$, we have $\sin{\gamma_i} = h$, and $\eta^{\prime} \sin{\gamma_t} = h$. Then from geometric knowledge:

<table><tr>
<td><img src=../../images/hair/ap1.png border=0 style="width: 80%; height: auto;"></td>
<td><img src=../../images/hair/ap2.png border=0 style="width: 80%; height: auto;"></td>
</tr></table>


we can see the ray's length projected to the normal plane is 

$$
    l_a = 2\cos{ \gamma_t }
$$

and considering the scaled factor caused by longitudinal projection $\cos{\theta_t}$, we get the length the light stays inside the hair

$$
    l=\frac{2 \cos \gamma_{\mathrm{t}}}{\cos \theta_{\mathrm{t}}}
$$

Given the absorption coefficient $\sigma_a$ and apply Beer's law, the transmittance is given by

$$
    T=\mathrm{e}^{-\sigma_{\mathrm{a}} l}
$$

We do not consider the cylinder diameter here. 

Denote the first air-hair boundary Fresnel reflectance as f. With the formula $A_0 = f$ and $A_p=(1-f)^2 T^p f^{p-1}$ for $p>0$, we can calculate $A_p$ term.

In Chiang et al. model, for $p >= 3$, they use a fourth term to describe the sum of $A_p$ for all $p$ to avoid heavy computation cost ([d'Eon et al. 2011](https://dl.acm.org/doi/10.1111/j.1467-8659.2011.01976.x)) and keep the energy loss small. We use this method for implementation.

$$
    A_{\text {pmax }}=\sum_{p=pmax}^{\infty} A_p=\frac{(1-f)^2 f^{pmax-1} T^pmax}{1-f T}
$$


### Importance sampling

For importance sampling we need two random numbers to sample $M_p$, an extra random number to sample a $N_p$ and an extra random number to sample $p$, where the probability is proportional to the attenuation term $A_p$. Because Mitsuba3 only passes 3 random numbers, we need to use discrete PDF to "reuse" the random number for sampling afterwards. PBRT3 cast a random number $\xi$ to a 64-bit Int, then take the value of the odd bit and the number on the even bit to form two 32-bit numbers respectively, but from our Chi-Squared test the numbers generated still have some pattern.

We first use a random number to sample $p$, and generate the reused random number. We use this number and the closed-form inverse CDF of logistic function to generate the relative azimuth $\phi_d$. For $M_p$, the perfect sampling function is from d'Eon et al. 2013

$$
    \begin{aligned}
        & u\left(\xi_1\right)=v \log \left(\mathrm{e}^{1 / v}-2 \xi_1 \sinh \frac{1}{v}\right) \\
        & \theta_o\left(\xi_1, \xi_2, v, \theta_{\text {cone }}\right)= \\
        & \quad \arcsin \left(u\left(\xi_1\right) \cos \theta^{\prime}+\sqrt{1-u\left(\xi_1\right)^2} \cos \left(2 \pi \xi_2\right) \sin \theta^{\prime}\right)
    \end{aligned}
$$

where $\theta_{\text {cone }}$ is the theoretical specular outgoing $\theta_o$ considering tilt, and $\theta^{\prime}=\frac{\pi}{2}-\theta_{\text {cone }}$.

However, this method is numerically unstable. PBRT3 and [Tungsten](https://benedikt-bitterli.me/tungsten.html) both use the following formula and we also take this version:

$$
\begin{aligned}
    & u\left(\xi_1\right)= 1 + v \log \left(\xi_1 + (1-\xi_1) e^{-2/v}\right) \\ 
\end{aligned}
$$

The corresponding PDF is 

$$
    \sum_{p=0}^{p_{\max }} M_p\left(\theta_{\mathrm{o}}, \theta_{\mathrm{i}}\right) \tilde{A}_p\left(\omega_{\mathrm{o}}\right) N_p(\phi)
$$

where $\tilde{A}_p$ is the normalized luminance-weighted PDF term.


## Implementation details

With all the formulas above, we can implement the BSDF, importance sampling and PDF function in Mitsuba3. The implementation borrows a lot from PBRT3's code. However, to integrate the model perfectly with the system, we need to note:

- Mitsuba3 supports differential rendering, but we do not consider it for hair and add `NonDifferentiable` flag to parameters.

- Mitsuba3 supports vectorized operation on Intel’s Embree and NVidia’s OptiX. For hair, we heavily depend on parallelism and need to use [Dr.Jit](https://rgl.epfl.ch/publications/Jakob2022DrJit) carefully for loop and branch to prevent overhead.

- In the framework above, we follow the local coordinate of PBRT, with local x axis the same as +u direction and y axis to be +v direction. This requires our curve to have the same local coordinate, otherwise it cannot render correctly

<table><tr>
<td><img src=../../images/hair/ourcoord.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/truecoord.png border=0 style="width: 100%; height: auto;"></td>
</tr></table>


- Most hair models require a parameter $h$ to specify the azimuthal offset. In PBRT's implementation $h$ can be calculated using $v$ coordinate of the curve shape, which means their $v$ coordinate is related to the direction of the incident ray. To have a consistent UV coordinate we don't change v but use normal at the intersection point and the incident ray projected to the normal plane to directly calculate $\gamma = acos(\vec{n}, \text{projected}(\vec{\omega_i)})$


## Numerical tests

To verify the correctness of the implementation, we use numerical tests in Mitsuba3 to test our model.

### White furnace test
The white furnace test checks whether energy is conserved if the hair does not absorb anything ($\sigma_a = 0$). For one arbitrary incident ray with unit radiance, the integral of reflected radiance over the outgoing direction should be one. 

$$
    \int_{\mathcal{S}^2} f\left(\omega_{\mathrm{i}}, \omega_{\mathrm{o}}\right)\left|\cos \theta_{\mathrm{o}}\right| \mathrm{d} \omega_{\mathrm{o}} = 1
$$

This integral is calculated using Monte Carlo method. We can either use uniform sampling or importance sampling. In our tests, uniform sampling can pass a 5\% relative tolerance using 300000 samples, and importance sampling can pass a 1\% relative tolerance using 10000 samples.

### Sampling consistency
To test if the importance sampling matches the PDF exactly, we can test if uniform sampling and importance sampling match under the situation where there are sufficient number of incident rays who have the same radiance if they are from the same direction. For every incident ray we sample, we can add some radiance to the shading point. In the end, the "color" of this point for uniform sampling and importance sampling should be close, as the rendering equation shows.

$$
    L_{\mathrm{o}}\left(\mathrm{p}, \omega_{\mathrm{o}}\right)=\int_{\mathrm{S}^2} f\left(\mathrm{p}, \omega_{\mathrm{o}}, \omega_{\mathrm{i}}\right) L_{\mathrm{i}}\left(\mathrm{p}, \omega_{\mathrm{i}}\right)\left|\cos \theta_{\mathrm{i}}\right| \mathrm{d} \omega_{\mathrm{i}}
$$

Our model can pass at a 5\% relative tolerance with 131072 samples, and 1\% relative tolerance with 819200 samples.

### Chi-squared test
Another method to test if importance sampling matches PDF is Chi-squared test: For one incident direction, we record how many outgoing samples fall within specific $\phi$ and $\theta$ bin. We also use numerical ways to integrate our PDF for each bin. Finally we can get two array of numbers, and we use Chi-squared test to see if these two set of data follow the same distribution.

By testing every $\phi$ and $\theta$ around the sphere in a $36^\circ$ granularity, we found our model can pass most of the tests, but cannot pass all. By changing the Chi-squared test parameters, such as resolution and quadrature, these failed tests can pass but others fail again. Except for increasing the quadrature (which did not work), we tried two ways to find out the reason why Chi-squared tests failed: (1) We fix scattering lobes by specify $p = 0/1/2/..$ and avoid the random number reuse for choosing $p$; (2) Since we have separable model, we can make the sampling of $M_p$ or $N_p$ to be uniform, and test if one of them has problem. After extensive tests, our conclusion is that only when we uniform sample $N_p$ can Chi-squared test pass, under different parameters. 

Chiang et al states that the azimuthal scattering is highly varying along the width. It is one of our reasons of failure because nearly all of our failing tests stay at low azimuthal roughness, and we used the same Chi-sqaured test for PBRT3 which also failed to pass all. We should look further into it to explain the problem with logistic distribution and its sampling.

## Results

### Synthetic hair

We use a simple enough scene with a point emitter and 50 straight hair to show the behavior of our renderer. For comparison, we use PBRT3's render with same hair model and Mitsuba3 with curve and cylinder shape with same width. For back scattering, with thin enough hair segment, cylinder, our hair BSDF and PBRT3's model do not have visible difference (Due to the resolusion of the figure, there might be Moire pattern).

Though we have same hair BSDF model as PBRT3, since we have different ray-curve intersection implementation, when fiber's size is not thin enough, we will have different behavior. PBRT3's curve does not have a 3D structure but their normal and UV coordinate is calculated based on the intersection condition. However, our curve has 3D structure and can get sufficient information of the intersection.

<img src="../../images/hair/backscattering.png" alt="ap1" style="width: 85%; height: auto;">

Backward scattering for (1) Hair (2) Cylinder (3) PBRT3's hair. Fiber with radius=0.0005. $\beta_m=0.3$, $\beta_n=0.3$


<img src="../../images/hair/backscattering2.png" alt="ap2" style="width: 85%; height: auto;">

Backward scattering for (1) Hair (2) Cylinder (3) PBRT3's hair. Fiber with radius=0.0005. $\beta_m=0.3$, $\beta_n=0.01$

<img src="../../images/hair/forwardscattering.png" alt="ap2" style="width: 85%; height: auto;">

Forward scattering for (1) Hair (2) Cylinder (3) PBRT3's hair. Fiber with radius=0.0005. $\beta_m=0.3$, $\beta_n=0.3$

<img src="../../images/hair/backscattering3.png" alt="ap1" style="width: 50%; height: auto;">

Backward scattering for (1) Hair (2) PBRT3's hair. Fiber with radius=0.005. $\beta_m=0.3$, $\beta_n=0.3$ We can see slight difference, where our hair is brighter.


<table><tr>
<td><img src=../../images/hair/curry.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/curry_pbrt.png border=0 style="width: 100%; height: auto;"></td>
</tr></table>

(1) Mitsuba3 Hair and (2) PBRT's Hair. With a small white area light below the curve

In both we could see the first reflection (the white specular at the bottom), first transmission (the bright blue spot on the central axis), and the TRT lobe from the dark blue spots on both sides of the curve. (There are some defects in our curve, and it might be related to the curve implementation then.)

### Real scene

Using [Bitterli](https://benedikt-bitterli.me/resources/)'s hair models, we could render more complex and realistic scenes. Due to the current problem of CUDA synchronization, we could not render more than about 40000 hair fibers. We added the eumelanin and pheomelanin parameter for better control the color of hair. It is wavelength related and $\sigma_a$ will be calculated every time when scattering happens, based on a model by [Donner and Jensen](https://dl.acm.org/doi/10.5555/2383894.2383946).

<table><tr>
<td><img src=../../images/hair/thick_m.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/thick_p.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/1_m.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/1_p.png border=0 style="width: 100%; height: auto;"></td>
</tr></table>

Thick and thin colored hair scene under directional light. (1) Hair. radius=0.01 (2) PBRT's Hair. radius=0.01 (3) Thin Hair. radius=0.001 (4) Thin PBRT's Hair. radius=0.001. There is visible difference between the two thick version, possibly because of the width of the hair

If our hair is thin enough, there is no visible difference between our and PBRT3's render. However, with larger width, our render will be brighter at some part of the hair, which is compatible with the 50 hair render. 

<table><tr>
<td><img src=../../images/hair/3_m_front.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/3_p_front.png border=0 style="width: 100%; height: auto;"></td>
</tr></table>

Curly hair under point light (1) Hair with front scattering (2) PBRT's Hair with front scattering

<table><tr>
<td><img src=../../images/hair/2.png border=0 style="width: 100%; height: auto;"></td>
<td><img src=../../images/hair/0.png border=0 style="width: 100%; height: auto;"></td>
</tr></table>

Hair scene under constant lighting. (1) $\sigma_a = [2, 2, 2]$ (2) $\sigma_a = [0, 0, 0]$ 

We can also see "white furnace" test under the complex scattering condition. With constant lighting, the hair with zero absorption coefficient will disappear, because the incident energy is equal to the outgoing energy.

---

I am very thankful to Mandy and Miguel for their guidance throughout this project!
