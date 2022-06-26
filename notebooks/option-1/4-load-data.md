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

```{code-cell} ipython3
:tags: ["remove-cell"]
path_gravity = ensaio.fetch_southern_africa_gravity(version=1)
path_topography = ensaio.fetch_earth_topography(version=1)
path_geoid  = ensaio.fetch_earth_geoid(version=1)

print(path_gravity)
print(path_topography)
print(path_geoid)
```
# Load the data

Now that the data files have been downloaded and cached to our computer, we can use standard Python libraries to load the data.

Use [Pandas](https://pandas.pydata.org/) to read the gravity data from the compressed CSV file.

```{code-cell} ipython3
data = pd.read_csv(path_gravity)
data
```

And use [xarray](https://xarray.pydata.org/) to read the topography and geoid grids from the netCDF files.

```{code-cell} ipython3
topography = xr.load_dataarray(path_topography)
topography
```

```{code-cell} ipython3
geoid = xr.load_dataarray(path_geoid)
geoid
```

