# 15-428 Project: Parallelized Real-Time CPU Raytracing
Jocelyn Huang (jocelynh)

## Summary

I am modifying and optimizing an existing CPU-based raytracer to run in real-time and in parallel. To get the raytracer up to speed, I will be exploring parallelization methods including SIMD instructions and OpenMP, and implementing and researching raytracing-specific optimizations to reduce memory and CPU usage.

## Pre-Submission Overview

### Challenges

There were a few major unforseen hurdles in parallelizing the the code, which had made optimizations for serial raytracing that unfortunately created many data dependencies and potential race conditions. This meant that I had to do some major refactoring in order to actually get pthreads working, which involved redesigning the code to remove reuse of data structures between individual rays and getting around class inheritance issues caused by the pthreads library not supporting the execution of C++ member functions. While doing this, I also made some small optimizations for cache locality, including changing some array accesses.

I also had to put some brainpower into animating the scene; the original format was not meant for being animated, as the models are themselves compiled C++ code and the traced scenes were originally just saved to .png images. This involved learning a bit of SDL to begin with, and then when the code was parallelized, figuring where to place synchronization code in order to get each frame rendered without race conditions. (It also turns out that MacOS doesn't actually implement pthread barriers, so that was also an adventure. I ended up borrowing code from [this library](http://blog.albertarmea.com/post/47089939939/using-pthreadbarrier-on-mac-os-x).)

Currently, I am attempting to vectorize the code in order to obtain a better speedup.

### Preliminary Results

I have managed to get a slightly greater than 2x speedup on my machine, with some benchmarked timing as follows:

|   Type           |  Timing (s)  |
| ---------------- | ------------ |
|Serial Code       |      0.76552 |
| OMP static (4)   |      0.42122 |
| OMP static (8)   |      0.36092 |
| OMP static (16)  |      0.37554 |
| OMP dynamic (4)  |      0.33272 |
| OMP dynamic (8)  |      0.31678 |
| OMP dynamic (16) |      0.31736 |
| pthreads (1)     |      0.69666 |
| pthreads (4)     |      0.31975 |
| pthreads (8)     |      0.31196 |

I hope to have SIMD results by Friday, with a corresponding speedup graph. If time allows, I will also be making some optimizations to the code to make use of SIMD cache locality.

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
(Last updated 04/25/2017)

### Week 1 (4/10-4/15)
- Research and finalize raytracing codebase to be used as the baseline

### Week 2 (4/16-4/22)
- Familiarize with codebase, get a feel for how everything is laid out and how it works
- Build and have baseline timings

### Week 3 (4/23-4/29)
- Implement naive multithreading
- Tuesday, April 25: Checkpoint!
- Modify output stream to draw to screen instead of .png (SDL)
- Rotation and re-draw

### Week 4 (4/30-5/6)
- Start a naive SIMD implementation
- Start framework for adaptive subsampling
- Implement adaptive subsampling, record speedup
- Work out work distribution and analyze amount of divergent execution caused by adaptive subsampling

### Week 5 (5/7-5/12)
- If necessary, look at alternative scheduling methods to minimize divergent execution
- Thursday, May 11: Finalize presentation and report
- Friday, May 12: Report due & Parallelism Competition Day!

## Checkpoint Report

So far, I have found a suitable C++ raytracing codebase (see the [site](http://www.cosinekitty.com/raytrace/)), and have timed a few simple benchmarks that I will be trying to improve on. The former required trawling the web for a raytracer that 1) did not already implement multithreading, as that would defeat the purpose of this project, and 2) was not too naive. After deciding on the codebase, I read through the relevant code and some of the documentation to figure out the code structure, and made sure that everything built and ran on my machine without any issues. Most recently, I added some timing code for the test images such that I have a baseline to refer to post-threading.

Finding a raytracer that fit both criteria mentioned was a bit of an ordeal (it was pretty hard to find a middle ground, and I ended up leaning more towards "simplistic but functional"), so I have had to push back some of my schedule. I also added a task for changing the way that the existing code writes files and another task for animating (just rotation for now), as I would like to have a way to write to the screen instead of directly to a .png file such that I can see how my edits do directly.

Given my current schedule, my goals are slighty modified:
1. Parallelize the raytracing per ray, and across the entire rendered image
2. Modify the parallel implementation to use SIMD and optimize with respect to cache accesses and SIMD instructions, aiming for a 4x speedup
3. REALLY HOPE TO ACHIEVE: Implement the adaptive subsampling technique, aiming to get to real-time raytracing
4. HOPE TO ACHIEVE: Additional speedup, to be determined from further research

In short, I am not entirely sure how long optimizing the code (#2) will take, which may mean that I don't have much time for #3. I still hope to have some speedup graphs comparing threaded performance and optimized threaded performance to the baseline speeds for a few test images. If at that point the animation is not painful to watch (hmm... to be determined what this means after I get a better grasp of speedup), I would also like to show a demo animation.

Current issues:
1. Finagling the code in the codebase--it seems pretty well-documented, but it's not the cleanest, so some of it still needs a bit of dedicated sit-down examining time before I will feel comfortable editing it in a big way.
2. Mostly just getting the code to work; there are a few warnings that I need to tamp down, hopefully without breaking anything vital.
