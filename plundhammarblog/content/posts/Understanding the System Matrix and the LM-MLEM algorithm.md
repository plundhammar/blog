---
title: "Understanding the System Matrix and the LM-MLEM Algorithm"
date : 2024-07-26T00:40:04-07:00
tags : ['Analytical Methods','Image Reconstruction','System Matrix','LM-MLEM']
---

__The List-Mode Maximum Likelihood Expectation Maximization (LM-MLEM) algorithm is widely used in Compton camera reconstruction. It's well suited for the list-mode data architecture and easy to parallelize. This post will be somewhat technical and a sort of summary of the main arguments found in the litterature.__

The problem has been stated many times, Ref.[1-8]. I will state it as follows (I will use the same notation as in Ref.[8]), and the proof of it will be the subject of a future post:

Let $\lambda$ be the mean and variance of the Poisson distribution of some source. The following sequence converges to $\lambda$:
$$
\widehat{\lambda}\_j^{l+1} = \frac{\widehat{\lambda}\_j^{(l)}}{s\_j}\sum_{i=1}^N t_{ij}\frac{1}{p_i^{(l)}}~~\text{with}~~p_i^{(l)} = \sum_{k=1}^M t_{ik}\widehat{\lambda}^{(l)}_k
$$
where
- $t_{ij}$ are elements of the system matrix, indexed on the events $i$ and voxels $j$;
- $s_j=\sum_i t_{ij}$is the sensitivity correction.
The initialization step consists in calculating $\widehat{\lambda}^{(0)}$, generally as the back-projection in the image volume of the $N$ events. The question is now what the _system matrix_ is.

#### Probabilistic Model of the System Matrix
Let $x$ be the vector of coordinates of a point $M$ from the source. The probability of occurrence of the measured event $y$ given that $\mathfrak{p}$ was emitted at $x$ is given by

$$
p_{Y:X}(y:x,E_0) = \int p_{Y:\tilde{Y}}(y:\tilde{y},E_0)p_{\tilde{Y}:X}(\tilde{y}:x,E_0)\textrm{d}\tilde{y}.
$$

This is a trick found in statistical physics, and it can be seen as a variant of the Chapman–Kolmogorov equation Ref. [9]. We will now argue for what properties the two probability distributions $p_{Y:\tilde{Y}}(y:\tilde{y},E_0)$ and $p_{\tilde{Y}:X}(\tilde{y}:x,E_0)$ should have.

Considering $p_{Y:\tilde{Y}}(y:\tilde{y},E_0)$, this distribution could be summarized to have the task of answering the following question

- How likely is it that the measured event $y$ was registered given that the real event $\tilde{y}$ happened?

Since the measurement is based on the measurement and real event of the position in the first and second scatterer, as well as the measurement and real event of the scattering angle which we may assume to be independent we have

$$
p\_{Y:\tilde{Y}}(y:\tilde{y},E\_0) =p\_{V_1:\tilde{V}\_1}(V\_1:\tilde{V\_1},E\_0)p\_{V\_2:\tilde{V}\_2}(V\_2:\tilde{V}\_2,E\_0)p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta},E\_0).
$$

The terms $p\_{V\_i:\tilde{V}\_i}(V_i:\tilde{V_i},E_0)$ depends on the spatial resolution of the detector and something that we may impose depending on the detector design. The term $p_{\beta:\tilde{\beta}}(\beta:\tilde{\beta},E_0)$ depends on the energy resolution of the detectors, since the measured scattering angle $\beta$ is calculated through an energy dependent relation (Compton scattering with additional corrections perhaps).  

Now considering $p_{\tilde{Y}:X}(\tilde{y}:x,E_0)$, it is the product of several probabilities:

- The probability of the Compton scattering process (including Doppler-broadening);

- Absorption probabilities (in the different environments such as the object, air, absorption of the scattered photon in the scatter detector);

- The absorption probability of the photon in the absorber;

- The solid angle subtended at the origin of the particle $M$ by the detector element containing the first hit, __which should be considered in 3D since the emission is isotropic__, and on the solid angle subtended at the position of the first hit by the detector element where the second hit takes place, __which should be considered in 2D since it is related to a cone surface uncertainty through the scattering angle__.

The probability of the Compton scattering process is proportional to the Compton scattering cross-section given by the Klein-Nishina distribution, denoted $K(\tilde{\beta}:E_0)$ at energy $E_0$.

In general a solid angle considered in 3D may be written as $A/r^2$ where $A$ is the sector of the sphere of radius $r$ that the half-angle covers. Similarly, in 2D it may be written $A/r$ with the same definition. So, the solid angle subtended at the origin of the particle at $M$ can be written

$$
\frac{1}{\tilde{V}\_1M^2}\int\_{S\_{\tilde{V}\_1M}}\textrm{d}S\propto \frac{1}{\tilde{V}\_1M^2}\int\_{-\theta_{\tilde{V}\_1M^2}}^{\theta\_{\tilde{V}\_1M^2}}\sin(\theta)\textrm{d}\theta \propto \frac{\cos(\theta_{\tilde{V}_1M})}{\tilde{V}\_1M^2}.
$$

Similarly, the solid angle subtended at the position of the first hit by the detector element where the second hit takes place is proportional to
$$
\frac{\cos(\theta_{\tilde{V}_1\tilde{V}_2})}{\tilde{V}_1\tilde{V}_2},
$$
since we now consider it in 2D. Ignoring absorption probabilities (see Ref. [10,11] for a discussion of this), we can now write

$$
p\_{\tilde{Y}:X}(\tilde{y}:x,E\_0)\propto K(\tilde{\beta}:E\_0)\frac{\cos(\theta\_{\tilde{V}\_1M})}{\tilde{V}\_1M^2}\frac{\cos(\theta_{\tilde{V}_1\tilde{V}_2})}{\tilde{V}_1\tilde{V}_2}
$$

If the real positions are known (there in reality unobservable) $\tilde{\beta}$ can be calculated geometrically, let $\tilde{\beta}_{\tilde{V}_1,\tilde{V}_2,M}$ denote the geometrically obtained scattering angle. This may be estimated by measured values of the deposed energies and incident energy of the incident photon by
$$
\cos(\beta) = 1 - \frac{m_e c^2E_1}{E_0(E_0-E_1)}
$$
with $m_ec^2$ begin the mass energy of the electron. Inserting these probabilities and integrating results in

$$
p\_{Y:X}(y:x,E\_0) = \int p\_{Y:\tilde{Y}}(y:\tilde{y},E\_0)p\_{\tilde{Y}:X}(\tilde{y}:x,E\_0)\textrm{d}\tilde{y}
$$
$$
=\int\_{\mathbb{R}^3}\frac{\cos(\theta_{\tilde{V}\_1M})}{\tilde{V}\_1M^2}p\_{V\_1:\tilde{V}\_1}(V\_1:\tilde{V\_1},E\_0)
$$
$$
\times\int\_{\mathbb{R}^3}\frac{\cos(\theta\_{\tilde{V}\_1\tilde{V}\_2})}{\tilde{V}\_1\tilde{V}\_2}p_{V\_2:\tilde{V}\_2}(V\_2:\tilde{V}\_2,E\_0)
$$
$$
\times K(\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}:E_0)p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M},E\_0)\textrm{d}\tilde{V}\_2\textrm{d}\tilde{V}\_1
$$

We will assume that the uncertainty on the direction of the Compton cone axis may be considered as negligible compared to the uncertainty of the measured Compton scattering angle. Consequently, the real and measured points where $\mathfrak{p}$ hits the detector may be merged, leading to the simplified formulation:
$$
p\_{Y:X}(y:x,E\_0) =\frac{\cos(\theta\_{{V}\_1M})}{{V}\_1M^2}\frac{\cos(\theta\_{{V}\_1{V}\_2})}{{V}\_1{V}\_2}K(\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}:E\_0)p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}_{\tilde{V}_1,\tilde{V}_2,M},E_0)
$$

The element $t_{ij}$ of the system matrix $T$ is defined as:

- the probability of observing the physical event $y_i$ when a photon is emitted by the voxel $v_j$.

With our notation this may be written

$$
t\_{ij} =\int\_{v\_j} p\_{Y\_i:X}(y\_i:x,E\_0)p(x:M\in v\_j)\textrm{d}x
$$


$$
=\frac{1}{\textrm{vol}(v_j)}\int_{v_j} p_{Y_i:X}(y_i:x,E_0)\textrm{d}x
$$

where $p(x:M\in v\_j)$ is the probability that the physical event took place in $v_j$, and assuming that a voxel is sufficiently small to allow a constant probability inside the voxel. With the expression for $p_{Y_i:X}(y_i:x,E_0)$ we get

$$
t\_{ij}= \frac{1}{\textrm{vol}(v\_j)}\frac{\cos(\theta_{{V}\_1{V}\_2})}{{V}\_1{V}\_2}\int\_{v\_j}  \frac{\cos(\theta_{{V}\_1M})}{{V}\_1M^2}K(\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}:E\_0)p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M},E\_0)\textrm{d}x
$$

The only remaining thing is to model the probability $p_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}_{\tilde{V}_1,\tilde{V}_2,M},E_0)$, which describes the uncertainty of the measurement of the scattering angle, $\beta$. For an ideal detector this probability would be

$$
p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M},E\_0) = \delta(\beta-\tilde{\beta}_{\tilde{V}\_1,\tilde{V}\_2,M}).
$$

For a finite resolution detector, a convenient choice may be the Gaussian distribution with mean $\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}$ and standard deviation $\sigma\_{\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}}$,

$$
p\_{\beta:\tilde{\beta}}(\beta:\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M},E\_0) = \frac{1}{\sigma\_{\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}} \sqrt{2\pi}}\exp\left[\frac{(\beta-\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M})^2}{2\sigma^2\_{\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}}}\right].
$$

The unknown mean $\tilde{\beta}\_{\tilde{V}\_1,\tilde{V}\_2,M}$ may be replaced by the measured value $\beta$. The standard deviation can be calculated frm the knwn detector uncertainties $\sigma\_{\tilde{E}\_1}$ and $\sigma\_{\tilde{E}\_2}$ as:

$$
\sigma\_{\tilde{\beta}} = \frac{m\_ec^2}{E\_0^2\sin(\tilde{\beta})}\sqrt{\sigma\_{\tilde{E}\_1}^2+\frac{\tilde{E}\_1^2(\tilde{E}\_2+E\_0)^2}{\tilde{E}\_2^4}\sigma\_{\tilde{E}\_2}^2}
$$

It may be seen from the iterative equation above that the factor multiplying the integral in the equation for the system matrix element gets canceled in the iterative reconstruction process and may thus be ignored.

Even though the sensitivity matrix is given by the system matrix it is often approximated to something easier to calculate since the proper definition of the sensitivity matrix is usually a bottleneck in computation.

#### Conclusions
The LM-MLEM algorithm seems to be fruitful when considering the reconstruction problem of Compton cameras. Understanding the theory is an essential first step to understanding the practicalities of this algorithm. It is the case, as I have experienced, that when implemented naively, this algorithm is very computationally heavy. However, there are smart ways of going about it and the LM-MLEM algorithm have certain properties that makes it very efficient in the right framework.

#### References

1. Feng, et al. 3-D Reconstruction Benchmark of a Compton Camera Against a Parallel-Hole Gamma Camera on Ideal Data. IEEE Transactions on Radiation and Plasma Medical Sciences

2. Lehner, et al. 4pi Compton Imaging Using 3-D Position-Sensitive CdZnTe Detector Via Weighted List-Mode Maximum Likelihood. IEEE Transactions on Nuclear Science. [Link](https://ieeexplore.ieee.org/abstract/document/1323740)

3. Lange, et al. EM Reconstruction Algorithms for Emission and Transmission Tomography.

4. Wilderman, et al.Improved Modeling of System Response in List Mode EM Reconstruction of Compton Scatter Camera Images. IEEE Transactions on Nuclear Science [Link](https://ieeexplore.ieee.org/document/910840)

5. Cucci et al. List-mode MLEM Image Reconstruction from 3D ML Position Estimates. IEEE Nuclear Science Symposuim & Medical Imaging Conference.  [Link](https://ieeexplore-ieee-org.focus.lib.kth.se/document/5874269)

6. Wilderman et al. List-Mode Maximum Likelihood Reconstruction of Compton Scatter Camera Images in Nuclear Medicine. 1998 IEEE Nuclear Science Symposium Conference Record. 1998 IEEE Nuclear Science Symposium and Medical Imaging Conference (Cat. No.98CH36255). [Link](http://ieeexplore.ieee.org/document/773871/)

8. Caucci, et al. Maximum Likelihood Event Estimation and List-mode Image Reconstruction on GPU Hardware.IEEE Nuclear Science Symposuim & Medical Imaging Conference. [Link](https://ieeexplore.ieee.org/document/5874269)

9. Voichita Maxim, et al. Probabilistic models and numerical calculation of system matrix and sensitivity in list-mode MLEM 3D reconstruction of Compton camera images Phys.Med.Biol. 61 243. [Link](https://dx.doi.org/10.1088/0031-9155/61/1/243)

10. [Chapman-Kolmogorov Equation](https://en.wikipedia.org/wiki/Chapman%E2%80%93Kolmogorov_equation)

11. Wilderman, et al. List-mode maximum likelihood reconstruction of Compton scatter camera images in nuclear medicine Nuclear Science Symp. Conf. Rec. 3 1716–20

12. Parra. Reconstruction of cone-beam projections from Compton scattered data. IEEE Transactions on Nuclear Science. 47.4. 1543-1550 [Link](https://ieeexplore.ieee.org/document/873014)