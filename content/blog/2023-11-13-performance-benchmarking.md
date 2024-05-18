+++
title = "Performance Benchmarking"
description = "Scrolling Tests with WeChat & TaoBao Apps"
[extra]
author = "Edward Tan"
website = "https://edwardtan.com"
twitter = "guofoo"
mastodon = "https://mastodon.social/@guofoo"
github = "guofoo"
+++

The last time we did a performance benchmark test, it was a few months ago. At that time we used only one simple application, an image manipulation program. During the GOSIM Workshop in September, I had mentioned that we would be doing more performance benchmarking tests and we have done just that.

This time we used Makepad to write a sample WeChat application and a sample TaoBao application. The main test criteria is to stress the "scrolling" feature of the apps. We also had access to two sample applications of the same written in Android "native".

## Test Methodology

Scrolling is one of the most important and common operations for mobile applications. For WeChat, a messaging application, and TaoBao, an e-commerce application, a smooth scrolling experience is often one of the key factors in user satisfaction with the application.

We benchmarked similar performance metrics as last time using Google [Perfetto](https://ui.perfetto.dev) tool. These include:

* CPU Cycles
* CPU Frequency (Average & Max)
* CPU Memory (Average & Max)
* GPU Memory (Average & Max)

To test, we use the applications' main message or product list page. We exercise the scrolling by quickly swiping up and down on this scrollable page. We first quickly swipe for 6 to 10 times up (to make the list go down) and then 6 to 10 times down and repeat, until we reach the 10 seconds sample time.

*(Note that in real applications the scrolling list might have network dependencies such as loading of images, etc. But for the sample applications, the images are cached locally to the mobile app so there’s no network latency or variance.)*

In order to have some more data points, we also benchmarked the official TaoBao app from the Google Play Store.

## Results

![](/blog/scrolling-test-table.png)

As can be seen from the results, they seem similar and consistent with our previous results from the image manipulation benchmarks.

### CPU Processing

![](/blog/cpu-cycles.png)

On the CPU cycle, the sample Makepad WeChat app shows very little cycles used, having less than half of the Android native sample app.

![](/blog/cpu-frequency.png)

The CPU frequencies also reflect that Makepad uses much less than the Android native counterparts.

### Memory Usage

The CPU memory usage is more similar among the Android native and Makepad sample applications, with Makepad apps using slightly less CPU memory. This is consistent with less usage of the CPU cycles as well.

![](/blog/cpu-memory.png)

The GPU memory shows that Makepad apps use more memory than the Android native apps. This is currently the only area where Makepad is not as efficient. However, the Makepad team is currently working on an improvement that will decrease this usage in the near future.

![](/blog/gpu-memory.png)

## Conclusion

Being a fully functioning application, the official TaoBao app had a much higher CPU cycle and CPU memory than the rest.

Also, there seems to be multiple processes related to the TaoBao application. We only counted the main TaoBao process. If we were to add the data from the TaoBao GPU process, then it will increase the amount of GPU memory usage by about 70 MB.

Overall, the results are consistent to what we found in our early benchmarking test. As Makepad framework continues to improve and evolve, the numbers will undoubtably change. We will periodically run more of these performance benchmarking and post our updates.

## References

Makepad WeChat sample:
[`https://github.com/project-robius/makepad_wechat`](https://github.com/project-robius/makepad_wechat)

Makepad TaoBao sample:
[`https://github.com/project-robius/makepad_taobao`](https://github.com/project-robius/makepad_taobao)
