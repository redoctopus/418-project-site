# 15-428 Project: Parallelized Real-Time CPU Raytracing
Jocelyn Huang (jocelynh)


## Summary

I am going to modify and optimize an existing CPU-based raytracer to run in real-time and in parallel. To do so, I will be exploiting SIMD instructions and implementing and researching raytracing-specific optimizations to reduce memory and CPU usage.


## Background

Raytracing is a common rendering technique that involves computing the paths taken by rays that would hit a "camera lens" in the place of the screen. Since the individual path of any given ray emanating from a pixel of the image is independent of the paths of any other ray, this problem is embarassingly parallel. Therefore, the challenge lies in the computation-heavy nature of the problem, specifically, in speeding a raytracer up to be able to render in real-time.

To this end, my initial goal is of course to parallelize the raytracing operations using multithreading. However, the main goal of this project is to speed up the CPU implementation enough for the raytracer to run in real-time. We first recognize that the operations in tracing any given ray do not change much across individual rays. So in order to attain the needed speedup for real-time rendering, I will first be exploiting instruction-level parallelism in the form of SIMD instructions to increase the rate at which work can be done.

In addition, I will be tweaking the code in order to make good use of the cache, which may be especially important when running multiple elements in a SIMD vector at a time, and see if improvements can be made by constraining the working set at any given time. I also plan to try implementing an "adaptive subsampling" technique (somewhat explained [here](https://web-beta.archive.org/web/20100923081853/http://www.exceed.hu/h7/subsample.htm)) in order to reduce the total number of rays that need to be traced, as part of the computational bound on this problem is the large number of rays in a single image.


## The Challenge

The main challenge here is not in maximizing the amount of parallelism in the program, but in speeding up and optimizing the operations needed for rendering a single frame such that the raytracer can produce real-time results (around 30fps). The work itself is very parallelizable, as the workload itself consists of a great many independent operations, but the computation for each ray also needs to access the metadata for the triangles it intersects, and the fact still stands that there are many individual rays that need to be computed for the entire image. Also note that a given CPU cannot run all that many threads at once (and certainly not as many threads in parallel as a GPU could), so it will be an interesting experience to try to see how well I can do with CPU parallelism.


## Resources

As suggested by Professor Kayvon, I will not be writing a renderer from scratch; instead, I will be modifying an existing CPU-based raytracer to run in parallel and then optimizing it. I have yet to find a good starting codebase, and would appreciate help in doing so. I would also appreciate having access to a machine that has a nicer CPU than the 2011 Macbook Air that I currently use, although that would certainly add to the challenge.


## Goals & Deliverables

### Goals
1. Parallelize the raytracing per ray, and across the entire rendered image
2. Modify the parallel implementation to use SIMD and optimize with respect to cache accesses and SIMD instructions, aiming for a 4x speedup
3. Implement the adaptive subsampling technique, aiming to get to real-time raytracing
4. HOPE TO ACHIEVE: Additional speedup, to be determined from further research

### Deliverables
When I demo my final project, I hope to show at least one real-time raytraced animation live, and give a general metric for the number of frames that can be rendered per second. I also hope to show speedup graphs comparing my implementation at various implementation checkpoints to the original renderer that my code will be based off of. The original, serial renderer will be the baseline for comparison for this project and the speedup achieved.


## Platform Choice

The restriction of using the CPU for parallel rendering (as opposed to the GPU) provides the point of interest in this problem. I will be using C/C++/possibly inline assembly for this task, as speed is of the essence, and since these languages provide readily available low-level hardware access that may be needed for some optimizations.


## Schedule

### Week 1 (4/10-4/15)
- Research and finalize raytracing codebase to be used as the baseline
- Familiarize with codebase, get a feel for how everything is laid out and how it works

### Week 2 (4/16-4/22)
- Implement naive multithreading
- Start a naive SIMD implementation
- If time allows, start optimizations and tweaks to SIMD to aim for 4x speedup

### Week 3 (4/23-4/29)
- Finish up with SIMD implementation and optimizations
- Tuesday, April 25: Checkpoint! (Have a working and nontrivial SIMD implementation, record speedup achieved)
- Thursday, April 27: If time allows, start working on a framework for adaptive subsampling

### Week 4 (4/30-5/6)
- Finish framework for adaptive subsampling
- Implement adaptive subsampling, record speedup
- Work out work distribution and analyze amount of divergent execution caused by adaptive subsampling

### Week 5 (5/7-5/12)
- If necessary, look at alternative scheduling methods to minimize divergent execution
- Thursday, May 11: Finalize presentation and report
- Friday, May 12: Report due & Parallelism Competition Day!
