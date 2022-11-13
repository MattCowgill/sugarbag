
<!-- README.md is generated from this README.Rmd. Please edit this file -->

# sugarbag <img src='man/figures/logo.png' align="right" height="138.5" />

[![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/sugarbag)](https://cran.r-project.org/package=sugarbag)
[![Downloads](http://cranlogs.r-pkg.org/badges/sugarbag?color=brightgreen)](https://cran.r-project.org/package=sugarbag)

The **sugarbag** package creates tessellated hexagon maps for
visualising geo-spatial data. Hexagons of equal size are positioned to
best preserve relationships between individual areas and the closest
focal point, and minimise distance from their actual location. This
method provides an alternative to cartograms that allows all regions to
be compared on the same visual scale.

Maps containing regions with a few small and densely populated areas are
extremely distorted in cartograms. An example of this is a population
cartogram of Australia, which distorts the map into an unrecognisable
shape. The technique implemented in this package is particularly useful
for these regions.

## Installation

You can install the CRAN release version of sugarbag from
[CRAN](https://CRAN.R-project.org) with:

``` r
# install.packages("sugarbag")
```

You can install the development version from GitHub using:

``` r
# install.packages("remotes")
# remotes::install_github("srkobakian/sugarbag")
```

## Getting started

## Creating a hexagon map of Tasmania

Tasmania is the southern-most state of Australia, it has one large land
mass and several smaller islands.

### Data

We will use the Australian Bureau of Statistics’ ESRI shape files to
build our map.

The set has been filtered for only Tasmanian areas. The data set of
Tasmanian Statistical Areas at level two has been provided as a package
data set, `?tas_sa2`.

### Centroids

The function `create_centroids` finds the central points of the polygons
provided as an argument.

``` r
library(sugarbag)
library(dplyr)
library(tidyr)
library(tibble)
library(ggplot2)
library(ggthemes)
# Find the longitude and latitude centroid for each region or area
centroids <- create_centroids(tas_lga, "lga_code_2016")
```

### Hexagon grid

To tessellate correctly, all the hexagons must be evenly spaced. This
function creates a grid of possible locations for the polygons.

``` r
grid <- create_grid(centroids = centroids, hex_size = 0.2, buffer_dist = 1.2)
```

The `sugarbag` package operates by creating a grid of possible hexagons
to allocate electorates. The buffer extends the grid beyond the
geographical space, this is especially useful for densely populated
coastal areas or cities, such as Brisbane and Sydney in this case,
Hobart.

### Allocate areas

Each polygon centroid will be allocated to the closest available hexagon
grid point. The capital cities data set will be used to preserve
neighbourly relationships. The `allocate` function requires two inputs,
the centroids and the grid.

``` r
# Allocate the centroids to the hexagon grid
# We have the same amount of rows, as individual regions
hex_allocated <- allocate(
  centroids = centroids,
  hex_grid = grid,
  hex_size = 0.2, # same size used in create_grid
  hex_filter = 3,
  focal_points = capital_cities,
  width = 30, 
  verbose = TRUE
)
```

The function `fortify_hexagon` assists in plotting. We now have 6 points
per region, one for each point of a hexagon. Connecting these points
will allow actual hexagons to be plotted.

The additional demographic information or data can now be added. This
can be used to allow plots to be coloured by region.

For animations to move between geography and hexagons the `sf_id` must
match, there also needs to be an identifier to separate the states to
animate between for `gganimate`.

``` r
hexagons <- fortify_hexagon(data = hex_allocated, sf_id = "lga_code_2016", hex_size = 0.2)

polygons <- fortify_sfc(tas_lga) %>% 
  mutate(poly_type = "geo")

ggplot(mapping = aes(fill = lga_code_2016)) +
  geom_polygon(data = polygons, 
               aes(x=long, lat, 
    group = interaction(lga_code_2016, polygon)), 
               alpha = 0.4) +
  geom_polygon(data = hexagons, 
               aes(x=long, lat, 
    group = interaction(lga_code_2016))) +
  scale_fill_viridis_d() +
  theme_map() +
  theme(legend.position = "none")
```

<img src="man/figures/README-unnamed-chunk-4-1.png" width="100%" />
