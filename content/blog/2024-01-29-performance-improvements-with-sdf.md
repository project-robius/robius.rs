+++
title = "Performance Improvements with SDF"
description = "Preliminary Results for Latest Makepad with Updated SDF Generation Algorithm"
[extra]
author = "Edward Tan"
+++

Recently, the [Makepad](https://github.com/makepad/makepad) framework updated its font handling with SDF generation algorithms. We decided to test out the WeChat and TaoBao apps built using this latest iteration of the framework to see if there are any changes to performance. The tests are same as before: fast scrolling of the main content screen back and forth within a 10 second window.

The WeChat and TaoBao apps' code have pretty much stayed the same. The only change was made last month to work with Makepad's then-new `MatchEvent()` event handling paradigm, which simplified the code base and reduced the size of it by 20~30%.

The SDF algorithm changes in Makepad Framework itself is transparent to the application developer. It is a Makepad internal performance optimization.

## Results Summary

### Makepad TaoBao

![TaoBao Results Summary](/blog/makepad-sdf-taobao-results.png)

As can be seen from the preliminary results, the CPU Cycles of the application reduced by about 12%, while the average and max CPU frequency went up by 18% and 56% respectively. This is because the SDF generation takes more CPU processing initially to process the fonts, but once it's done, the overall CPU usage is reduced during subsequent rendering.

The CPU memory usage increased by an insignificant amount. But the GPU memory usage both reduced by over 23%. This is a significant improvement over the old Makepad way of using the Font Atlas feature. The dynamic SDF generation algorithm uses less than 1/4 of the GPU memory, a huge improvement.

### Makepad WeChat

![WeChat Results Summary](/blog/makepad-sdf-wechat-results.png)

Unlike the TaoBao app, the WeChat app is more "text-centric", with less images and mostly text-based rendering. The improvements here are more apparent.

The CPU cycles reduced by 24%, while the CPU frequency increases by average of 15% and max of 21% respectively.

The GPU memory usage reductions were even more significant, with 35% less average memory usage and 32% decrease in max memory usage.

The detailed chart of each of the metrics follows below.

## Individual Graphs

### CPU Cycles

On the CPU cycle, we see the significant reduction of CPU cycles with the new SDF algorithm as compared to the old Font Atlas rendering.

For the mostly text-based app such as WeChat, the results are more significant.

> _(For all graphs, the right side bar(s) is the new Makepad with SDF.)_

Makepad TaoBao | Makepad WeChat
--- | ---
<div style="display:flex">
  <div style="flex:1;padding-right:5px;">
    <img src="/blog/cpu-cycles-3.png" width="100%" alt="CPU Cycles" />
  </div>
  <div style="flex:1;padding-left:5px;">
    <img src="/blog/cpu-cycles-4.png" width="100%" alt="CPU Cycles" />
  </div>
</div>

### CPU Frequency

The only metric where the new method resulted in higher values due to running of the SDF algorithm.

Makepad TaoBao | Makepad WeChat
--- | ---
<div style="display:flex">
  <div style="flex:1;padding-right:5px;">
    <img src="/blog/cpu-frequency-3.png" width="100%" alt="CPU Frequency" />
  </div>
  <div style="flex:1;padding-left:5px;">
    <img src="/blog/cpu-frequency-4.png" width="100%" alt="CPU Frequency" />
  </div>
</div>

### CPU Memory Usage

The CPU memory usage had minimal increases compared to the previous method.

Makepad TaoBao | Makepad WeChat
--- | ---
<div style="display:flex">
  <div style="flex:1;padding-right:5px;">
    <img src="/blog/cpu-memory-3.png" width="100%" alt="CPU Memory" />
  </div>
  <div style="flex:1;padding-left:5px;">
    <img src="/blog/cpu-memory-4.png" width="100%" alt="CPU Memory" />
  </div>
</div>

### GPU Memory Usage

The GPU memory shows reflects the biggest difference between the old and new algorithms. Both averagea and maximum GPU memory usage were reduced by over 30%.

Makepad TaoBao | Makepad WeChat
--- | ---
<div style="display:flex">
  <div style="flex:1;padding-right:5px;">
    <img src="/blog/gpu-memory-3.png" width="100%" alt="GPU Memory" />
  </div>
  <div style="flex:1;padding-left:5px;">
    <img src="/blog/gpu-memory-4.png" width="100%" alt="GPU Memory" />
  </div>
</div>

## Conclusion

The results of these tests validates the benefits of the SDF generation algorithm that the latest Makepad has incorporated. CPU Cycles were reduced, indicating less processing after the initial ramp up. With SDF, Makepad can also cache the generation of the SDF on local storage to lower the CPU usage.

The GPU memory use reduction is the most significant difference. The usage of the SDF generation algorithm reduced GPU memory usage by a large amount. In addition, it will also provide improved performance for dynamically scaled texts.

## References

All tests were performed on the Android build of the Makepad TaoBao and Makepad WeChat apps, running on Google Pixel 7 Pro.  The tests were run a minimum of 6 times each, and the results were averaged.

Makepad WeChat sample app:
[`https://github.com/project-robius/makepad_wechat`](https://github.com/project-robius/makepad_wechat)

Makepad TaoBao sample app:
[`https://github.com/project-robius/makepad_taobao`](https://github.com/project-robius/makepad_taobao)

Makepad Framework:
[`https://github.com/makepad/makepad`](https://github.com/makepad/makepad)

SDF Generation Algorithm:
[`https://github.com/LykenSol/sdfer`](https://github.com/LykenSol/sdfer)
