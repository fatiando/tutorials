---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

```{code-cell} ipython3
:tags: ["remove-cell"]
import numpy as np
import pandas as pd
import xarray as xr
import pyproj
import pygmt
import ensaio
import pooch
import verde as vd
import boule as bl
import harmonica as hm
```
# Fetch the data
The three data sets required are:
- Gravity
- Topography 
- Global geoid model 

The global geoid model (co-registered with our topography grid) will be use to convert the topography from orthometric (referenced to "sea level" or the geoid) to geometric (referenced to the WGS84 ellipsoid) heights. This conversion is required to calculate gravity disturbances instead of anomalies (more on that below).

For downloading the data we can use two options: using **Ensaio** or using directly **Pooch**.
+++

## Downloading data using Ensaio
[Ensaio](https://github.com/fatiando/ensaio) (Portuguese for "rehearsal") is a Python package for downloading open-access sample datasets for Geoscience, specifically from the [Fatiando a Terra Datasets](https://github.com/fatiando-data) collection. To see available datasets with a short description of each, you can visit the [documentation](https://www.fatiando.org/ensaio/latest/gallery/index.html).
Ensaio provides functions for downloading datasets from to your computer. These functions don’t attempt to do any loading of the data into memory and only return the path of the downloaded file on your computer.

```{code-cell} ipython3
path_gravity = ensaio.fetch_southern_africa_gravity(version=1)
path_topography = ensaio.fetch_earth_topography(version=1)
path_geoid  = ensaio.fetch_earth_geoid(version=1)

print(path_gravity)
print(path_topography)
print(path_geoid)
```

Ensaio uses [Pooch](https://www.fatiando.org/pooch) under the hood to manage downloading and caching the data on your computer. Therefore, the other option is to use *Pooch* directly. 

+++

## Downloading data using Pooch
[Pooch](https://www.fatiando.org/pooch) is a more general tool to download data files from anywhere on the web and store them locally for reuse. It is used to manage sample data downloads not only by our own tools but also other popular Scientific Python libraries like [scikit-image](https://github.com/scikit-image/scikit-image), [xarray](https://github.com/pydata/xarray) among others.
To download the data from our [curated sample datasets](https://zenodo.org/record/5167357#.YiJYLOiIaHs) stored on [Zenodo](https://zenodo.org/) we need the Digital Object Identifier (DOI) of the Zenodo data archive:

+++

    doi = "10.5281/zenodo.5167357"
    path_gravity = pooch.retrieve(
        f"doi:{doi}/southern-africa-gravity.csv.xz", 
        known_hash="md5:1dee324a14e647855366d6eb01a1ef35",
    )
    path_topography = pooch.retrieve(
        f"doi:{doi}/earth-topography-10arcmin.nc", 
        known_hash="md5:c43b61322e03669c4313ba3d9a58028d",
    )
    path_geoid = pooch.retrieve(
        f"doi:{doi}/earth-geoid-10arcmin.nc", 
        known_hash="md5:39b97344e704eb68fa381df2eb47da0f",
    )
    print(path_gravity)
    print(path_topography)
    print(path_geoid)

+++

Whether you use Ensaio or Pooch directly, you will have the same datasets, the only difference is in the paths to the location of the files.

+++