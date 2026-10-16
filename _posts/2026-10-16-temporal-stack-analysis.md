---
title: "Temporal Stack Analysis: What 125 Images of the Pecos River Can Tell Us"
excerpt: "A proof of concept in GRIME AI that summarizes every pixel of a fixed camera's image series with six statistics."
header:
  teaser: /assets/images/blog/temporal-stack-analysis/pecos-mean.jpg
---

*By John E. Stranzl Jr., PhD*

A fixed camera takes the same picture over and over. Because the camera doesn't move, each pixel location sees the same spot on the ground in every frame. That means each pixel can be treated as its own time series and summarized.

In March I built a proof of concept in GRIME AI called **Temporal Stack Analysis**. You point it at a folder of images from one camera, and it computes six per-pixel statistics across the whole set (the "stack"). Each statistic is saved as an image.

This post walks through the six outputs using 125 images from the USGS camera on the Pecos River near Acme, New Mexico.

## How it works

For every pixel location, and for each color channel, the tool collects the value from every image in the stack, computes a statistic, and writes the result back out as an image the same size as the originals.

Before stacking, images can be filtered by blur, exposure, and outlier checks, and by time of day and date range. For this run I used no filters, so all 125 images went into the stack. That choice matters, as you'll see below.

## Mean

The average value of each pixel across all images. This is the long-term "normal" appearance of the scene. Things that come and go, such as glare on the water, animals, or people, fade out.

![Mean of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-mean.jpg)

The river, banks, and utility pole stay sharp because they don't move. The date and time stamp in the upper left changes in every frame, so it averages into a smear.

## Median

The middle value of each pixel across all images. It looks much like the mean, but it is more resistant to one-time events. A value that shows up in fewer than half the frames, such as a bird on a gauge or a piece of floating debris, is largely removed.

![Median of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-median.jpg)

## Standard deviation

How much each pixel varies over time. Bright areas change a lot (flowing water, moving vegetation, changing water levels). Dark areas stay about the same (fixed structures, stable ground). Think of it as a map of where change happens.

One note: this output is stretched to use the full brightness range, so brightness is relative within the image and can't be compared directly across cameras.

![Standard deviation of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-std-deviation.jpg)

In the Pecos stack, the sky varies more than anything else in the scene, because lighting changes from frame to frame.

## Minimum

The darkest value each pixel reached in any image. Useful for finding permanent shadows, sensor vignetting, or the lowest extent of water.

![Minimum of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-minimum.jpg)

## Maximum

The brightest value each pixel reached in any image. Useful for finding the highest water extent, flood inundation, or overexposure.

![Maximum of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-maximum.jpg)

The sky is washed out to white, and the sun's glare sits on the horizon at the left. At least one image in the stack was taken with the sun low in the camera's view.

## Difference (maximum minus minimum)

The full range each pixel covered. It is related to standard deviation, but more sensitive to rare events. A single flood image or a single ice image shows up strongly here, while standard deviation would dilute it across the whole stack.

![Difference (maximum minus minimum) of 125 images from the Pecos River near Acme, NM camera](/assets/images/blog/temporal-stack-analysis/pecos-difference.jpg)

## What the Pecos run taught us

The minimum, maximum, and difference images are dominated by lighting. The darkest values come from low-light frames and the brightest from frames with the sun in view, so the range is large almost everywhere. Any change in the river itself is hidden under the change in light.

This is why the time-of-day filter exists. Restricting the stack to a midday window keeps lighting roughly constant, so the remaining variation is more likely to come from the scene itself: water level, vegetation, and sediment.

## Where this could go next

**Single-image comparison.** Subtract the mean from one chosen image to show how that moment differs from normal. A flood image would highlight ground that is usually dry. An ice image would highlight ice cover. The choice of image is the question being asked, so the natural choice is the image you are already looking at in GRIME AI.

**Sliding window.** Right now the tool summarizes one whole folder. Running it over a moving window (for example, three days at a time) opens up other uses. One is smoothing segmentation results: the mean of a window of binary vegetation masks gives the fraction of images in which each pixel was labeled vegetation. That reduces the image-to-image flicker that segmentation models produce along the edges of vegetation patches.

## Tell us what you would use it for

This is a proof of concept, and the six outputs are a starting point. If you work with time-lapse imagery and have a use for these statistics, or a statistic we should add, we would like to hear from you. See the [Contact](/contact/) page.
