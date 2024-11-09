# lt-gee-s2-py
Python interface to the Google Earth Engine implementation of the LandTrendr spectral-temporal segmentation algorithm. The LandTrendr spectral-temporal segmentation algorithm is being adapted for use with Sentinel-2

## Introduction

**LandTrendr** is set of spectral-temporal segmentation algorithms that are useful for change detection in a time series of moderate resolution satellite imagery (primarily Landsat) and for generating trajectory-based spectral time series data largely absent of inter-annual signal noise. LT was originally implemented in IDL (Interactive Data Language), but with the help of engineers at Google, it has been ported to the GEE platform.

The **LandTrendr** class from **lt-gee-py** is a light wrapper around the Google Earth Engine API that includes convenience methods to generate images and collections in the format required for LandTrendr on GEE. Please be aware this package is in alpha and may change.

## Getting Started

### Download and Install packages and dependencies

- Install the Python API for Google Earth Engine. This is the only dependency thus far.

```
!pip install earthengine-api
```

- git clone.

```
!git clone https://github.com/vannateck168/lt-gee-s2-py.git
```
- Navigate to the directory.
  
```
%cd /content/lt-gee-s2-py
```
- Install the package.
  
```
!pip install .
```
- Initialize Earth Engine.
  
```
!import geemap
!import ee
```
### Authenticate on Google Earth Engine

- [Earth Engine Authentication and Initialization](https://developers.google.com/earth-engine/guides/auth)

```
earthengine authenticate # There are several alternatives for this. See link above.
```

## Basic Usage

```python
from ltgee import LandTrendr, LtCollection
from ltgee import Sentinel2Composite  # Import Sentinel2Composite directly from ltgee
from datetime import date

# Initialize access to Google's EE servers
ee.Initialize("my_project_name")

# Example arguments to pass to the LandTrendr class. See docstring of LandTrendr for more information'
composite_params = {
    "start_date": date(2018, 1, 1),  # Changed to date object
    "end_date": date(2024, 12, 31),    # Changed to date object
    "area_of_interest": ee.Geometry.Polygon([
        [
            [105.23383507567384, 12.788253594979945],
            [105.99738488036134, 12.788253594979945],
            [105.99738488036134, 13.627858054119361],
            [105.23383507567384, 13.627858054119361],
            [105.23383507567384, 12.788253594979945]
        ]
    ])
}
lt_collection_params = {
        "sr_collection": Sentinel2Composite(**composite_params),
        "index": 'NBR',
        "ftv_list": ['TCB', 'TCG', 'TCW', 'NBR'],
}
lt_params = {
    "lt_collection": LtCollection(**lt_collection_params),
    # "lt_collection": lt_collection_params, # - you may also just pass in your own collection or the params directly
    "run_params": {
            "maxSegments": 6,
            "spikeThreshold": 0.9,
            "vertexCountOvershoot":  3,
            "preventOneYearRecovery":  True,
            "recoveryThreshold":  0.25,
            "pvalThreshold":  .05,
            "bestModelProportion":  0.75,
            "minObservationsNeeded": 6,
        }
}
# Instantiating LandTrendr object. Note: The object will immediately request to run the algorithm on Google's servers.
lt = LandTrendr(**lt_params)

# Access resulting image using the getInfo attribute. LandTrendr is a sublass of ee.Image
print(lt.getInfo())
```

## Features / Methods

- **run**:
    Initiates the LandTrendr algorithm on Google's servers using the specified run_params and generates an image. This is a wrapper around build_sr_collection and build_lt_collection functions. The array image result is saved to LandTrendr.data as an ee.Image.

- **build_sr_collection**:
    Builds an annual cloud and cloud shadow masked medoid composite of Landsat surface reflectance TM-equivalent bands 1,2,3,4,5,7. This collection can be useful outside of use by LandTrendr, but is also the base for creating the input collection for LandTrendr.

- **build_lt_collection**:
    Builds a collection as input to LandTrendr. It will prepare a collection where the first band is the spectral index to base temporal segmentation on, and the subsequent bands will be fitted to segmentation structure of the segmentation index.

- **get_change_map**:
    Generates a set of map layers describing either vegetation loss or gain events with attributes including: year of change detection, spectral delta, duration of change event, pre-change event spectral value, and the rate of spectral change. Each attribute is a band of an ee.Image.

- **get_fitted_data**:
    Generates an annual band stack for a given index provided as ftvList indices to either buildLTcollection or runLT. It flattens the FTV array format to a band per year for a given FTV index.

- **get_segment_data**:
    Generates an array of information about spectral-temporal segments from the breakpoint vertices identified by LandTrendr. Returns either all spectral-temporal segments, or just vegetation loss segments, or just vegetation growth segments.

- **get_segment_count**:
    Given a segment data array produced by the getSegmentData function, this function returns the number of segments identified by LandTrendr as an ee.Image.

- **collection_to_band_stack**:
    Transforms an image collection into an image stack where each band of each image in the collection is concatenated as a band into a single image. Useful for mapping a function over a collection, like transforming surface reflectance to NDVI, and then transforming the resulting collection into a band sequential time series image stack.

- **transform_sr_collection**:
    Transforms the images within an annual surface reflectance collection built by buildSRcollection to a list of provided indices or bands.

- **get_fitted_rgb_col**:
    Creates a collection of RGB visualization images from three FTV bands resulting from a call to LandTrendr segmentation. This is useful for creating thumbnails, filmstrips, and GIFs.

## [Manuscript](http://www.mdpi.com/2072-4292/10/5/691) 

## Citation

>Kennedy, R.E., Yang, Z., Gorelick, N., Braaten, J., Cavalcante, L., Cohen, W.B., Healey, S. (2018). Implementation of the LandTrendr Algorithm on Google Earth Engine. Remote Sensing. 10, 691.

Except as otherwise noted, the content of this repository and accompanying description site (https://emapr.github.io/LT-GEE/) are licensed under the [Creative Commons Attribution 4.0 License](https://creativecommons.org/licenses/by/4.0/), and code samples are licensed under the [Apache 2.0 License](https://www.apache.org/licenses/LICENSE-2.0).
