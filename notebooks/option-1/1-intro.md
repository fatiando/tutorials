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

# Processing and gridding gravity data

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

+++ 



The **Bushveld Igneous Complex** is located in South Africa and is the largest known layered igneous intrusion. It has been tilted and eroded forming the outcrops around what appears to be the edge of a great geological basin: the Transvaal Basin. It is approximately 2 billion years old and is divided into four different limbs: the northern, southern, eastern, and western limbs. The Bushveld Complex comprises the Rustenburg Layered suite, the Lebowa Granites and the Rooiberg Felsics, that are overlain by the Karoo sediments. See [Webb et al. (2004)](https://doi.org/10.2113/107.1-2.207) for an overview and previous interpretations of the Bushveld in depth.

```{image} ../../images/bushveld_igneous_complex_geology.jpg
:alt: fishy
:class: bg-primary mb-1
:width: 1000px
:align: center
```
+++