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

+++ {"tags": []}

# Processing and gridding gravity data with Fatiando a Terra

**Authors:**
[Santiago Soler](https://github.com/santisoler),
[Andrea Balza Morales](https://github.com/andieie),
[Agustina Pesce](https://github.com/aguspesce),
[Leonardo Uieda](https://github.com/leouieda)

    
In this tutorial, we will walk through the steps of transforming observed absolute gravity measurements into a grid of residual gravity disturbances at a constant height. On our way there, we will showcase some of the core utilities of many of our open-source libraries:

1. Fetch and cache the data using [Ensaio](https://github.com/fatiando/ensaio) or [Pooch](https://github.com/fatiando/pooch)
1. Calculate the gravity disturbance using [Boule](https://github.com/fatiando/boule)
1. Interpolate data and apply map projections to grids using [Verde](https://github.com/fatiando/verde)
1. Perform topographic correction and equivalent-source interpolation using [Harmonica](https://github.com/fatiando/harmonica)

+++ {"tags": []}

## Video Tutorial

You can watch a video of our tutorial, "Processing gravity data with Harmonica," streamed live on Apr 22, 2021.
```{note}
Remember that the Youtube tutorials may differ slightly from the ones presented on this website, which contains the more recent version of our tutorials
```

```{code-cell} ipython3
:tags: ["remove_input"]

from ipywidgets import widgets
out1 = widgets.Output()
with out1:
  from IPython.display import YouTubeVideo
  video = YouTubeVideo(id="0bxZcCAr6bw", width=854, height=480, fs=1, rel=0)
  display(video)

out = widgets.Tab([out1])
out.set_title(0, 'Youtube')
display(out)
```

## Study area

The **Bushveld Igneous Complex** is located in South Africa and is the largest known layered igneous intrusion. It has been tilted and eroded forming the outcrops around what appears to be the edge of a great geological basin: the Transvaal Basin. It is approximately 2 billion years old and is divided into four different limbs: the northern, southern, eastern, and western limbs. The Bushveld Complex comprises the Rustenburg Layered suite, the Lebowa Granites and the Rooiberg Felsics, that are overlain by the Karoo sediments. See [Webb et al. (2004)](https://doi.org/10.2113/107.1-2.207) for an overview and previous interpretations of the Bushveld in depth.

```{image} ../../images/bushveld_igneous_complex_geology.jpg
:alt: fishy
:class: bg-primary mb-1
:width: 1000px
:align: center
```
+++

## Import packages

Start by loading everything we need.

```{code-cell} ipython3
# The standard Python science stack
import numpy as np
import pandas as pd
import xarray as xr

# For projections (wrapped for Proj)
import pyproj

# Plotting maps using GMT
import pygmt

# The Fatiando stack
import ensaio
import pooch
import verde as vd
import boule as bl
import harmonica as hm
```

## Fetch the data
The three data sets required are:
- Gravity
- Topography 
- Global geoid model 

The global geoid model (co-registered with our topography grid) will be use to convert the topography from orthometric (referenced to "sea level" or the geoid) to geometric (referenced to the WGS84 ellipsoid) heights. This conversion is required to calculate gravity disturbances instead of anomalies (more on that below).

For downloading the data we can use two options: using **Ensaio** or using directly **Pooch**.




+++

### Downloading data using Ensaio
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

### Downloading data using Pooch
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

## Load the data

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

## Crop the data to the Bushveld

The grids are global and the gravity data covers all of Southern Africa. So before we start working we'll need to crop the data to the immediate region around the Bushveld complex. For the topography and geoid grids, we'll use a larger region to avoid edge effects in the topographic correction.

```{code-cell} ipython3
# West, East, South, North (degrees)
region = (25, 32, -27, -23)
# Pad the region for slicing the grids
region_pad = vd.pad_region(region, pad=5)
```

To slice a `pandas.DataFrame`, we can use [Verde](https://www.fatiando.org/verde/)'s `inside` function to get the index of all points falling inside the given region.

```{code-cell} ipython3
data = data[vd.inside((data.longitude, data.latitude), region)]
data
```

For the grids, we use the `sel` method of the `xarray.DataArray` objects directly (one of the many perks of using xarray for gridded data).

```{code-cell} ipython3
geoid = geoid.sel(longitude=slice(*region_pad[:2]), latitude=slice(*region_pad[2:]))
topography = topography.sel(longitude=slice(*region_pad[:2]), latitude=slice(*region_pad[2:]))
topography
```

## Plot the data

Use [PyGMT](https://www.pygmt.org) to plot the data so we can see what we have. PyGMT is particularly well suited for dealing with spatial data, maps, and projections.

```{code-cell} ipython3
fig = pygmt.Figure()
pygmt.makecpt(cmap="viridis", series=[data.gravity_mgal.min(), data.gravity_mgal.max()])
fig.plot(
    x=data.longitude,
    y=data.latitude,
    color=data.gravity_mgal,
    cmap=True,
    style="c4p",
    projection="M15c", 
    frame="afg",
)
fig.colorbar(frame='af+l"observed gravity [mGal]"')
fig.show()
```

Also plot the topography grid with the data points overlaid so we can get a sense for how much topographic variation there is and how much we padded the grids.

```{code-cell} ipython3
:tags: []

fig = pygmt.Figure()
fig.grdimage(topography, projection="M15c", cmap="etopo1", frame=True)
fig.colorbar(frame='af+l"topography [m]"')
fig.coast(shorelines="white", area_thresh=1e6)
fig.plot(
    x=data.longitude,
    y=data.latitude,
    style="c0.04c",
    color="black",
)
fig.show()
```

> **NOTE: This topography grid is not ideal for use with gravity processing.** The resolution of the grid is much coarser than our gravity data, which will cause under-correction when we remove the effect of the topography. For this example, we made the trade-off between a less-than-ideal dataset but one that is of manageable size. In practice, **always try to match the spatial resolution of the data and topography grid**.

+++

## Project the grids and data

Since our study area is relatively small, we can use a map projection to transform our geographic data into a Cartesian coordinate system. This simplifies things like interpolation and topographic correction, which are often much faster if performed in Cartesian rather than spherical or geodetic coordinates.

We'll use [pyproj](https://github.com/pyproj4/pyproj) to create a Mercator projection function with latitude of true scale centered on the data.

```{code-cell} ipython3
projection = pyproj.Proj(proj="merc", lat_ts=data.latitude.mean())
```

We can now use the `projection` to transform our gravity data and include the new coordinates in the `pandas.DataFrame`.

```{code-cell} ipython3
easting, northing = projection(data.longitude.values, data.latitude.values)
data = data.assign(easting_m=easting, northing_m=northing)
data
```

Projecting the grids is more complicated since it involves some interpolation to maintain evenly spaced grid nodes. Verde provides the `project_grid` function that performs all of these calculations automatically for us. We'll use nearest-neighbor interpolation here since it's faster and our project doesn't warp the data too much.

```{code-cell} ipython3
topography_proj = vd.project_grid(topography, projection, method="nearest")
geoid_proj = vd.project_grid(geoid, projection, method="nearest")
topography_proj
```

## Reference heights to the ellipsoid

Before we can calculate gravity disturbances, we need both the observation height and the topography to be referenced to the ellipsoid instead of "sea level" or the geoid. We can do that by adding the geoid height to the current heights.

```{code-cell} ipython3
topography_geometric = topography_proj + geoid_proj
topography_geometric
```

For the observation heights, we have to interpolate the geoid height values at the same points as the observations. We can do this with the cubic interpolation in Verde.

```{code-cell} ipython3
# Unravel the grid so that we can pass it to the interpolator
geoid_table = vd.grid_to_table(geoid_proj)
interpolator = vd.ScipyGridder(method="cubic")
interpolator.fit((geoid_table.easting, geoid_table.northing), geoid_table.geoid)
# Predict the geoid height at same locations as the observation points
data = data.assign(geoid_m=interpolator.predict((data.easting_m, data.northing_m)))
data = data.assign(height_geometric_m=data.height_sea_level_m + data.geoid_m)
data
```

## Gravity disturbance

The gravity disturbance is defined as:

$$
\delta g (\lambda, \phi, h) = g (\lambda, \phi, h) - \gamma (\lambda, \phi, h)
$$

in which $(\lambda, \phi, h)$ are the longitude, latitude, and geometric height, $g$ is the observed gravity acceleration, and $\gamma$ is the calculated **normal gravity** of a reference ellipsoid. 
Notice that all 3 quantities are located at the same point (the observation point).

> **A note on disturbances vs anomalies:** We want to calculate the disturbance $\delta g(\lambda, \phi, h)$ instead of the traditional "gravity anomaly" $\Delta g(\lambda, \phi) = g(\lambda, \phi, h=N) - \gamma(\lambda, \phi, h=0)$ because the anomaly is not exclusively related to the subsurface density anomalies, while the gravity disturbance is. In fact, it does not even make sense to talk about the "height of the gravity anomaly" since by definition it is only variable in longitude and latitude. Most geophysicists will agree that, in practice, the "height" of observations matter for modeling and interpretation. In most cases, even if we calculate gravity anomalies, we end up pretending that they are gravity disturbances when interpreting the data. So why not calculate disturbances directly?

We can use [Boule](https://www.fatiando.org/boule) to compute the **normal gravity** of the WGS84 reference ellipsoid on any point outside of the ellipsoid using a closed-form equation. This eliminates the need for a free-air correction, which is a rough approximation at best. With that, we can compute the gravity disturbance and store it in our `pandas.DataFrame`.

```{code-cell} ipython3
data = data.assign(
    normal_gravity_mgal=bl.WGS84.normal_gravity(data.latitude, data.height_geometric_m)
)
data = data.assign(gravity_disturbance_mgal=data.gravity_mgal - data.normal_gravity_mgal)
data
```

Now plot the gravity disturbance that we just calculated on a map.

```{code-cell} ipython3
fig = pygmt.Figure()
maxabs = vd.maxabs(data.gravity_disturbance_mgal)
pygmt.makecpt(cmap="polar+h", series=[-maxabs, maxabs])
fig.plot(
    x=data.longitude,
    y=data.latitude,
    color=data.gravity_disturbance_mgal,
    cmap=True,
    style="c4p",
    projection="M15c", 
    frame="af",
)
fig.colorbar(frame='af+l"gravity disturbance [mGal]"')
fig.show()
```

## Topographic correction

Now that we have removed normal gravity and calculated a gravity disturbance, the next step is to remove the effect of the topographic and oceanic masses above and below the ellipsoid. We need to do this in order to isolate the effect of the Bushveld rocks.

We will use [Harmonica](https://www.fatiando.org/harmonica) to calculate and remove the gravitational effect of the topography by modeling the masses as a layer of right-rectangular prisms.

The `prism_layer` function creates a prism model based on a surface undulating around a reference level. In our case, the reference is the ellipsoid (0 height) and the surface is our topography (as geometric heights). The difficult part is assigning the correct density (or density contrast) to each part of our model:

* $\rho = 2670\ kg/m^3$ for continental parts above the ellipsoid
* $\rho = -2670\ kg/m^3$ for continental parts below the ellipsoid 
* $\rho = 1040 - 2670\ kg/m^3$ in the oceans (1040 being an approximate density of ocean water)

Here we'll define "continental" as places with positive orthometric height since 0 orthometric height roughly matches the coastlines.

<figure>
<img src="images/topographic-correction.svg" alt="Sketch of the surfaces and masses involved in topographic correction of gravity disturbances.">
<figcaption><em>
    Sketch of the surfaces and masses involved in topographic correction of gravity disturbances. 
    (CC-BY).
</em></figcaption>
</figure>

> **Note:** There is an extra effect of under- and over-correcting for ocean water in the oceans. This happens because we are assuming that the surface of the water is at 0 geometric height (the surface of the ellipsoid) when in fact it roughly follows the geoid. So in oceanic places where the geoid is below the ellipsoid, we correct for water where there was none and vice-versa. Luckily, this effect is negligible here since our study area is relatively far away from the ocean. We can safely ignore this correction but it should be performed when close to the coast or directly above the oceans.

```{code-cell} ipython3
# Start by placing the continental density above and below the ellipsoid
topography_density = np.where(topography_geometric > 0, 2670, -2670)
# Now correct the value in the oceans (topography < 0)
topography_density = np.where(topography_proj < 0, 1040 - 2670, topography_density)
# Create the prism model of the topographic masses
topography_model = hm.prism_layer(
    coordinates=(topography_geometric.easting, topography_geometric.northing),
    surface=topography_geometric,
    reference=0,
    properties={"density": topography_density}
)
topography_model
```

To help us visually what this correction is doing, let's take a look at the density that we assigned across our topographic model.

```{code-cell} ipython3
fig = pygmt.Figure()
scale = vd.maxabs(topography_model.density)
pygmt.makecpt(cmap="roma+h", reverse=True, series=[-scale, scale])
fig.grdimage(topography_model.density, projection="X15c", frame=True)
fig.colorbar(frame='af+l"Density of the prism model [kg/m3]"')
fig.show()
```

As prescribed by the model, the density is 2670 kg/m³ in the continents and 1040 - 2670 kg/m³ in the oceans. The exceptions are parts of the continents that are below 0 geometric height, which have a density of -2670 kg/m³.

+++

Now we can forward model the gravitational effect of the topographic/bathymetric masses at the data points and remove it from our gravity disturbance to calculate a **topography-free (Bouguer) disturbance**.

```{code-cell} ipython3
coordinates = (data.easting_m, data.northing_m, data.height_geometric_m)
data = data.assign(
    terrain_effect_mgal=topography_model.prism_layer.gravity(coordinates, field="g_z"),
)
data = data.assign(
    gravity_bouguer_mgal=data.gravity_disturbance_mgal - data.terrain_effect_mgal
)
data
```

Plot the Bouguer disturbance on a map so we can see what we have.

```{code-cell} ipython3
fig = pygmt.Figure()
maxabs = vd.maxabs(data.gravity_bouguer_mgal)
pygmt.makecpt(cmap="polar", series=[-maxabs, maxabs])
fig.plot(
    x=data.longitude,
    y=data.latitude,
    color=data.gravity_bouguer_mgal,
    cmap=True,
    style="c4p",
    projection="M15c", 
    frame=True,
)
fig.colorbar(frame='af+l"Bouguer disturbance [mGal]"')
fig.show()
```

As expected, the values are mostly negative and are dominated by the Moho signal.

+++

## Regional-residual separation


Bouguer gravity is dominated by the effect of variations in Moho depth because it involves large density contrasts and thicknesses. If we want to see the signal from the shallower intrusions of the Bushveld (often called the **residual**), we'll need to remove the effect of the deeper and larger sources (often called the **regional**).

Traditionally, this is done by fitting a low-degree 2D polynomial to the data and using it as the regional field. We could do this with `verde.Trend` but we'll use a different (and better) approach. The reason for not using a polynomial is that it doesn't take into account the variable height of the observations or the physical nature of the data (a potential field). 

A possible alternative would be to model and remove the Moho effect via an Airy isostasy model, which assumes that all topography is locally compensated by the Moho depth variation. This could be done with `harmonica.isostasy_airy` to calculate the Moho depth, followed by `harmonica.prism_layer` to calculate and remove its gravitational effect.

Here, we will use a simpler approach with the **equivalent sources** present in Harmonica to fit a set of deep point sources to the data. 

<figure>
<img src="images/equivalent-sources.svg" alt="Sketch equivalent sources method to perform interpolation or predict residuals.">
<figcaption><em>
    Sketch equivalent sources method to perform interpolation or predict residuals.
    (CC-BY).
</em></figcaption>
</figure>

Setting a large depth for the sources means that their field will be long-wavelength and is a good way to represent our regional. The choice of depth is somewhat subjective (as is the choice of polynomial degree) and will depend on the interpretation of the results.

> **Note**: The equivalent sources approach may have trouble scaling to large datasets. New features in Harmonica are in development to account for this and should be released with the next version of the package. For the moment, it is recommended to pass the data through a `verde.BlockMean` prior to fitting the deep sources to avoid running out of memory.

```{code-cell} ipython3
# Create a set of deep sources at a depth of 500 km
deep_sources = hm.EquivalentSources(damping=1000, depth=500e3)
# Fit the sources to the data
deep_sources.fit((data.easting_m, data.northing_m, data.height_geometric_m), data.gravity_bouguer_mgal)

# Use the sources to predict the regional field
data = data.assign(
    gravity_regional_mgal=deep_sources.predict((data.easting_m, data.northing_m, data.height_geometric_m))
)
# Calculate a residual field (which is what we want)
data = data.assign(
    gravity_residual_mgal=data.gravity_bouguer_mgal - data.gravity_regional_mgal
)
data
```

Plot a map of the residual field. This should resemble what we know about the geology of the Bushveld.

```{code-cell} ipython3
fig = pygmt.Figure()
scale = vd.maxabs(data.gravity_residual_mgal)
pygmt.makecpt(cmap="polar", series=[-scale, scale])
fig.plot(
    x=data.longitude,
    y=data.latitude,
    color=data.gravity_residual_mgal,
    cmap=True,
    style="c5p",
    projection="M15c", 
    frame=True,
)
fig.colorbar(frame='af+l"residual field [mGal]"')
fig.show()
```

## Grid the residuals

Finally, we can create a regular grid of the residuals at a uniform height. This is a good way to aid visualization and would also allow us to do calculations involving Fourier transforms and other techniques that required gridded data.

The best way to achieve this is, again, the **equivalent sources** in Harmonica. This method allows us to predict the data values on a grid at whatever height we wish (effectively upward-continuing the data), all in a single step. Since our goal here is to accurately model and predict the residual field, we'll use a shallower depth so that the sources can reproduce the short wavelengths in our data.

```{code-cell} ipython3
eqs = hm.EquivalentSources(damping=10, depth=10e3)
eqs.fit((data.easting_m, data.northing_m, data.height_geometric_m), data.gravity_residual_mgal)
```

Now use the equivalent source model to predict data values on a regular grid at a uniform height. In this case, we will use a height that is slightly larger than the highest observation point to avoid downward continuation. As a bonus, we can pass the projection functions that we used earlier to generate a grid in **geographic coordinates** (longitude, latitude) instead of Cartesian.

```{code-cell} ipython3
residual_grid = eqs.grid(
    upward=2200,  # Height in meters
    region=region,
    spacing=2 / 60,  # Decimal degrees
    data_names=["gravity_residual"],
    dims=("latitude", "longitude"),
    projection=projection,
)
residual_grid
```

Plot the gridded residuals on a map and overlay the observation points. It's always important to know where the actual data points are so we can avoid interpreting features in the grid that are likely just an artifact of the data sampling. This is particularly important in areas with sparse and uneven data coverage.

```{code-cell} ipython3
fig = pygmt.Figure()
scale = vd.maxabs(residual_grid.gravity_residual)
pygmt.makecpt(cmap="polar", series=[-scale, scale], no_bg=True)
fig.grdimage(
    residual_grid.gravity_residual,
    shading="+a45+nt0.15",
    projection="M15c",
    frame=True,
)
fig.colorbar(frame='af+l"residual field [mGal]"')
fig.plot(
    x=data.longitude,
    y=data.latitude,
    style="c0.02c",
    color="black",
)
fig.show()
```

## Where to go next

**Congratulations!** You've now completed our gravity processing tutorial. Here are few things you can do now:

* Try using your own data.
* Change the values of certain parameters (topographic density, equivalent source depth, etc) and see what happens.
* Learn more about the individual packages by reading their documentation: https://www.fatiando.org/learn

+++

---

## License

All Python source code is made available under the BSD 3-clause license. You
can freely use and modify the code, without warranty, so long as you provide
attribution to the authors.

Unless otherwise specified, all figures and Jupyter notebooks are available
under the Creative Commons Attribution 4.0 License (CC-BY).

The full text of these licenses is provided in the [`LICENSE.txt`](LICENSE.txt)
file.
