## 15-428 Project: Parallelized Real-Time CPU Raytracing
Jocelyn Huang (jocelynh)

### Summary

I am going to modify and optimize an existing CPU-based raytracer to run in real-time and in parallel. To do so, I will be exploiting SIMD instructions and implementing and researching raytracing-specific optimizations to reduce memory and CPU usage.

### Background

Raytracing is a common rendering technique that involves computing the paths taken by rays that would hit a "camera lens" in the place of the screen. Since the individual path of any given ray emanating from a pixel of the image is independent of the paths of any other ray, this problem is embarassingly parallel. Therefore, the challenge lies in the computation-heavy nature of the problem, specifically, in speeding a raytracer up to be able to render in real-time.

```
**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)


For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/redoctopus/418-project-site/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
