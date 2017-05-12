# 15-428 Project: Parallelized Real-Time CPU Raytracing
Jocelyn Huang (jocelynh)

## Summary

My goal was to modify and optimize an existing CPU-based raytracer to run in parallel and at a framerate close to real-time, on a CPU. To get the raytracer up to speed, I explored a few parallelization methods including SIMD instructions and OpenMP, and implementing and researching raytracing-specific optimizations to reduce memory and CPU usage. All benchmarks and the final code were run on my 2011 Macbook Air, which has two cores (four hardware threads).

## Pre-Submission Overview

<!--### Challenges

There were a few major unforseen hurdles in parallelizing the the code, which had made optimizations for serial raytracing that unfortunately created many data dependencies and potential race conditions. This meant that I had to do some major refactoring in order to actually get pthreads working, which involved redesigning the code to remove reuse of data structures between individual rays and getting around class inheritance issues caused by the pthreads library not supporting the execution of C++ member functions. While doing this, I also made some small optimizations for cache locality, including changing some array accesses.

I also had to put some brainpower into animating the scene; the original format was not meant for being animated, as the models are themselves compiled C++ code and the traced scenes were originally just saved to .png images. This involved learning a bit of SDL to begin with, and then when the code was parallelized, figuring where to place synchronization code in order to get each frame rendered without race conditions. (It also turns out that MacOS doesn't actually implement pthread barriers, so that was also an adventure. I ended up borrowing code from [this library](http://blog.albertarmea.com/post/47089939939/using-pthreadbarrier-on-mac-os-x).)

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

I ended up switching from OMP to pthreads in order to avoid the overhead of repeatedly spawning and joining instances for every frame; with pthreads, I could simply use a barrier and have the next iteration of calculations starting while the current frame is being written to the screen, which also involved double-buffering so as to not cause screen tearing.
-->

## Background

Raytracing is a common rendering technique that involves computing the paths taken by rays that would hit a "camera lens" in the place of the screen. Since the individual path of any given ray emanating from a pixel of the image is independent of the paths of any other ray, this problem is embarassingly parallel, up until drawing the raytraced image to the screen. Therefore, the bulk of the challenge lies in the computation-heavy nature of the problem, specifically, in speeding a raytracer up to be able to render in real-time. An additional, not insignificant challenge is the constraint of running on a CPU; even if the problem is embarassingly parallel, we are still constrained to having four hardware threads, limiting the amount of true parallelism we can actually enjoy.

To this end, my initial goal is of course to parallelize the raytracing operations for multiple frames of a simple animation using multithreading. However, the main goal of this project is to speed up the CPU implementation enough for the raytracer to run in real-time, so we must of course address ways to hide or reduce the computational load. We notice that there are a few opportunities for instruction-level parallelism in the raytracing algorithm, either across pixels or across colors per pixel, which can be exploited using SIMD execution. There is also opportunity for hiding the latency of copying a buffer to the screen by pipelining operations in a sequence of frames, as drawing to the screen is essentially a serial operation (unless I start trying to parallelize SDL, which is a whole 'nother story). In addition, we can also tweak the code in order to make better use of data cache and to avoid sharing of cache lines in the drawing buffer.


## Approach

As per Kayvon's suggestion, I did not write a raytracer from scratch; instead, I found [an existing CPU-based raytracer by Don Cross](http://www.cosinekitty.com/raytrace/), which is written in C++, to modify and optimize. (Finding a suitable codebase was a bit more time-intensive than I was originally expecting--it's surprisingly hard to find a good raytracer that 1) does not already support multithreading and 2) is not too simplistic.)

To prime the code for my project and to get some reasonable baselines, I had to make a few changes to the code to begin with. The original code implemented antialiasing, which led to each frame taking more than five seconds to complete, so I decided to forego the antialiasing for my implementation. In addition, I had to edit the code to use SDL to draw the image directly to the screen, rather than write to a .png file. The original code also included several optimizations for serial execution which meant that several data structures were shared between pixels, so I took the time to restructure the data structures and functions where needed in order to make them async-safe after I had taken timings for a serial implementation baseline.

### Baselines

I chose to use the built-in "chessboard" image for my baseline timings, as the code uses compiled models in its raytracing rather than taking in a standard format, and the chessboard was one of the more complex and interesting ones included. The serial code with the original serial optimizations took 0.766 seconds/frame (1.3 frames/sec) averaged over several runs to raytrace for one frame on my machine.

In addition to a serial baseline, I also wrote an OpenMP implementation of the raytracer, partially as a warm-up and partially to get a baseline for a simple parallel approach, just to get an idea of what sort of speedup I could expect. Timings for these baselines can be found in the Results section below.

### Methods for Speedup


- pthreads
- thread spawn overhead reduction
- pipelining
- SIMD
- cache
- subsampling

I also looked into ways of reducing the actual amount of computation workload through heuristics, and found that an "adaptive subsampling" technique (somewhat explained [here](https://web-beta.archive.org/web/20100923081853/http://www.exceed.hu/h7/subsample.htm)) was useful in reducing the total number of rays that need to be traced, as part of the computational bound on this problem is the large number of rays to be traced in a single image. Essentially, subsampling takes every few consecutive pixels as sample points, checks if they are close in color, and interpolates between them if they are.

## Results

Baselines
The best performance I achieved with OpenMP was about 0.316 seconds/frame (3.2 frames/sec), a speedup of 2.4x. This is in fact quite reasonable

|   Type           |  Timing (s)  |
| ---------------- | ------------ |
|Serial Code       |      0.76552 |
| OMP static (4)   |      0.42122 |
| OMP static (8)   |      0.36092 |
| OMP static (16)  |      0.37554 |
| OMP dynamic (4)  |      0.33272 |
| OMP dynamic (8)  |      0.31678 |
| OMP dynamic (16) |      0.31736 |

- appropriate graphs, table from before + updates
- example images


## Possible Future Work

Shared resources, duplicate since no writes to the objects
SIMD for color class
