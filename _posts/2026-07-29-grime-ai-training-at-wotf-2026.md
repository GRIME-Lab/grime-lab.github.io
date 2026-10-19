---
title: "GRIME AI and SAGE at the 2026 Water Observing Technology Forum"
excerpt: "Turning existing camera networks into non-contact, AI-enhanced water sensors: a GRIME AI and SAGE training at the USGS Hydrologic Instrumentation Facility."
header:
  teaser: /assets/images/blog/wotf-2026/slide-pipeline.jpg
---

*By John E. Stranzl Jr., PhD*

Today I presented GRIME AI and SAGE at the [2026 Water Observing Technology Forum](https://ciroh.ua.edu/water-observing-technology-forum/) (WOTF) in Tuscaloosa, Alabama. My PhD advisor, Troy Gilmore, joined me at the forum.

## About the forum

WOTF is hosted by the Cooperative Institute for Research to Operations in Hydrology (CIROH) at the University of Alabama. The planning committee also includes the Alabama Water Institute, the USGS Next Generation Water Observing System and Hydrologic Instrumentation Facility, NOAA, and the Alliance for Coastal Technologies.

This year's theme was non-contact water monitoring technologies enhanced with artificial intelligence. The forum ran July 28 to 30. The second day was held at the USGS Hydrologic Instrumentation Facility (HIF) and was devoted to hands-on training workshops, including a track on image-based monitoring.

![Entrance of the USGS Hydrologic Instrumentation Facility with the USGS sign](/assets/images/blog/wotf-2026/usgs-hif-entrance.jpg)

*The USGS Hydrologic Instrumentation Facility in Tuscaloosa, home to the USGS instrument development and testing labs, a hydraulics lab, and training classrooms. Photo: PHOTO_CREDIT.*

![The long indoor tow tank in the HIF hydraulics lab, seen from above](/assets/images/blog/wotf-2026/hif-tow-tank.jpg){: .align-center}

*The 350-foot tow tank in the HIF hydraulics lab. [Watch my video of the lab on LinkedIn](https://www.linkedin.com/posts/johnestranzl_last-week-at-the-usgs-hydrologic-instrumentation-ugcPost-7490236642847510528-WFtx/).*

## The camera is already a non-contact sensor

The forum asked how non-contact monitoring, enhanced with AI, changes water operations. My answer was that the non-contact sensor is already in the field. More than 1,000 USGS HIVIS webcams and more than 1,000 PhenoCam sites are recording today, along with NEON cameras, state mesonets, and others. The USGS installs its cameras for visual context and verification. GRIME AI adds the missing layer: quantitative water measurements from those same images.

The bottleneck is the pipeline that turns pixels into hydrology: retrieval, quality control, segmentation, and validation at scale. Every camera already installed, and every new one, becomes another instrument GRIME AI can read.

For operations, non-contact sensing has practical advantages. Nothing sits in the channel to be buried, swept away, or fouled during the high-flow events that matter most. Analysis runs on imagery the camera already transmits, which means fewer site visits, and observation continues through floods.

## One pipeline from image to water data

GRIME AI carries imagery through four stages: retrieve imagery from multiple monitoring networks, screen and standardize it, segment water in every frame with a SAM2-based model, and analyze the results against held-out data.

![Slide showing the four GRIME AI stages: retrieve, quality control, segment, and analyze](/assets/images/blog/wotf-2026/slide-pipeline.jpg)

A single retrieval layer covers USGS HIVIS, NEON, and PhenoCam, so a new site on any of those networks is ready for analysis without custom integration work. Managing API keys and tracking endpoint changes, such as the USGS move to its new NIMS service, keeps that access working over time.

### Built to run without a GUI

The measurement path runs headless. Image triage, composite slices, COCO dataset handling, segmentation, training, and region-of-interest feature extraction are all command-line subcommands that can run unattended on a schedule, on a data-center server or an edge box at the site. Settings come from a configuration file, so a run repeats identically, and the results come out as CSV or Excel files. Downloading imagery from the networks still happens in the GUI.

![Slide showing the headless GRIME AI workflow from camera to edge node or server to telemetry or database](/assets/images/blog/wotf-2026/slide-headless.jpg)

## SAM2, tuned for water

GRIME AI fine-tunes SAM2, a promptable foundation segmentation model, on annotated water imagery for each site and season. Two engineering details make a real difference in accuracy:

- **Run-length-encoded masks** preserve complex shapes such as water around sandbars, braided channels, and rings, which polygon outlines lose.
- **A Mahalanobis blob filter** fits a 2-D Gaussian to the centroids of the ground-truth water regions and rejects detections that don't fit, a more reliable test than a simple size threshold.

## SAGE: where the models learn what water looks like

SAGE is the human-in-the-loop annotation tool that produces GRIME AI's ground truth. Annotators draw masks with guided tools or accept and correct masks proposed by SAM2. Annotations are stored in COCO format and go straight into training and holdout validation in GRIME AI: annotate, train, validate, refine, and annotate again.

![Slide describing SAGE and the annotation-to-model loop](/assets/images/blog/wotf-2026/slide-sage.jpg)

## What holdout testing taught us

We test models with leave-one-season-out holdouts: the model is scored on a season it never saw in training. Accuracy tracked how well the training set covered the scene conditions in the held-out season. Where a holdout contained conditions missing from training, such as exposed sandbars, a braided channel, heavy vegetation, or ice, the overlap between predicted and true water (the Dice score) dropped sharply. Season is a stand-in for what the camera sees. The design goal is to train across the full range of scenes a camera will encounter.

![Slide with a bar chart of leave-one-season-out Dice scores for the Pecos and Grand Island sites](/assets/images/blog/wotf-2026/slide-holdout.jpg)

GRIME AI builds accuracy checks into the workflow: the one-click holdout pipeline reports Dice, ROC and precision-recall curves, confusion matrices, and F1 across thresholds; inter-annotator comparison measures how much the human labels agree before the model is trusted; image triage screens frames before they reach the model; and automated PDF reports make every run auditable.

## From research to operations

GRIME AI is open source on [GitHub](https://github.com/GRIME-Lab/GRIME-AI), installable through the GRIMELab Anaconda channel and Docker (see the [Download](/download/) page), and its design was shaped by USGS funding around agency workflows.

The next steps move intelligence closer to the point of measurement: slimming the models so inference can run on the camera itself, and adding more camera networks.

After the overview, participants worked through a hands-on GRIME AI tutorial.

## Thanks

Thank you to the WOTF organizers and to the SAGE annotation team, whose careful labeling work everything else rests on: R. Bin Issa, J. Blais, K. Johnson, C. Terry, D. Kosmicki, M. Smith, M. Wells, and J. S. Camacho Puerto.

If you'd like help installing GRIME AI or applying it to your own cameras, reach out through the [Contact](/contact/) page.
