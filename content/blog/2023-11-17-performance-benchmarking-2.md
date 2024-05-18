+++
title = "Performance Benchmarking - Part 2"
description = "Scrolling Tests with WeChat & TaoBao Apps Updated"
[extra]
author = "Edward Tan"
website = "https://edwardtan.com"
twitter = "guofoo"
mastodon = "https://mastodon.social/@guofoo"
github = "guofoo"
+++

Several days ago we did a [performance benchmark](/blog/performance-benchmarking) test between sample apps written in Android "native" vs. Makepad. Afterwards, we were given access to another version of the sample TaoBao app with more functionality, including more animation/video content in the scroll list, and the ability to turn toggle some of the special effects.

So we did the same scrolling tests with this new version, both with the special features turned on, as well as with all features turned OFF, in order to compare the differences.

## Updated Results

![](/blog/scrolling-test-table-2.png)

As can be seen from the results, they seem similar and consistent with our previous results from the image manipulation benchmarks.

### CPU Processing

![](/blog/cpu-cycles-2.png)

On the CPU cycle, the sample Makepad WeChat app shows very little cycles used, having less than half of the Android "native" sample apps.

The Native TaoBao apps were similar to the Official TaoBao in that they use a lot more CPU cycles. This also results in higher CPU frequency values.

When special functionality are turned OFF, the Native TaoBao used ~40% less cycles. While the full-effect TaoBao used slightly more than the Official TaoBao app.

![](/blog/cpu-frequency-2.png)

The CPU frequencies also reflect that Makepad uses much less than the Android native counterparts.

Not much difference were noticed with the new Native TaoBao versions for GPU memory usage.

### Memory Usage

The CPU memory usage is more similar among the Android native and Makepad sample applications, with Makepad apps using slightly less CPU memory. This is consistent with less usage of the CPU cycles as well.

As expected, the new Native TaoBao performed noticeably better when its special effects were turned off.

![](/blog/cpu-memory-2.png)

The GPU memory shows that Makepad apps use more memory than the Android native apps. This is currently the only area where Makepad is not as efficient. However, the Makepad team is currently working on an improvement that will decrease this usage in the near future.

Not much difference were noticed with the new Native TaoBao versions for GPU memory usage.

![](/blog/gpu-memory-2.png)

## Conclusion

Being a more fully developed Android Native TaoBao app, we noticed much smoother and faster scrolling speed and effects compared to the previous simpler "native" apps. The performance characteristics of this version matches more closely to the official Play Store TaoBao, though with less CPU memory footprint.

Overall, the results still reflect what we had found in our previous benchmarking tests. The Makepad versions of the apps have consistently outperformed all other versions, at least in terms of pure scrolling tests.

## References

Makepad WeChat sample:
[`https://github.com/project-robius/makepad_wechat`](https://github.com/project-robius/makepad_wechat)

Makepad TaoBao sample:
[`https://github.com/project-robius/makepad_taobao`](https://github.com/project-robius/makepad_taobao)
