# Identifying Solar PV from Aerial Imagery

Blurb:
Locating solar panels is useful for better production forecasts, understanding load on the low-voltage electricity grid, and tracking solar growth across the world. In the UK, despite the Feed In Tariff database, many panel locations are not known. I used computer vision and machine learning techniques combined with overhead aerial imagery to find and locate UK solar panels and you can too.

## Me

My name is Laurence Watson. I'm a data scientist and energy analyst. Currently I am building tools for data scientists at a company I co-founded called Treebeard. We really want to make data science workflows more reliable, and reproducable, so we're trialling a continuous integration tool that automatically creates a container from your python project and builds it for you.

I've worked at several non-profits, like Carbon Tracker, Sandbag etc etc

### UCL Energy Systems and Data Analytics MSc

New masters course at UCL. Heavy use of data and statistics for energy modelling, housed within the Energy Institute.

Who supplied my data:
Ian Holmes, Digimap Support
2TB of imagery data
How many images? 244,417
year 1998 2000 2001 2002 2005 2006 2007 2008 2009 2010 2011 2012 2013 2014 2015 2016 2017
count 13 16 277 2 118 993 2282 1193 6127 12686 6118 5590 23514 37618 73562 17971 56337

## What?

Locating UK solar panels

## Why?

Better forecasts

Note: Inaccuracy of production forecasts for intermittent generators like solar and wind resources mean spinning reserves needed by National Grid. Better forecasts -> less standby generation needed, which is usually more polluting.

[NOWCASTING VIDEO cf Jack Kelly]

Note: 'Nowcasting' or near-realtime forecasting could make use of precise cloud-cover predictions combined with accurate solar PV locations

## Don't we know where the PV is already?

Note: We know where most of it is. Large sites are all in the Renewable Energy Planning Database (~two thirds of capacity). The Feed-in-Tariff database covers ~4GB of capacity - but not all of that is precisely located.
And now the Feed In Tariff is gone, at present there is no centralised method to capture new deployments.
This work is also useful as it could be applied to other countries, or to answer other questions.

[Image of known generation]

## How?

Cunning plan

1. Get image data set of the UK
2. Use Open Street Map labels for solar panels to create a training set of images
3. Use semantic segmentation on all images to find the rest of the panels
4. Write up the results and relax

Note: at the time there were around 15000 labels of solar panels - this would soon change!

[WORKFLOW DIAGRAM]

## Ok, but how? 1

## Digimap products

Aerial Digimap
Licensed by Getmapping plc.
25cm vertical ortho-photography
Note: Data download - was going to be 100's of requests - so got in touch with Digimap and asked for a bulk download. Machine learning with neural networks depends on large datasets.

## OK but how? 2

Enter [RasterVision](https://rastervision.io/)
Quote: "An open source framework for deep learning on satellite and aerial imagery."
Thank you Azavea 👏👏
Note: By Azavea, a USA B-corp who use "Advanced geospatial technology and research for civic and social impact".

## RasterVision

[RasterVision image]
Note: hmmm that workflow looks familiar doesn't it. RasterVision is set up to do one of three tasks - classification of a small image 'chip', object detection - drawing a bounding box, and semantic segmentation, which means labelling each pixel according to which class it is in. I want two classes only, PV or not-PV.

## Setup

Docker image
Must use Linux, due to `nvidia-docker` not supporting other OS'
Dependencies are a pain to manage otherwise
GPUs mandatory - model runs can easily take 12hrs+ depending on experiment

## Experiments

RasterVision takes on a lot of the work that would be reasonably boilerplate between geospatial analysis projects.

import rastervision as rv

##########################################

# Experiment

##########################################
class SolarExperimentSet(rv.ExperimentSet):
def exp_main(self, test=False): # experiment goes here

## Experiment sections - SETUP

[setup code]

## STEPS

SETUP
TASK
BACKEND
DATASET (TRAINING & VALIDATION)
ANALYZE
EXPERIMENT

## In practice:

Start docker container with GPU runtime

> docker run --runtime=nvidia --rm -it -p 6006:6006 \

     -v ${RV_QUICKSTART_CODE_DIR}:/opt/src/code  \
     -v ${RV_QUICKSTART_EXP_DIR}:/opt/data \
     quay.io/azavea/raster-vision:gpu-latest /bin/bash

Set the experiment running

> rastervision run local -p find_the_solar_pv.py

## Nope

[IMAGE OF FAILED TRAINING]

## Challenges

Open Street Map data labels sometimes inconsistent
Images sometimes older than PV deployments

## New cunning plan

Use an existing dataset to train the ML model

Enter the **Distributed Solar Photovoltaic Array Location and Extent Data Set for Remote Sensing Object Identification** [1]

Bradbury, K., Saboo, R., Johnson, T. L., Malof, J. M., Devarajan, A., Zhang, W., Collins, L. M. & Newell, R. G. (2016), ‘Distributed solar photovoltaic array location and extent dataset for remote sensing object identification’, Scientific data 3, 160106.

19,863 PV labels from 4 Californian cities

### PROs

High quality labels, each one annotated multiple times then averaged
Decent size
PV looks the same

### CONs

Image context, building types not similar, potentially leading to poor performance for the UK

## Transfer Learning

Train model on Distributed Solar PV Dataset
Strip final layers off.
Train again on UK good quality labels.

## Or just train on the US set and see

Note: US papers get 76% accuracy (IOU) on their own dataset.

## Results

How do I check the results when I don't have a good UK dataset?

Switch to chip classification.

Why is this ok?

Use FIT database locations to find recent panels.
Use this to create chips that have PV in - labelling less important
Take random urban chips from similar locations & hand inspect to ensure empty

Predict on this new dataset

## OSM

http://osm.gregorywilliams.me.uk/solar/
