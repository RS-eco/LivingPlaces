Map of lived places
================

## Data

First we specify the exact adresses and the cities

``` r
#' Adresses
adresses <- list(c("Schlüsselbergstraße 8", "81673", "München", "Germany"), 
                 c("Schlossangerweg 10", "85635", "Höhenkirchen-Siegertsbrunn", "Germany"),
                 c("Bachlände 10", "83620", "Feldkirchen-Westerham", "Germany"),
                 c("Y Glyder 007 - Ffriddeodd Site, Ffriddeodd Road", "LL57 2JY", "Bangor", "United Kingdom"),
                 c("Llwyn Onn, Deniol Road", "LL57 2UP", "Bangor", "United Kingdom"),
                 c("Landsdiep 4", "1797 SZ", "'t Horntje (Texel)", "Netherlands"),
                 c("Greenfield Terrace, Hill Street", "LL57 5AY", "Menai Bridge", "United Kingdom"),
                 c("Kuredu Island Resort", "Kuredu", "Lhaviyani Atoll", "Maldives"),
                 c("Königsallee 36", "95448", "Bayreuth", "Germany"),
                 c("Lise-Meitner-Platz 1", "95448", "Bayreuth", "Germany"),
                 c("Meyernreuth 5", "95448", "Bayreuth", "Germ´any"),
                 c("Triftstrasse 4", "06114", "Halle (S.)", "Germany"),
                 c("Dreieichstrasse 12", "60594", "Frankfurt am Main", "Germany"),
                 c("Windthorststr. 15", "06114", "Halle (S.)", "Germany"),
                 c("Kulmbacherstr. 62", "95445", "Bayreuth", "Germany"))

# April 1987 - in München - Schlüsselbergstr. 8, 81673
# Juni 1990 - in Höhenkirchen, Schlossangerweg 10, 85635
# 1. November 1995 in Feldkirchen, Bachlände 10, 83620

# Load world cities
data(world.cities, package="maps")

#' Subset by living places
cities <- c("Munich", "Bangor", "Den Helder", "Bayreuth", "Halle", "Frankfurt am Main", "Halle", "Bayreuth")
cities <- world.cities[world.cities$name %in% c(as.character(cities)),]
```

## Map of lived cities

``` r
#' Get coordinate of addresses
#library(ggmap)
#locations <- lapply(adresses, FUN=function(x) ggmap::geocode(location = toString(x), 
#                                                      output = c("latlon"), 
#                                                      source = c("google")))

# locations_df <- do.call("rbind", locations)
locations_df <- data.frame(lon = c(11.631700, 11.733300, 11.850000, -4.113668, -4.113668,
                                   4.784720, -4.169260, 73.483330, 11.59369, 11.59568,
                                   11.61163, 11.96128, 8.69203, 11.96935, 11.56238), 
                           lat = c(48.12650, 48.01670, 47.90000, 53.22200, 53.22200, 
                                   53.00500, 53.22775,  5.38333, 49.93965, 49.92444,
                                   49.92324, 51.49838, 50.10609, 51.49616, 49.94888),
                           z=c(1:15))

# Load countries data
data(countriesHigh, package="rworldxtra")

#' Plot world map with locations
library(ggplot2)
ggplot() + geom_polygon(data=countriesHigh, aes(x=long, y=lat, group=group), colour="black", fill="gray50") + 
  geom_point(data=locations_df, aes(x=lon, y=lat), colour="red") + 
  coord_sf(xlim=c(-180,180), ylim=c(-90,90))
```

![](figures/unnamed-chunk-2-1.png)<!-- -->

``` r
ggplot() + geom_polygon(data=countriesHigh, aes(x=long, y=lat, group=group), colour="black", fill="gray50") + 
  geom_point(data=locations_df, aes(x=lon, y=lat), colour="red") + 
  coord_sf(xlim=c(-10,80), ylim=c(0,60))
```

![](figures/unnamed-chunk-2-2.png)<!-- -->

``` r
#ggplot() + geom_polygon(data=countriesHigh, aes(x=long, y=lat, group=group), colour="black", fill="gray50") + 
#  geom_point(data=locations_df, aes(x=lon, y=lat, colour=as.factor(z))) + 
#  scale_colour_discrete(name="ID") + coord_sf(xlim=c(-8,16), ylim=c(45,55))
```

Non-ggplot map of lived cities

``` r
#' Non-ggplot Alternative
library(sp)
plot(countriesHigh)
points(x=locations_df$lon, y=locations_df$lat, type="p", col="red", pch=16)
```

![](figures/unnamed-chunk-3-1.png)<!-- -->

## Open Street Map

``` r
#' Get Open Street Map Data
library(Ope´nStreetMap)
tile <- osmtile(x = locations_df$lon[1], y = locations_df$lat[1], zoom=8, type="osm")
map <- openmap(tile$bbox$p1, tile$bbox$p2, zoom=8)
plot(map)

#' Alternatively
library(osmar)
studyarea_bb <- center_bbox(locations_df$lon[1], locations_df$lat[1], 2000, 2000)
OSMdata <- get_osm(studyarea_bb, source = osmsource_api())
plot(OSMdata)
```

## Landsat

``` r
locations_sp <- locations_df
coordinates(locations_sp) <- c("lon","lat")
library(raster)
projection(locations_sp) <- CRS("+init=epsg:4326")

#' Determine path and row for Landsat images
library(wrspathrow)
(pr_sa <- pathrow_num(locations_sp))
(pr_sa_sp <- pathrow_num(locations_sp, as_polys=TRUE))
plot(locations_sp)
plot(pr_sa_sp, add=TRUE)
```

## Sentinel

``` r
#' Get Landsat/Sentinel-2 Images from Earthexplorer (http://earthexplorer.usgs.gov/, user: mb_eco)

#' Read Sentinel-2 JPEG2000 Images using GDAL
#' 1. Check JPEG2000 Driver is installed in GDAL; easy for Linux more challenging for Windows
library(rgdal)
gdalDrivers()

#' 2. Read file into R
filedir <- "/home/matt/Documents/Wissenschaft/Data/Sentinel2"
(dirs <- list.files(path=filedir, pattern=".SAFE", full.names=TRUE))
subdirs <- lapply(dirs, FUN=function(x) paste0(x, "/GRANULE/", list.files(paste0(x, "/GRANULE/"))))
bands <- lapply(subdirs, FUN=function(x) paste0(x, "/IMG_DATA/", list.files(paste0(x, "/IMG_DATA/"))))
# Gives number of lists according to number of scenes, with sublists for all bands!

# Read single band
s2a_b1 <- readGDAL(bands[[1]][1])
s2a_b2 <- readGDAL(bands[[1]][2])

#' Band information
#' Band name    Resolution (m)  Central wavelength (nm) Band width (nm) Purpose
#' B01  60  443     20  Aerosol detection
#' B02  10  490     65  Blue
#' B03  10  560     35  Green
#' B04  10  665     30  Red
#' B05  20  705     15  Vegetation classification
#' B06  20  740     15  Vegetation classification
#' B07  20  783     20  Vegetation classification
#' B08  10  842     115     Near infrared
#' B08A     20  865     20  Vegetation classification
#' B09  60  945     20  Water vapour
#' B10  60  1375    30  Cirrus
#' B11  20  1610    90  Snow / ice / cloud discrimination
#' B12  20  2190    180     Snow / ice / cloud discrimination

# Read RGB stack
library(raster)
s2a_rgb <- stack(bands[[1]][4], bands[[1]][3], bands[[1]][4])

# Plot RGB image
library(RStoolbox)
ggRGB(s2a_rgb, r=3, g=2, b=1, stretch="lin")
plotRGB(s2a_rgb)
```
