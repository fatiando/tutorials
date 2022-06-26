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


# Import packages

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
+++