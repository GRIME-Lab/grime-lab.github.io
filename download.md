---
layout: single
title: Download
permalink: /download/
---

GRIME AI and SAGE are distributed together. Every option below installs both.

## Conda (recommended)

The Conda package includes every dependency GRIME AI needs, including PyTorch and the SAM 2 model dependencies.

```bash
conda install grimelab::grime-ai
```

Package page: [anaconda.org/GRIMELab](https://anaconda.org/GRIMELab)

## Docker

A ready-to-run image with GRIME AI, GPU-enabled PyTorch, and the SAM 2 model checkpoints already downloaded.

```bash
docker pull ghcr.io/grime-lab/grime-ai:stable
```

The `stable` tag is the latest release. Each release is also tagged with its version number.

## Zenodo

Archived releases with a citable DOI: [doi.org/10.5281/zenodo.22087226](https://doi.org/10.5281/zenodo.22087226)

## Source code

[github.com/GRIME-Lab/GRIME-AI](https://github.com/GRIME-Lab/GRIME-AI), with release notes on the [Releases page](https://github.com/GRIME-Lab/GRIME-AI/releases).

GRIME AI requires Python 3.11 and is released under the Apache 2.0 license.
