---
title: "`Compton_MLEM`, A GPU-Based Image Reconstruction Application in CUDA"
date : 2024-07-26T00:40:04-09:00
tags : ['LM-MLEM','CUDA','Simulation']
---

__In this post I want to try to reconstruct an image from the simulated data in the previous post. Supposedly the original source consists of two point sources close together around z= -0.5 cm. However, I found it hard to understand the properties of the reconstruction. I will need to redo this simulation with full a priori information about the simulated source.__

So, I have looked into the reconstruction application `Compton_MLEM` by Matt Leigh, Ref. [1]. It is written in CUDA Ref. [2], an extension of the C programming language with the ability to specify thread-level parallelism in C. This programming language was new to me, and I had a hard time simply trying to install the necessary software to make everything work (especially the nividia drivers caused me some headache)! I recommend doing the tutorial in Ref. [3] just to get a feeling for the language, and to demystify it. It is very similar to C++, but with some additional methods to parallelize the calculations.

The CUDA-program `Compton_MLEM` reconstructs an image from list-mode data using the Maximum-Likelihood Expectation Maximization Algorithm; an overview of the theory can be found in one of my [previous posts](http://plundhammar.github.io/posts/understanding-the-system-matrix-and-the-lm-mlem-algorithm/).

***
__Note:__ The `Compton_MLEM` does not work with the newest version of CUDA. It seems that the flag `sm_30` was deprecated when building the executable, see Ref. [4]. However, simply removing it in `makefile` works, or at least does not return any errors. That is, in `makefile` there is one value specifying the flags of the build `NVFLAGS = -arch=sm_30 '-std=c++11'` which I changed to `NVFLAGS = '-std=c++11'` and it seems to work.
***

In the repo of `Compton_MLEM` it seems that the folder `/MLEM_NO_MATRIX` is both the fastest and the best maintained part of this project. In it there are three main CUDA scripts found in the `/Source` folder:
- __CPUFunctions.cu__
- __GraphicsCardFunctions.cu__ and
- __Parallel_Reform.cu__
The __Parallel_Reform.cu__ is the main script and allocates memory and jobs between the CPU and GPU through the scripts __GraphicCardFunctions.cs__ and __CPUFunctions.cu__. I will come back to the functionality of these scripts in the future, there exists some questions regarding the calculation of some values.

Using the data in the ![previous post]() I tried to reconstruct the source distribution in the cube $X\times Y\times Z = [-10,10]\times [-10,10]\times [-10,10]~\mathrm{mm}^3$ in figure 1 and 2. The initial image is the back projected source distribution. It looks kind of strange. I would have guessed that the backprojection would have some symmetry properties along some axis. However, the 150th iteration looks less noisy and more closely like one (two?) point sources.

|       |  |
| ----------- | ----------- |
| ![Title](/movieI0_higher_resolution_larger.gif)      | ![Title](/movieI150_higher_resolution_larger.gif)       |
| __(a)__: The backprojected image initiating the algorithm.   | __(b)__: The 150th iteration.        |


__Figure 1__: Slices of the $z$-axis. 


|       |  |
| ----------- | ----------- |
| ![Title](/Image_I0_larger42.png)      | ![Title](/Image_I150_larger42.png)       |
| __(a)__: The backprojected image initiating the algorithm.   | __(b)__: The 150th iteration.        |


__Figure 2__: Slice of the $z$-axis.



Since I don't know exactly where the source is placed I can only guess. Looking at figure 1b it seems that we have overestimated the $X$ and $Y$ intervals, perhaps even the $Z$ interval. In figure 3 and 4 I consider the voxel space in $X\times Y\times Z = [1.0,1.0]\times [1.0,1.0]\times [5.0,5.0]~\mathrm{mm}^3$.

|       |  |
| ----------- | ----------- |
| ![Title](/movieI0_higher_resolution.gif)      | ![Title](/movieI150_higher_resolution.gif)       |
| __(a)__: The backprojected image initiating the algorithm.   | __(b)__: The 150th iteration.        |


__Figure 3__: Zoomed in version of figure 1. 

|       |  |
| ----------- | ----------- |
| ![Title](/Image_I15047.png)      | ![Title](/Image_I150_larger50.png)       |
| __(a)__: The backprojected image initiating the algorithm.   | __(b)__: The 150th iteration.        |


__Figure 4__: Zoomed in version of figure 2. 


Now it kind of looks like a circle emerges, with no real symmetric properties. I may have misunderstood the algorithm but these results seem strange to me. 

### Conclusion
The `Compton_MLEM` program returns an image. What exactly I'm looking at requires me to understand the CUDA script in depth. I have some queries on how the Klein-Nishina values are calculated but I can't really draw any conclusions as of yet. Furthermore, I do not know exactly where the source is positioned in the list-mode data I have (though this might be more realistic). I will redo this simulation study with another data set where the system parameters are known, and try to catch the logic in the code.
### References

[1] [Compton_MLEM](https://github.com/mattcleigh/Compton_MLEM)

[2] [CUDA](https://developer.nvidia.com/cuda-toolkit)

[3] https://cuda-tutorial.readthedocs.io/en/latest/tutorials/tutorial01/

[4] https://forums.developer.nvidia.com/t/ptxas-fatal-value-sm-30-is-not-defined-for-option-gpu-name/163708