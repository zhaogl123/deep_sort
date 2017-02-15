# Deep SORT

## Introduction

This repository contains code for *Simple Online and Realtime Tracking with a Deep Association Metric* (Deep SORT).

## Dependencies

Tracking code:

* NumPy
* sklean (for linear assignment)
* OpenCV (for visualization)

Additionally, feature generation requires:

* TensorFlow
* tfslim

## Installation

First, clone the repository:
```
git clone https://github.com/nwojke/deep_sort.git
```
Then, download pre-generated detections and the CNN checkpoint file from
[here](https://owncloud.uni-koblenz.de/owncloud/s/f9JB0Jr7f3zzqs8).

## Running the tracker

The following example starts the tracker on one of the
[MOT16 benchmark](https://motchallenge.net/data/MOT16/)
sequences.
We assume resources have been extracted to the repository root directory and
the MOT16 benchmark data is in `./MOT16`:
```
python deep_sort_app.py \
    --sequence_dir=./MOT16/test/MOT16-06
    --detection_file=./resources/detections/MOT16_POI_test/MOT16-06.npy \
    --min_confidence=0.3 \
    --nn_budget=100 \
    --display=True
```
Check `python deep_sort_app.py -h` for an overview of available options.
There are also scripts in the repository to visualize results, generate videos,
and evaluate the MOT challenge benchmark.

## Generating detections

Beside the main tracking application, this repository contains a script to
generate features for person re-identification, suitable to compare the visual
appearance of pedestrian bounding boxes using cosine similarity.
The following example generates these features from standard MOT challenge
detections. Again, we assume resources have been extracted to the repository
root directory and MOT16 data is in `./MOT16`:
```
python generate_detections.npy
    --model=resources/networks/mars-small128.ckpt \
    --mot_dir=./MOT16/train \
    --output_dir=./resources/detections/MOT16_train
```
For each sequence of the MOT16 dataset, the output is stored as separate binary
file in NumPy native format. Each file contains an array of shape `Nx138`,
where N is the number of detections in the corresponding MOT sequence.
The first 10 columns of this array contain the raw MOT detection copied over
from the input file. The remaining 128 columns store the appearance descriptor.

## Highlevel overview of source files

In the top-level directory are executable scripts to execute, evaluate, and
visualize the tracker. The main entry point is in `deep_sort_app.py`.
This file runs the tracker on a MOTChallenge sequence.

In package `deep_sort` is the main tracking code:

* `detection.py`: Detection base class.
* `kalman_filter.py`: A Kalman filter implementation and concrete
   parametrization for image space filtering.
* `linear_assignment.py`: This module contains code for min cost matching and
   the matching cascade.
* `iou_matching.py`: This module contains the IOU matching metric.
* `nn_matching.py`: A module for a nearest neighbor matching metric.
* `track.py`: The track class contains single-target track data such as Kalman
  state, number of hits, misses, hit streak, associated feature vectors, etc.
* `tracker.py`: This is the multi-target tracker class.

The `deep_sort_app.py` expects detections in a custom format, stored in .npy
files. These can be computed from MOTChallenge detections using
`generate_detections.py`. We also provide [pre-generated detections](https://owncloud.uni-koblenz.de/owncloud/s/f9JB0Jr7f3zzqs8).

