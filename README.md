Counting trees outside the forest with image segmentation
==============================

# Overview
![img](references/readme/example.png?raw=true)

[Restoration Mapper](https://restorationmapper.org) is an online tool to create wall-to-wall maps from a Collect Earth Online (CEO) mapathon using open source artificial intelligence and open source satellite imagery. The Restoration Mapper approach enables restoration monitoring stakeholders to:
*  Rapidly assess tree density in non-forested landscapes
*  Establish wall-to-wall baseline data
*  Measure yearly change in tree density without conducting follow-up mapathons
*  Generate maps relevant to land use planning
*  Identify agroforestry, riparian buffer zones, and crop buffer zones
*  Generate GeoTIFFs for further spatial analysis or combination with other datasets

# Methodology

## Model
This model uses a Fully Connected Architecture with:
*  [Convolutional GRU](https://papers.nips.cc/paper/5955-convolutional-lstm-network-a-machine-learning-approach-for-precipitation-nowcasting.pdf) encoder with [layer normalization](https://arxiv.org/abs/1607.06450)
*  [Feature pyramid attention](https://arxiv.org/abs/1805.10180) between encoder and decoder
*  Concurrent spatial and channel squeeze excitation [decoder](https://arxiv.org/abs/1803.02579)
*  [AdaBound](https://arxiv.org/abs/1902.09843) optimizer
*  Binary cross entropy, weighted by effective number of samples, and boundary loss
*  Hypercolumns to facilitate pixel-level accuracy
*  DropBlock and Zoneout for generalization
*  Smoothed image predictions across moving windows
*  Heavy use of skip connections to facilitate smooth loss functions

![img4](references/readme/model.png?raw=true)

## Data
The input images are 24 time series 16x16 Sentinel 2 pixels, interpolated to 10m with DSen2 and corrected for atmospheric deviations, with additional inputs of the slope derived from the Mapzen DEM and Sentinel-1 VV-VH. The specific pre-processing steps are:

*  Download all L1C and L2A imagery for a 16x16 plot
*  Download DEM imagery for a 180x180m region and calculate slope, clipping the border pixels
*  Download Sentinel 1 imagery (VV-VH, gamma backscatter) and fuse to Sentinel 2
*  Super-resolve 20m bands to 10m with DSen2
*  Identify missing and outlier band values and correct by linearly interpolating betwen the nearest two "clean" time steps
*  Calculate cloud cover probability with a 20% threshold, and identify shadows by band thresholding
*  Select the imagery closest to a 15 day window that is clean, linearly interpolating when data is missing
*  Apply 5-pixel median band filter to DEM
*  Apply Whittaker smoothing (lambda = 800) to each time series for each pixel for each band
*  Calculate EVI, BI, MSAVI2

![img3](references/readme/preprocessing-pipeline.png?raw=true)

The current metrics are **95% accuracy, 94% recall** at 10m scale across 1100 plots distributed globally.

The training and testing areas are located below.

![img3](references/readme/train-plots.png?raw=true)
![img4](references/readme/test-plots.png?raw=true)


# Development roadmap

*  Stochastic weight averaging
*  Augmentations: shift, small rotations, mirroring
*  Hyperparameter search
*  Regularization search
*  Self training
*  CRF

# Changelog

### May 01
*  Add final global model and paper replication code

### April 09, 2020
*  Reduce dropblock to 0.85 from 0.75, increase block size from 3 to 4, as per the original paper
*  Reduce zoneout from 0.2 to 0.15 to conform to original paper recommendations
*  Add CSSE block to each conv-bn-relu-drop block instead of after GRU and after FPA. Within FPA, cSSE blocks are after three forward and five forward
*  Remove CSSE block after each time step in convGRU
*  Fix sentinel-1 and sentinel-2 fusion issues


### April 03, 2020
*  Switch Bilinear upsampling in FPA to Nearest Neighbor + Conv-BN
*  Add learning schedule for batch renormalization as implemented in the original paper
*  Switch last SELU to RELU
*  Incorporate drop block into every conv BN block
*  Add docstrings and code formatting to most of the notebooks
*  Finalize notebook naming conventions


# Project Organization
------------

    ├── LICENSE
    ├── Makefile           <- Makefile with commands like `make data` or `make train`
    ├── README.md          <- The top-level README for developers using this project.
    ├── docs               <- A default Sphinx project; see sphinx-doc.org for details
    │
    ├── models             <- Trained and serialized models, model predictions, or model summaries
    │
    ├── notebooks          <- Jupyter notebooks
    │   └── baseline 
    │   └── replicate-paper 
    │   └── visualization 
    │
    ├── references         <- Data dictionaries, manuals, and all other explanatory materials.
    │
    ├── requirements.txt   <- The requirements file for reproducing the analysis environment, e.g.
    │                         generated with `pip freeze > requirements.txt`
    │
    ├── setup.py           <- makes project pip installable (pip install -e .) so src can be imported
    ├── src                <- Source code for use in this project.
    │   ├── __init__.py    <- Makes src a Python module
    │   │
    │   ├── data           <- Scripts to download or generate data
    │   │   └── make_dataset.py
    │   │
    │   ├── features       <- Scripts to turn raw data into features for modeling
    │   │   └── build_features.py
    │   │
    │   ├── models         <- Scripts to train models and then use trained models to make
    │   │   │                 predictions
    │   │   ├── predict_model.py
    │   │   └── train_model.py
    │   │
    │   └── visualization  <- Scripts to create exploratory and results oriented visualizations
    │       └── visualize.py
    │
    └── tox.ini            <- tox file with settings for running tox; see tox.testrun.org


--------
