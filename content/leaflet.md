---
author: Laura DeCicco
date: 2016-08-23
slug: leaflet
draft: True
title: Using Leaflet and htmlwidgets in a Hugo blog post
categories: Data Science
image: static/leaflet/screenshot.png
tags: 
  - R
  - dataRetrieval
 
---
Get some data. In this example, we are looking for phosphorus measured throughout Wisconsin. Using `dplyr`, we filter the data to sites that have records longer than 15 years, and more than 100 measurements.

``` r
library(dataRetrieval)
pCode <- c("00665")
phos.wi <- readNWISdata(stateCd="WI", parameterCd=pCode,
                     service="site", seriesCatalogOutput=TRUE)

library(dplyr)
phos.wi <- filter(phos.wi, parm_cd %in% pCode) %>%
            filter(count_nu > 50) %>%
            mutate(period = as.Date(end_date) - as.Date(begin_date)) %>%
            filter(period > 15*365)
```

Plot the sites on a map:

``` r
library(leaflet)
leafMap <- leaflet(data=phos.wi) %>% 
  addProviderTiles("CartoDB.Positron") %>%
  addCircleMarkers(~dec_long_va,~dec_lat_va,
                   color = "red", radius=3, stroke=FALSE,
                   fillOpacity = 0.8, opacity = 0.8,
                   popup=~station_nm)
```

The following code could be generally hid from the reader using `echo=FALSE`.

``` r
library(htmlwidgets)
library(htmltools)

currentWD <- getwd()
dir.create("static/leaflet", showWarnings = FALSE)
setwd("static/leaflet")
saveWidget(leafMap, "leafMap.html")
setwd(currentWD)
```

Then, the html that was saved with the `saveWidget` function can be called with the `iframe` html tag. It's not completely obvious to me why the path is `static/leaflet/leafMap/index.html`. If you build the Hugo project, that is where the map ends up.

``` r
<iframe seamless src="/static/leaflet/leafMap/index.html" width="100%" height="500"></iframe>
```

<iframe seamless src="/static/leaflet/leafMap/index.html" width="100%" height="500">
</iframe>
One issue however is that the map itself now indexs to the home page.