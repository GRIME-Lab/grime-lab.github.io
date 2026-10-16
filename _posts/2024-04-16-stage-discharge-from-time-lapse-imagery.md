---
title: "Filling Gaps in Streamflow Records with Documentary Time-Lapse Imagery"
excerpt: "Our PLOS Water paper tests whether photos taken for a documentary project can predict river stage and discharge during year-long gaps in a gage record."
header:
  teaser: /assets/images/blog/stage-discharge-time-lapse/weir-feature-regions.jpg
---

*By John E. Stranzl Jr., PhD*

Stream gages fail. Sensors are installed poorly, equipment breaks, and funding for monitoring programs lapses. When that happens, the stage and discharge record has a hole in it, and anyone who depends on that record for water management, flood forecasting, or model calibration has to work around it.

Our paper, published today in *PLOS Water*, asks a practical question: if a camera happens to be pointed at the river, can its images fill the gap?

> Chapman KW, Gilmore TE, Mehrubeoglu M, Chapman CD, Mittelstet AR, Stranzl JE (2024). Stage and discharge prediction from documentary time-lapse imagery. *PLOS Water* 3(4): e0000106. [https://doi.org/10.1371/journal.pwat.0000106](https://doi.org/10.1371/journal.pwat.0000106)

## A camera that wasn't built for hydrology

The images come from the [Platte Basin Timelapse](https://plattebasintimelapse.com/) project, a documentary effort that photographs the Platte River watershed. One of its cameras overlooks the North Platte River State Line Weir, about 20 meters from USGS stream gage 06674500 at the Wyoming-Nebraska state line.

The camera captured high-resolution (4288 × 2848) images every hour during daylight from 2012 to 2019. It was installed to tell a story, so it has no staff gage, calibration targets, or other reference points in view. It also moved over the years, by roughly ±4° in rotation and ±0.25 m in position.

![The State Line Weir scene, with the weir search region, bank anchor, and detected weir line marked](/assets/images/blog/stage-discharge-time-lapse/state-line-weir-search.jpg)

*Top: a typical image of the State Line Weir. Bottom: the same image showing the weir search region (green), the bank location (yellow), and the detected weir (red). Images from the Platte Basin Timelapse project, CC BY, with permission from Michael Farrell.*

## Turning images into numbers

Hand-labeling tens of thousands of images wasn't practical, so we computed 42 scalar features for each image automatically, using C++ and OpenCV. They fall into two groups:

- **Whole-image features** that can be computed for any photo: the mean and standard deviation of pixel intensity, entropy, and hue, saturation, and value.
- **Domain-informed features** that a hydrologist would recognize: the shape, color, and texture of the whitewater below the weir, and the color and texture of the water in regions above and below it.

![Annotated image showing the above-weir and below-weir regions, the whitewater contour, and the whitewater texture area](/assets/images/blog/stage-discharge-time-lapse/weir-feature-regions.jpg)

*Regions used for the domain-informed features. Images from the Platte Basin Timelapse project, CC BY, with permission from Michael Farrell.*

Because the camera drifted, the code first searches each image for the weir and fits a line to it before measuring anything relative to it.

Not every image cooperates. Of 55,804 images, features could be calculated for 42,059. The weir itself was found in 36,134 of those. Snow, ice, debris, high water, and frost on the lens hid it in the rest. Images that were too dark or too low in contrast were removed automatically. When the weir couldn't be found, its features were set to a "not calculable" value, and the models learned to rely on the other features for those images.

![Six example images: snow and ice, frost on the lens, high water, debris over the weir, a dark dawn image, and a typical good image](/assets/images/blog/stage-discharge-time-lapse/weir-not-found-examples.jpg)

*Examples where the weir could not be found: (A) snow, ice, and debris, (B) ice or frost on the lens, (C) high water, (D) debris over the weir, (E) a dawn or dusk image that is too dark. (F) is a typical good image. Images from the Platte Basin Timelapse project, CC BY, with permission from Michael Farrell.*

The whole feature calculation for about 40,000 high-resolution images ran in roughly 12 hours on a laptop with an i7 processor, 16 GB of RAM, and no GPU. That was deliberate. We wanted a method that students and other researchers could repeat without a computing cluster.

## Models and the gap test

We paired each image's features with the USGS stage and discharge measurement closest to its capture time, then trained three standard machine learning models: a multilayer perceptron (MLP), random forest regression, and support vector regression. A Kalman filter smoothed each model's predictions over time, since real stage and discharge change gradually from hour to hour.

To simulate real outages, we removed full water years (2015, 2016, and 2017) one at a time, trained on the 5,000 images before and 5,000 images after each gap, and asked the models to reconstruct the missing year.

## What we found

For 2015 and 2017, the models reconstructed the missing year well, with Nash-Sutcliffe Efficiency (NSE) between 0.83 and 0.90. The MLP performed best overall, with random forest close behind.

2016 was harder. Peak discharge that year reached about 240 m³/s, well above anything in the training data for that gap (the 2015 and 2017 peaks were about 175 and 125 m³/s). The models had never seen flows that high, and they underpredicted the peak.

![Observed and predicted stage and discharge for the 2015, 2016, and 2017 gap years](/assets/images/blog/stage-discharge-time-lapse/gap-year-observed-vs-predicted.jpg)

*Observed (blue) and predicted (orange) stage and discharge for each gap year. Only daylight hours with usable images are shown.*

To confirm the cause, we split 2016 into two half-year gaps, so the training data included that year's high flows. NSE rose to between 0.87 and 0.95.

The lesson applies to any gap-filling method: a model trained on a narrow range of conditions can't be trusted outside that range. Imagery has an advantage here. Images from the gap can be compared directly with images from the training period, and in 2016 the high water is obvious just by looking at them. A purely statistical gap-filling method gives you no way to see that the conditions were unusual.

Two other results surprised us:

- **The whole-image features mattered.** Features that have nothing to do with the weir contributed strongly to the models, in some cases about as much as the whitewater features. They also kept predictions going when the weir couldn't be found.
- **Hourly data is a strength.** Comparable studies worked at time steps from six hours to a month. Hourly images gave us finer-grained reconstructions. The tradeoff is that this camera took no images at night, so there are no nighttime predictions.

## Where this led

The process we built to develop, test, and refine image features for this paper is central to our ongoing development of **GRIME AI**, software designed so other researchers can run similar studies at their own sites without writing the pipeline from scratch.

The feature datasets, training and test sets, and models from the paper are available at [https://doi.org/10.32873/unl.dr.20210322](https://doi.org/10.32873/unl.dr.20210322).
