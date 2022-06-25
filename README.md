CIPS-Data Analysis Networking Group (DANG)
================
Austin McCoy
5/19/2022

This DANG workshop session will focus on data visualization relevant for
Plant Pathologists using ‘ggplot2’, probably the most popular and widely
used data visualization package in R. This workshop will assume you have
some prior experience in R, particularly for the Maps section.

The figures produced represent a good starting point, but are by no
means the “end all be all” for these types of figures. For example,
these static maps that we produce herein are great for Extension or
Seminars, but with different packages (i.e. Leaflet) you can make
JavaScript interactive and more informative maps for websites, lab
notebooks, lab websites, etc.

------------------------------------------------------------------------

Figure Production in this workshop:

1.  Violin plots

2.  Maps using Arc-GIS data (.shp files)

<!-- -->

1.  State/County Data overlayed onto maps

<!-- -->

3.  Heat maps (i.e. gene expression analysis)

------------------------------------------------------------------------

Some notes:

-   Lines 5,6, and 7 (In the .Rmd file) are setup so that upon knitting
    this .rmd document, a folder containing all figures produced will be
    saved in the directory of this R-Project

-   In each ‘r chunk’ header (‘{r}’) which contain figures, I have
    denoted “dpi=600” so that the figure produced will have a high DPI
    and not appear grainy when knitted. This is a quick, painless, way
    to print figures in high DPI from R markdown. If figures appear
    bunched on the x or y axis, you can also change the length and width
    of your high DPI figures in this same chunk header. Give it a try on
    the Maps section!

-   I have added themes to these figures so that they do not look so
    plain. Feel free to experiment with the theme variables and see what
    they do!

------------------------------------------------------------------------

Install the necessary packages

``` r
#install.packages(tidyverse)
#install.packages(sf)
#install.packages(here)
#install.packages(readxl)
#install.packages(readr)
#install.packages(cowplot)
```

loading in the necessary packages

``` r
library(tidyverse) # loads in ggplot and a few other packages that are very useful for wrangling code/making figures
```

    ## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --

    ## v ggplot2 3.3.5     v purrr   0.3.4
    ## v tibble  3.1.4     v dplyr   1.0.7
    ## v tidyr   1.1.3     v stringr 1.4.0
    ## v readr   2.0.1     v forcats 0.5.1

    ## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library(sf) # "simple format", needed to work with maps/geometry objects
```

    ## Linking to GEOS 3.9.0, GDAL 3.2.1, PROJ 7.2.1

``` r
library(here) # again, quality of life package
```

    ## here() starts at C:/Users/amcco/OneDrive/Desktop/basic-maps-in-R-ARC-GIS-main

``` r
library(readxl) # to read in excel based documents
library(cowplot) # to bring multiple figures into a single, multi-pane, figure
```

------------------------------------------------------------------------

# Violin Plots

Violin plots are great for showing the distribution of your response
variables data points for different treatments. Here I will show how to
produce a violin plot containing individual data points, as well as a
boxplot so we can visually compare treatments.I like to use these to
visualize EC50 data from dose-response testing of oomycetes to
fungicides.

Lets take the fungicide data from [McCoy et al
(2021)](https://github.com/AGmccoy/Phytophthora-sojae-pathotype-distribution-and-fungicide-sensitivity-in-Michigan),
which tested four different oomicide compounds to a population of
*Phytophthora sojae* in Michigan and visualized it via violin plots.

``` r
psojae_fung <- read_excel("TEST.finaldata.set1set2set3.xlsx") # reading in the fungicide data

ggplot(psojae_fung, aes(x=chemistry, y=abs.EC50.estimate, color=chemistry)) +
  geom_violin(scale="width", size=0.1, position = position_dodge(width = 0.5), trim=FALSE) +
  #geom_jitter(shape=16, size=0.01, position = position_jitter(0.1)) + 
  geom_boxplot(width=0.1) + 
  theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
  labs(title="P. sojae fungicide sensitivity testing", x= "Oomicide", y="Absolute EC50 Estimate")
```

    ## Warning: Removed 1 rows containing non-finite values (stat_ydensity).

    ## Warning: Removed 1 rows containing non-finite values (stat_boxplot).

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

------------------------------------------------------------------------

# Maps using .shp (shapefiles) from Arc-GIS

Here we will go over how to use ArC-GIS data to produced layered maps
showing data we choose. In this case, we will make a map of the lower 48
United States showing the known range of *Phytophthora sojae* and
*Phytophthora sansomeana*.

We will then produce a county map of Michigan to show which counties
were sampled in McCoy et al (2021), as well as locations for individual
fields which were sampled when data was provided.

Finally, we will plot “acres planted” by county for four row crops grown
in Michigan: Corn, Wheat, Soy, and Oats.

[Information on Map making in
ggplot2](https://ggplot2-book.org/maps.html)

[USA map
shapefile](https://www.census.gov/cgi-bin/geo/shapefiles/index.php)

US map - *Phytophthora sansomeana* and *P. sojae* range (qualitative
data)

``` r
usa <- here(
  "tl_2021_us_state",
  "tl_2021_us_state.shp"
) %>%
  st_read() # download the shapefiles for US states from the 'USA map shapefile' link above
```

    ## Reading layer `tl_2021_us_state' from data source 
    ##   `C:\Users\amcco\OneDrive\Desktop\basic-maps-in-R-ARC-GIS-main\tl_2021_us_state\tl_2021_us_state.shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 56 features and 14 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -179.2311 ymin: -14.60181 xmax: 179.8597 ymax: 71.43979
    ## Geodetic CRS:  NAD83

``` r
phy_sanso_range <- usa %>%
  filter(NAME %in% c("Oregon", "New York"))

phy_sojae_range <- usa %>%
  filter(NAME %in% c("North Dakota", "South Dakota", "Missouri", "Arkansas", "Mississippi"))

both_range <- usa %>%
  filter(NAME %in% c("Nebraska", "Kansas", "Minnesota", "Iowa", "Wisconsin", "Illinois", "Indiana", "Michigan", "Ohio"))

(usa_48 <- usa %>%
  filter(!(NAME %in% c("Alaska", "District of Columbia", "Hawaii", "Puerto Rico", "United States Virgin Islands", "Commonwealth of the Northern Mariana Islands", "Guam", "American Samoa"))))
```

    ## Simple feature collection with 48 features and 14 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -124.849 ymin: 24.39631 xmax: -66.88544 ymax: 49.38448
    ## Geodetic CRS:  NAD83
    ## First 10 features:
    ##    REGION DIVISION STATEFP  STATENS GEOID STUSPS           NAME LSAD MTFCC
    ## 1       3        5      54 01779805    54     WV  West Virginia   00 G4000
    ## 2       3        5      12 00294478    12     FL        Florida   00 G4000
    ## 3       2        3      17 01779784    17     IL       Illinois   00 G4000
    ## 4       2        4      27 00662849    27     MN      Minnesota   00 G4000
    ## 5       3        5      24 01714934    24     MD       Maryland   00 G4000
    ## 6       1        1      44 01219835    44     RI   Rhode Island   00 G4000
    ## 7       4        8      16 01779783    16     ID          Idaho   00 G4000
    ## 8       1        1      33 01779794    33     NH  New Hampshire   00 G4000
    ## 9       3        5      37 01027616    37     NC North Carolina   00 G4000
    ## 10      1        1      50 01779802    50     VT        Vermont   00 G4000
    ##    FUNCSTAT        ALAND      AWATER    INTPTLAT     INTPTLON
    ## 1         A  62266298634   489204185 +38.6472854 -080.6183274
    ## 2         A 138961722096 45972570361 +28.3989775 -082.5143005
    ## 3         A 143778561906  6216493488 +40.1028754 -089.1526108
    ## 4         A 206232627084 18949394733 +46.3159573 -094.1996043
    ## 5         A  25151992308  6979074857 +38.9466584 -076.6744939
    ## 6         A   2677763359  1323686988 +41.5964850 -071.5264901
    ## 7         A 214049931578  2391569647 +44.3484222 -114.5588538
    ## 8         A  23190115212  1025971768 +43.6726907 -071.5843145
    ## 9         A 125933327733 13456093195 +35.5397100 -079.1308636
    ## 10        A  23872569964  1030754609 +44.0589536 -072.6710173
    ##                          geometry
    ## 1  MULTIPOLYGON (((-80.85847 3...
    ## 2  MULTIPOLYGON (((-83.10874 2...
    ## 3  MULTIPOLYGON (((-89.17208 3...
    ## 4  MULTIPOLYGON (((-92.74568 4...
    ## 5  MULTIPOLYGON (((-75.76659 3...
    ## 6  MULTIPOLYGON (((-71.67881 4...
    ## 7  MULTIPOLYGON (((-111.0455 4...
    ## 8  MULTIPOLYGON (((-71.24548 4...
    ## 9  MULTIPOLYGON (((-76.91598 3...
    ## 10 MULTIPOLYGON (((-72.43462 4...

``` r
sf_use_s2(FALSE)
```

    ## Spherical geometry (s2) switched off

``` r
ggplot() +
  geom_sf(data = usa_48) +
  geom_sf(data = both_range, mapping = aes(fill="red")) +
  geom_sf(data = phy_sanso_range, mapping = aes(fill="black")) +
  geom_sf(data = phy_sojae_range, mapping = aes(fill="white")) +
  coord_sf() +
  scale_fill_identity(name = 'Species Reported', guide = 'legend', labels = c('P.sansomeana', 'Both', 'P.sojae')) +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude")+
      ggtitle("Reported US range of Phytophthora sojae and P. sansomeana")
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

# Plotting data by county

[Michigan
shapefile](https://gis-michigan.opendata.arcgis.com/datasets/67a8ff23b5f54f15b7133b8c30981441/explore?location=44.706068%2C-86.594000%2C6.52)

Michigan Map - presence of *P. sojae* in MI by county from McCoy et al
2021 study with sampling locations as pins.

``` r
MI_map <- here(
  "MI_Counties_(v17a)",
  "Counties_(v17a).shp"
) %>%
  st_read()
```

    ## Reading layer `Counties_(v17a)' from data source 
    ##   `C:\Users\amcco\OneDrive\Desktop\basic-maps-in-R-ARC-GIS-main\MI_Counties_(v17a)\Counties_(v17a).shp' 
    ##   using driver `ESRI Shapefile'
    ## Simple feature collection with 83 features and 15 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -90.41829 ymin: 41.69613 xmax: -82.41348 ymax: 48.26269
    ## Geodetic CRS:  WGS 84

``` r
county_sampled <- MI_map %>%
  filter(NAME %in% c("Allegan", "Antrim", "Barry", "Bay", "Berrien", "Branch", "Cass", "Clare", "Clinton", "Gratiot", "Hillsdale", "Ingham", "Isabella", "Ionia", "Kalamazoo", "Kalkaska", "Kent", "Lapeer", "Lenawee", "Monroe", "Ottawa", "Saginaw", "Sanilac", "Shiawassee", "St. Joseph", "Tuscola", "Van Buren"))

county_positive <- MI_map %>%
  filter(NAME %in% c("Allegan", "Barry", "Berrien", "Branch", "Claire", "Gratiot", "Ionia", "Kalamazoo", "Lenawee", "Ottawa", "Saginaw", "Shiawassee"))

ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data = county_sampled, mapping = aes(fill="white")) +
  geom_sf(data = county_positive, mapping = aes(fill="green")) +
  coord_sf() +
  scale_fill_identity(name = 'Soil baiting', guide = 'legend', labels = c('Phytophthora sojae isolated', 'Phytophthora sojae not isolated')) +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude")+
      ggtitle("McCoy et al 2021 soil sampling")
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

Lets now add the exact location of fields that were sampled to this map

``` r
fields_sampled <- read_excel("gps coordinates for MI plot.xlsx", 
    col_types = c("text", "text", "text", 
        "numeric", "numeric", "text"))

fields_sampled <- na.omit(fields_sampled) # removing missing GPS data points

fields_sf <- st_as_sf(fields_sampled, coords = c("Longitude", "Latitude")) #changing to sf format so the coordinates will be accurately placed

ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data = county_sampled, mapping = aes(fill="white")) +
  geom_sf(data = county_positive, mapping = aes(fill="green")) +
  geom_point(data = fields_sampled, aes(x=Longitude, y=Latitude), shape = 1) +
  coord_sf() +
  scale_fill_identity(name = 'Soil baiting', guide = 'legend', labels = c('Phytophthora sojae isolated', 'Phytophthora sojae not isolated')) +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude")+
      ggtitle("McCoy et al 2021 soil sampling")
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

Lets plot some quantitative data on this Michigan plot now, just to show
how we would do that.

In this case we will use acres planted for corn, wheat, soybean, and
oats data by county in Michigan. This data was retrieved from
[USDA\_NASS](https://www.nass.usda.gov/Statistics_by_State/Michigan/Publications/County_Estimates/index.php).

Lets read in the 2021 data we downloaded from the links above

``` r
soybean <- read_csv("MI 2021 soybean county data.csv")

corn <- read_csv("MI 2021 corn county data.csv")

wheat <- read_csv("MI 2021 wheat county data.csv")

oats <- read_csv("MI 2021 Oat county data.csv")
```

We will make four individual maps, each having data from a single crops
acres planted overlayed on each county

Lets start with soybeans

``` r
MI_map$NAME <- toupper(MI_map$NAME) # changing NAME variable to all uppercase, so we can 'left_join()' this data with USDA-NASS acres planted. To use 'left_join()' the variable must match exactly, even in case.

soybean_map <- left_join(MI_map, soybean, by = c("NAME"="County")) # joining the map data and the acres planted data for soybean

colnames(soybean_map)[35] <- "soybean acres planted" # original name for this column was very long. Here I am changing it so that it is easier to work with/plot.

soybeans_planted <- ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data=soybean_map, mapping = aes(fill=`soybean acres planted`)) +
  coord_sf() +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude") +
      ggtitle("Soybean acres planted by Michigan County")

soybeans_planted
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

Then Corn

``` r
corn_map <- left_join(MI_map, corn, by = c("NAME"="County")) # joining the map data and the acres planted data for corn

colnames(corn_map)[33] <- "corn acres planted" # original name for this column was very long. Here I am changing it so that it is easier to work with/plot.

corn_planted <- ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data=corn_map, mapping = aes(fill=`corn acres planted`)) +
  coord_sf() +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude") +
      ggtitle("Corn acres planted by Michigan County")

corn_planted
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

followed by wheat

``` r
wheat_map <- left_join(MI_map, wheat, by = c("NAME"="County")) # joining the map data and the acres planted data for wheat

colnames(wheat_map)[35] <- "wheat acres planted" # original name for this column was very long. Here I am changing it so that it is easier to work with/plot.

wheat_planted <- ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data=wheat_map, mapping = aes(fill=`wheat acres planted`)) +
  coord_sf() +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude") +
      ggtitle("Wheat acres planted by Michigan County")

wheat_planted
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

and lastly oats

``` r
oats_map <- left_join(MI_map, oats, by = c("NAME"="County")) # joining the map data and the acres planted data for oats

colnames(oats_map)[35] <- "oats acres planted" # original name for this column was very long. Here I am changing it so that it is easier to work with/plot.

oats_planted <- ggplot() +
  geom_sf(data = MI_map) +
  geom_sf(data=oats_map, mapping = aes(fill=`oats acres planted`)) +
  coord_sf() +
    theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
     xlab("Longitude") + ylab("Latitude") +
      ggtitle("Oats acres planted by Michigan County")

oats_planted
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

lets put all of these figures together to give an idea of how many acres
are planted in each Michigan county for these row crops. We will use
‘cowplot’ to put these four figures into a single, compound, figure.

``` r
MI_field_crops <- plot_grid(soybeans_planted,corn_planted,wheat_planted,oats_planted) # 'plot_grid()' allows us to take these saved figures and plot them in a grid at high DPI. Useful if you want to use a figure panel in a manuscript.

MI_field_crops
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

------------------------------------------------------------------------

# Heat maps for gene expression data

Lets visualize *Aspergillus fumigatus* gene expression change between a
fungicide “Compound 089” and “DMSO” treatments. This will be a,
relatively, very small heatmap as there is only comparison between a
single control and a single treatment. With more treatments the columns
of the heatmap will increase.

Having data in “long” format will be needed to plot the gene expression
data with ggpplot.

[data from](https://www.ebi.ac.uk/gxa/experiments/E-MTAB-5309/Results)

``` r
Afumigatus_gene_expression <- read_delim("E-MTAB-5309/E-MTAB-5309-analytics.tsv", 
    delim = "\t", escape_double = FALSE, 
    trim_ws = TRUE)

Afumigatus_gene_expression$comparison <- c("comp089_DMSO") # adding a column to detail how the change in expression was calculated

Afumigatus_gene_expression_rs <- Afumigatus_gene_expression[sample(nrow(Afumigatus_gene_expression), 25), ] # too many genes for this example. so here we are randomly sampling 25 genes and their expression relative to the control "DMSO"

afum_25gene <- ggplot(Afumigatus_gene_expression_rs, aes(x=comparison,y=`Gene ID`, fill=g2_g1.log2foldchange)) +
  geom_tile() +
  theme(axis.text.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.text.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.x = element_text(size = 10, face = "bold", family = "serif"),
    axis.title.y = element_text(size = 10, face = "bold", family = "serif"),
    axis.line.x = element_line(colour = 'gray', size=0.5, linetype='solid'),
    axis.line.y = element_line(colour = 'gray', size=0.5, linetype='solid'),
    legend.text = element_text(size = 10, face = "bold", family = "serif"),
    legend.key = element_blank(),
    legend.title = element_text(size = 10, face="bold", family = "serif"),
    legend.position = "right",
    strip.text.x = element_text(size = 15, face = "bold", family = "serif"),
    title = element_text(size = 10, family = "serif")) +
  xlab("Comparison") + ylab("Gene ID") +
      ggtitle("A. fumigatus gene expression")

afum_25gene
```

![](figure-production-in-R_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

------------------------------------------------------------------------
