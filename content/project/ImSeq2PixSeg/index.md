---
title: Image Sequence to Pixel Segmentation for Satellite change detection
summary: ""
tags:
- Deep Learning
- Remote Sensing
- Computer Vision
date: "2016-04-27T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: "Harvested footprint detection.<br />
   Left: processed triplet of images (one being fully cloudy).<br /> 
   Center: pseudo-labeling.<br /> 
   Rigth: predictions"
  focal_point: Smart

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---

## The issue

Analyzing time series of satellite images often presents several challenges. When examining agricultural areas, the focus is typically on tracking changes over time, specifically through optical indices that are generally calculated as the zonal average of farming parcels. This approach requires precise boundary definitions for each parcel. However, many of these optical indices are highly sensitive to atmospheric conditions, especially cloud cover. A parcel that is partially obscured by clouds or even cloud shadows can result in skewed data. While standard convolutional networks applied to individual images can efficiently detect cloud footprints thanks its ability to observe spatial patterns, analyzing the time series to filter out clouds through anomaly detection is more complex. This is because changes such as harvesting, burning, or flooding might also be flagged as anomalies. However, traditional convolutional architectures struggle to properly analyze time series of images, as they are not well-suited for focusing on relative differences between time steps.

As a result, two main approaches are typically used:

- Analyzing structured values obtained through zonal averages after extensive preprocessing. However, this approach is highly sensitive to preprocessing errors, and developing such preprocessing models can be complex and time-consuming, especially if you want to apply convolutional networks to each time step image.
- Connecting the results of a convolutional network to maintain the ability to analyze spatial patterns and easily detect clouds. Unfortunately, most architectures that take this approach end up with weak models because they fail to properly assess evolution between time steps.

## The solution

A solution to this issue is to combine features from both convolutional and recurrent neural networks. This hybrid architecture allows the raw input structure (a two-dimensional grid) to be preserved, making it easier to detect spatial patterns from phenomena, such as cloud cover. At the same time, the model can effectively evaluate changes between time steps, offering greater flexibility and accuracy in time series analysis.

To do so, a network leveraging so-called ConvLSTM cells can be built. These cells are recurrent units that process time steps sequentially. Each LSTM cell has two internal states, often referred to as long-term and short-term memory, which carry information from previous time steps to the next. These states are updated through specific mathematical operations called forget, input, and output gates. For example, the forget gate can evaluate which time steps are relevant. In our case, with time steps being satellite images, an irrelevant one could be a cloudy image.

Here lies the strength of ConvLSTM. Unlike regular LSTMs, which involve multilinear regression to process time steps, ConvLSTMs apply convolution, preserving the 2D grid structure of the image. This allows ConvLSTMs to assign internal states to each pixel based on its local neighbors, enabling the network output to retain the same resolution as the original satellite image sequence.

![Alt text](layer_1_parcel_2.png?raw=true "a title")

The previous image shows the internal hidden states, referred to as short-term memory, of 20 ConvLSTM cells while ingesting a sequence composed of 12 satellite images. As shown here, several cells (such as cells 8, 10, 13, and 18) do not react when ingesting cloudy time steps. Such cells, if returning hidden states and if used as early layers, may propagate the information about the presence of clouds to the lower part of the network.

## The results

![Alt text](144.png?raw=true "a title")

In this experiment, ConvLSTM-based networks were trained to identify the most recently harvested areas in agricultural parcels. The training labels were generated using pseudo-labeling,by applying a heuristic formula with discretization to sections of the images. Parcels were divided into training, validation, and test sets to ensure that none were present in both the test/validation and training sets. As seen in the previous image, the model manages to detect harvested surfaces more accurately and more integrally than the pseudo-labeling used.  It also demonstrates a much stronger ability to recognize harvests under various climatic conditions.

