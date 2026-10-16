---
title: "GRIME2: Open-Source Software for Measuring Water Level in Images"
excerpt: "Our Water Resources Research technical note introduces GRIME2, free software that measures water level from camera images of a calibration target."
header:
  teaser: /assets/images/blog/grime2-water-level/water-level-result-overlay.jpg
---

*By John E. Stranzl Jr., PhD*

Water level sensors report a number. They can't show you whether that number is right. A pressure transducer drifts, a float sticks, ice forms, debris piles up, and the sensor keeps reporting. A camera pointed at the same spot records both the measurement and the conditions behind it, so anyone can look at the image and check.

Our technical note in *Water Resources Research* describes **GRIME2**, free, open-source software that measures water level automatically from images.

> Chapman KW, Gilmore TE, Chapman CD, Birgand F, Mittelstet AR, Harner MJ, Mehrubeoglu M, Stranzl JE Jr. (2022). Technical note: Open-source software for water-level measurement in images with a calibration target. *Water Resources Research* 58, e2022WR033203. [https://doi.org/10.1029/2022WR033203](https://doi.org/10.1029/2022WR033203)

## What GRIME2 is

GRIME2 stands for GaugeCam Remote Image Manager-Educational 2. The "2" distinguishes this open-source program from the original GRIME, which was lab software written for a single research project and wasn't built for other people to use.

GRIME2 is written in C++ on open-source libraries (OpenCV, Boost, ExifTool, Qt, and ffmpeg) under commercial-friendly licenses. It comes as a Windows program with a graphical interface and as a command-line program for batch processing. Anyone can use it, change it, or build it into their own work.

The methods behind it were refined over three studies:

- A **laboratory study** that measured water level to within ±3 mm under tightly controlled conditions.
- A **field deployment** at a remote research site in a North Carolina tidal marsh.
- A **field comparison** with traditional sensors that measured within ±5 mm (90% confidence interval) over about 1 m of range in a tidal creek.

For comparison, the USGS accuracy standard for stage is 0.01 ft (about 3 mm) or 0.2% of effective stage.

## The calibration target

GRIME2 needs a dedicated target in the water: a flat, white, vertical board printed with two columns of bow-tie shapes. The water line shows up clearly against the white background, and the bow-ties give the software reference points with known positions.

![Diagram of the bow-tie target installed in a small stream, and a suggested bracing method](/assets/images/blog/grime2-water-level/calibration-target-installation.jpg)

*(A) The target installed in a small stream. (B) A suggested way to brace it so it stays perpendicular to the water surface. From Chapman et al. (2022), CC BY-NC-ND 4.0.*

The target has to stand perpendicular to the water surface and stay flat. If it tilts or warps, accuracy suffers.

## Calibrating: from pixels to centimeters

A camera reports positions in pixels. A hydrologist needs centimeters. GRIME2 bridges the two with one calibration image in which all eight bow-ties are visible.

1. The software finds the center of each bow-tie by matching a synthetic bow-tie template, rotated from −10° to 10° in 1° steps, and then refines each center to a fraction of a pixel.
2. The user supplies the real-world position of each bow-tie center in a small CSV file.
3. From the two sets of points, GRIME2 builds the transform that converts any pixel position on the target into a real-world measurement, and back again.

![Calibration overlay showing the bow-tie grid, world-coordinate grid lines, the waterline search area, and the camera-movement search areas](/assets/images/blog/grime2-water-level/calibration-overlay.jpg)

*A calibration result: the bow-tie grid and real-world heights (yellow), the waterline search area (blue), and the search areas used to detect camera movement (red). From Chapman et al. (2022), CC BY-NC-ND 4.0.*

Cameras move a little over time, from wind or from soil shifting around the post. On every image, GRIME2 finds the top two bow-ties, compares their positions with the calibration image, and corrects the water level for any shift.

## Finding the water line

Inside the blue search area, the program:

1. Cleans up the image to suppress biofouling on the target while keeping the water line intact.
2. Splits the search area into 10 narrow vertical strips.
3. In each strip, sums the brightness of every row and looks for the sharpest drop from light (target) to dark (water). That drop is the water line, located to a fraction of a pixel.
4. Fits a straight line through the strip results with RANSAC, a method that ignores stray points.
5. Converts the line's position at the center of the search area to a real-world water level.

![Five panels showing the search area, the vertical strips, the brightness profile of one strip, the edge point found in that strip, and the final fitted water line](/assets/images/blog/grime2-water-level/waterline-search-steps.jpg)

*The waterline search: (A) search area, (B) vertical strips, (C) brightness profile and its derivatives for one strip, (D) the edge found in that strip, (E) all strip results and the fitted water line. From Chapman et al. (2022), CC BY-NC-ND 4.0.*

If fewer than five strips find a valid edge, or the fitted line is tilted more than the user allows, the image is flagged as a failed measurement. Fog, shadows, glare, debris, and wildlife are the usual causes. Every result can be checked by eye, because GRIME2 draws its answer on the image.

![Overlay image showing a successful water level measurement with the level, camera-movement adjustment, and adjusted level printed in the corner](/assets/images/blog/grime2-water-level/water-level-result-overlay.jpg)

*A successful measurement, with the water level, the camera-movement adjustment, and the adjusted level printed in the corner. From Chapman et al. (2022), CC BY-NC-ND 4.0.*

## From a folder of images to a hydrograph

GRIME2 can process a single image, a folder, or a whole folder tree. It writes a CSV file with each image's file name, timestamp, water level, line angle, and camera-movement adjustment, along with the annotated images. It can also make GIF animations for checking results or for presentations. Everything in the graphical program can also run from the command line, so large image archives can be processed in batch.

We ran 2,975 images from the North Carolina tidal marsh, taken in February and March 2012. The water line was found in 98.8% of them. Only 35 images failed, a small enough number to review by hand.

![Water level over March 2012 at the North Carolina tidal marsh site, showing the tidal cycle](/assets/images/blog/grime2-water-level/tidal-marsh-water-level-march-2012.jpg)

*Tidal water levels for March 2012, built entirely from the GRIME2 CSV output. From Chapman et al. (2022), CC BY-NC-ND 4.0.*

GRIME2 is already in use outside our group, for research at the University of Kansas and for operational monitoring at Idaho Power.

## Limits and next steps

GRIME2's main limitation is the target. It needs a bow-tie board fixed in the water, and that isn't practical everywhere. The paper points to future work on methods that need a smaller target, or none at all. It also identifies places where the community can contribute: better subpixel and pattern-finding methods, calibration that tolerates a tilted target, and preprocessing tuned to other kinds of water scenes.

The source code is on GitHub, and installers, tutorials, and target setup guides are at [gaugecam.org](https://gaugecam.org).
