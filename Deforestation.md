# INTRODUCTION 

## Introduction to Remote Sensing
This section is adopted from module 1, section 1 of [Open Nighttime Lights](https://worldbank.github.io/OpenNightLights/tutorials/mod1_1_introduction_to_remote_sensing.html). Please refer to it for more details and a video introduction of remote sensing.

**Remote sensing** is the science of identifying, observing, collecting, and measuring objects without coming into direct contact with them. This can be accomplished through many devices that carry sensors and capture the characteristics of Earth remotely. Sensors on board satellites also record the electromagnetic energy that is reflected or emitted from objects on Earth. There are two types of sensors:

* **Passive sensors** record the natural energy that is (naturally) reflected or emitted from the Earth's surface (e.g. sunlight, moonlight, city lights).
* **Active sensors** provide their own energy source for illumination (e.g. RADAR, LIDAR).

## Introduction to Forest Cover Data
In this exercise, we will primarily work with the [Vegetation Continuous Fields](https://lpdaac.usgs.gov/products/mod44bv006/) (VCF) data provided by the Land Processes Distributed Active Archive Center (LP DAAC), a component of NASA's Earth Observing System Data and Information System (EOSDIS). The MOD44B Version 6 VCF is a yearly representation of surface vegetation from 2000 to 2020 at 250 m resolution. Each pixel stores a percentage of three ground cover components: percent tree cover, percent non-tree cover, and percent bare.

The ground cover percentages are estimates from a machine learning model based on the combination of the Moderate Resolution Imaging Spectroradiometer ([MODIS](https://modis.gsfc.nasa.gov/)) data and other high resolution data from NASA and Google Earth. The machine learning model incorporates the visible bandwidth as well as other bandwidth such as brightness temperature (from MODIS bands 20, 31, 32). 

The VCF data utilize thermal signatures and other correlates to distinguish forest and non-forest plantation, which is an improvement compared to the Normalized Differenced Vegetation Index (NDVI). For this use case, VCF also improves on the Global Forest Cover (GFC) data set, another source used to study deforestation, which only provides binary data points. GFC records baseline forest cover in the year 2000 and includes a binary indicator for the year of deforestation for each 30m × 30m pixel. If over 90% of the forest cover was lost in a pixel by a given year, then the pixel is marked as deforested, while a pixel is marked as reforested if the forest cover went from 0 in 2000 to a positive value. The VCF records continuous changes in percent of ground cover components, which provides more details than the GFC data.   

# INTRODUCTION TO GEOSPATIAL DATA AND TOOLS

## Data Structure
In geospatial data analysis, data can be classified into two categories: raster and vector data. A graphic comparison between raster and vector data can be found in [this](https://worldbank.github.io/OpenNightLights/tutorials/mod2_1_data_overview.html) page.

* **Raster data**: Data stored in a raster format is arranged in a regular grid of cells, without storing the coordinates of each point (namely, a cell, or a pixel). The coordinates of the corner points and the spacing of the grid can be used to calculate (rather than to store) the coordinates of each location in the grid. Any given pixel in the grid stores one or more values (in one or more bands).
* **Vector data**: Data in a vector format is stored such that the X and Y coordinates are stored for each point. Data can be represented, for example, as points, lines and polygons. A point has only one coordinate (X and Y), a line has two coordinates (at the start and end of the line) and a polygon is essentially a line that closes on itself to enclose a region. Polygons are usually used to represent the area and perimeter of continuous geographic features. Vector data stores features in their original resolution, without aggregation.

In this tutorial, we will use both vector data and raster data. Geospatial data in vector format are often stored in a **shapefile** format. Because the structure of points, lines, and polygons are different, each individual shapefile can only contain one vector type (all points, all lines, or all polygons). You will not find a mixture of point, line, and polygon objects in a single shapefile. Raster data, on the other hand, is stored in **Tagged Image File Format (TIFF or TIF)**. A GeoTIFF is a TIFF file that follows a specific standard for structuring meta-data. The meta-data stored in a TIFF is called a tif tag and GeoTIFFs often contain tags including spatial extent, coordinate reference system, resolution, and number of layers. We will see examples in section 2.2. 

More information and examples can be found in sections 3 & 4 of the [Earth Analytics Course](https://www.earthdatascience.org/courses/earth-analytics/). 

## Tools

### Installation of R
To get started with R, we provide instructions on how to download and install R on your computer. R is an open source software, which means users like you can also inspect, modify, and improve its source code.

The Comprehensive R Archive Network ([CRAN](https://cran.r-project.org/)) provides links to install R under different operating systems. RStudio [page](https://support.rstudio.com/hc/en-us/articles/200554786-Problem-Installing-Packages) provides a brief guide for troubleshooting. 

### RStudio
RStudio is an integrated development environment for R. R provides the engine for running code, while RStudio is a user friendly control panel to perform various tasks. RStudio facilitates R code writing and debugging and provides tools for workspace management. RStudio can be downloaded from the RStudio [IDE page](https://www.rstudio.com/products/rstudio/download/). 

There are numerous posts, tutorials, and courses on the internet. Once you have installed R and RStudio, pick any of the following resources to get familiar with R:

* Online courses
  + Datacamp online course: [Introduction to R](https://www.datacamp.com/courses/free-introduction-to-r)
  + Coursera in collaboration with Johns Hopkins University provides an online course on [R programming](https://www.coursera.org/learn/r-programming)

* Books
  + [R Cookbook 2nd Edition](https://rc2e.com/) by James Long and Paul Teetor
  + [R for Data Science](https://r4ds.had.co.nz/) by Hadley Wickham and Garrett Grolemund

### Setting up Environment

In order to perform data manipulation, we need to attach packages. The first step is downloading R packages from CRAN. For this exercise, we are going to use the  package _luna_ to download data from MODIS and the packages _terra_, _tidyverse_, _raster_, and _sf_ for data manipulation. To download these packages, in the R or RStudio console, type the following code (you will need the _remotes_ package in order to download _luna_):
```{r install, eval=FALSE, include=TRUE}
install.packages(c("terra", "remotes", "tidyverse", "raster", "sf"))
remotes::install_github("rspatial/luna")
```
We follow [this](https://rspatial.org/terra/modis/index.html) tutorial to get MODIS data with _luna_. For details of the _terra_ package, please refer to the [package manuscript](https://cran.r-project.org/web/packages/terra/terra.pdf) and [this](https://rspatial.org/terra/pkg/index.html#) tutorial. If you are not familiar with the _tidyverse_ workflow, please refer to the [R for Data Science](https://r4ds.had.co.nz/) book we suggested in the previous section. 

Note: If you receive the message 'These packages have more recent versions available. It is recommended to update all of them. Which would you like to update?', it is best to update all packages. If some of these updates fail (potentially due to a permission issue: 'cannot remove prior installation of package <package>'), you may need to manually update them below.

We now attach these packages.  
```{r, message=FALSE, results='hide'}
library(terra)
library(luna)
library(tidyverse)
library(raster)
library(sf)
```

Troubleshooting Note: If you receive the message "package or namespace load failed", it may be because of incompatible versions for dependent packages. If this is the case, you will see a message like: "namespace ‘dplyr’ 1.0.2 is already loaded, but >= 1.0.4 is required". To proceed, you will need to manually update the package identified, which can be done using `install.packages()` (for example: `install.packages("dplyr")`). Once the update completes, try to re-attach the package which failed (for example: `library(tidyverse)`). Continue repeating this process if you see further errors until you are able to successfully attach all packages. 


### Accessing VCF in R

Once the required packages have been attached, we can access VCF in R. We prefer using R for its ability to download large numbers of files and enable regular, automated updates. For other tools to access _MODIS_ data, see this NASA [website](https://modis.gsfc.nasa.gov/tools/).

We can first use _luna_ to check the list of data products available from _MODIS_. Since _luna_ can also access data from the _LANDSAT_ and _SENTINEL_ platforms, we add `"^MOD|^MYD|^MCD"` to narrow our scope to _MODIS_ data. The printed results below list six products from _MODIS_. 
```{r}
MODIS.product = getProducts("^MOD|^MYD|^MCD")
head(MODIS.product)
```
The product name for VCF is _MOD44B_. We can use the function `productInfo` to launch the information page of VCF. 
```{r, eval=FALSE}
productInfo("MOD44B")
```
We can query _MODIS_ and only download a subset of the data. We need to specify the start and end dates and our area of interest (AOI). The date format is "yyyy-mm-dd". Suppose here we want to subset data from 2010 to 2012.
```{r}
start.date = "2010-01-01"
end.date   = "2012-12-31"
```
In order to subset your area of interest, you need to provide a "map" to `getModis()`. This can be obtained from online databases such as the global administrative area database ([GADM](https://gadm.org/index.html)). You can download map data directly from GADM, as in this [post](https://keithnewman.co.uk/r/maps-in-r-using-gadm.html), or you can use R to obtain GADM map data. We will use R below, which requires first installing the package `geodata`. 
```{r, eval=F}
remotes::install_github("rspatial/geodata")
```

Geographic levels in GADM are defined as:

* level 0: National
* level 1: State/province/equivalent
* level 2: County/district/equivalent
* level 3/4: Smaller administrative levels

For our example, we are interested in India at the district level. We can download the map of India and its level 2 administrative areas with the following code:

```{r, eval=F}
india = geodata::gadm("India", level=2, path="./data")
```
The boundary data is downloaded to the path that you specified in the `path` argument. The downloaded data through `gadm()` will be in the _PackedSpatVector_ class. If you want to convert it to another class (for example, the _sf_ class, which is easier to work with in R), you can first read it using `readRDS()`, then convert to a _SpatVector_ via `vect()` from the _terra_ package, and finally convert it to a _sf_ object.
```{r}
india = readRDS("./data/gadm36_IND_2_pk.rds") %>% vect() %>% st_as_sf(india)
```

The map we downloaded is at the district level (level 2). Assume our AOI is the state of Odisha. Each row of the data represents a county in Odisha, and the geospatial information for each county is stored in the last column: `geometry`. We can filter to obtain the boundaries for our AOI, which will return `aoi` in vector format, stored as a data frame in R.

```{r}
aoi = india %>% filter(NAME_1 == "Odisha")
head(aoi)
ggplot(data = aoi) +
  geom_sf()
```

Now that we have our AOI as well as time frame, we can filter the _MODIS_ VCF data on these values and see what is available.
```{r}
vcf.files = getModis("MOD44B", start.date, end.date, aoi, download = F)
head(vcf.files)
```
The products we are going to download are tiled products. For details of tiled products, the tilling system, and the naming convention, please refer to the _MODIS_ overview page [here](https://lpdaac.usgs.gov/data/get-started-data/collection-overview/missions/modis-overview/#modis-tiling-systems). In essence, we will be downloading grids of maps that cover our AOI.

To actually download these files from the NASA server, you will need a username and password. Please register on [NASA Earth Data](https://urs.earthdata.nasa.gov/) if you haven't done so.

The following code will download the files. Replace the path value with the location on your computer where you would like to store these files. Replace the username and password values with your NASA Earth Data credentials from above. (Example: `getModis("MOD44B", start.date, end.date, aoi, download = T, path = "./data/VCFexample",  username = "myusername", password = "mypwd")`).
```{r, eval=F}
getModis("MOD44B", start.date, end.date, aoi, download = T, path = YourPathHere,  username = YourNASAUserName, password = YourPassWord)
```

The data format from _MODIS_ is HDF and may include sub-datasets. We can use `terra` to read these files and create raster files. For example,
```{r, warning=F}
hdf.example = rast("./data/VCFexample/MOD44B.A2009065.h25v06.006.2017081034537.hdf")
hdf.example
```
We can find basic information such as the coordinate reference system, number of cells, and resolution from the above output. There are `r length(names(hdf.example))` layers in each of the VCF tiled files. We are interested in the percent tree coverage layer. 
```{r}
names(hdf.example)
```
A quick plot of the data can be done with the `plotRBG()` function. We will dive into data processing and basic operations in the next section.
```{r}
plotRGB(hdf.example, stretch = "lin")
```

# BASIC OPERATIONS
In this section, we will follow [this](https://tmieno2.github.io/R-as-GIS-for-Economists/before-you-start-3.html) _R as GIS for Economists_ tutorial to process data that we have downloaded from _MODIS_. 

Note on using raster data in R: _terra_ is a relatively new package, so although it is maintained frequently, many other packages have not yet adopted _terra_ objects. Hence, it may sometimes be necessary to use the _raster_ package to process data. If it is necessary to convert between _terra_  and _raster_ objects, it's important to be aware of the way layers are stored in each object class. _terra_ uses `SpatRaster` objects to store both single and multi-layered data, but _raster_ uses `RasterLayer` objects for single layers and `RasterStack` or `RasterBrick` objects for multiple layers. (For a detailed discussion of these different object classes, see section 4.1 of the _R as GIS for Economists_ source referenced above). In the previous section, we saw that our VCF data has `r length(names(hdf.example))` layers stored in a `SpatRaster` object. If we tried to directly convert `SpatRaster` to `RasterLayer`, it would only convert the first layer by default, so we would need to ensure we selected our layer of interest prior to converting. While we work with _terra_ below, other use cases may require _raster_ and this careful conversion of layers.  

## Merging, cropping and masking

Since there are four hdf files in each year for our AOI, we can first merge the four `SpatRaster` files into one file per year. We'll use 2010 as an example. We can filter to only include our layer of interest - percent of tree cover - from each hdf file, which can be done by subsetting the output using `[[1]]` (using `1` because percent tree cover is the first layer in each file). 

```{r}
# getting file names and directories
vcf.files.2010 = paste0("./data/VCFexample/", vcf.files[grep("A2010065", vcf.files)])

# read hdf files as SpatRaster
vcf.raster.2010 = lapply(vcf.files.2010, function(x) rast(x)[[1]])
```
Before we merge these `SpatRster` objects, it is often a good practice to check their origins and resolutions. `merge` requires origin and resolution to be the same across objects.

```{r}
lapply(vcf.raster.2010, res)
lapply(vcf.raster.2010, origin)
```
We see that origins of these files are slightly different, but all are close to (0, 0). We do not need to worry about these slight differences, as `merge` will handle them automatically.
```{r}
vcf.raster.2010 = do.call(merge, vcf.raster.2010)
plot(vcf.raster.2010)
```

Note: cells with 200% represent water and rivers.

We are now ready to crop and mask the raster file to match our AOI. [This](http://132.72.155.230:3838/r/combining-rasters-and-vector-layers.html) tutorial explains the difference between cropping and masking. 

To crop a raster file according to vector data boundaries (eg, our `aoi` object representing Odisha districts), we first align the coordinate reference systems of our raster file and vector file. Then, use `crop(raster data, vector data)`. To mask, use `mask(raster data, vector data)`. Note that for `terra::mask()`, the second argument needs to be `SpatVector`. _terra_ does not support `sf` objects yet, so we use `vect(aoi)` to convert our sf object `aoi` to a `SpatVector`. 
```{r}
# align coordinate reference systems
aoi = aoi %>% st_transform(crs = crs(vcf.raster.2010))

# crop raster data
vcf.raster.2010.aoi = terra::crop(vcf.raster.2010, aoi)

# mask raster data
vcf.raster.2010.aoi = terra::mask(vcf.raster.2010.aoi, vect(aoi))
```
To plot our new raster file, we use:
```{r}
plot(vcf.raster.2010.aoi)
plot(st_geometry(aoi), add = TRUE)
```

## Extracting values and computing statistics
After we have cropped and masked the raster file to our AOI, we can extract values for each county in the state of Odisha. For more detail on extracting values from raster data points, please refer to section [5.2](https://tmieno2.github.io/R-as-GIS-for-Economists/extracting-values-from-raster-layers-for-vector-data.html) in _R as GIS for Economists_.

```{r}
# extract values for each county
aoi.county.vcf = terra::extract(vcf.raster.2010.aoi, vect(aoi))
colnames(aoi.county.vcf) = c("ID", "Percent Tree Cover")
head(aoi.county.vcf)
```
The values extracted by `terra::extract` are stored in a data frame. Note that the `ID` corresponds to the row number of your vector file (i.e. object `aoi` in our case). We can then compute statistics based on this data frame. Here I compute several statistics describing the percent of forest cover for each county. Note that cells with 200% represent water and river and should be excluded from calculation.
```{r}
aoi.summary = aoi.county.vcf %>% filter(`Percent Tree Cover` <= 100) %>%
  group_by(ID) %>%
  summarise(Mean    = mean(`Percent Tree Cover`),
            Median  = median(`Percent Tree Cover`),
            Max     = max(`Percent Tree Cover`),
            Min     = min(`Percent Tree Cover`),
            `Positive Percent` = sum(`Percent Tree Cover` > 0)/length(`Percent Tree Cover`) * 100)
aoi.summary = aoi.summary %>% mutate(ID = aoi$NAME_2) %>% rename(County = ID)
knitr::kable(aoi.summary, digits = 2)
```
These statistics describe the mean, median, max, min, and percent of positive forest covered land per county.

## Storing and exporting results
With _terra_ you can easily write shape files and several formats of raster files. The main function for writing vector data is `writeVector()`, while for writing raster data we use `writeRaster()`. For details, you can refer to [this](https://rspatial.org/terra/spatial/5-files.html) page and the documentation of _terra_.

# USAGE EXAMPLE

In this section, we will replicate some main results in the [paper](https://academic.oup.com/ej/article-abstract/130/629/1173/5798996?redirectedFrom=fulltext) _The Ecological Impact of Transportation Infrastructure_ by Sam Asher, Teevrat Garg and Paul Novosad (2020). To access the full replication data and code, check [this](https://github.com/devdatalab/paper-agn-forests-roads) github repo. We are going to replicate Table 3 in the paper.

The research question is whether newly constructed rural roads impact local deforestation. The authors explored this question using two empirical strategies: fuzzy RD and difference-in-difference. In the following sections, we implement the difference-in-difference method and replicate the regression results.

## Preparation
### Packages
In order to run fixed effects models, we will need the `fixest` package. If you have not download it, you can type `install.packages("fixest")` in the R or RStudio console. 
```{r, results='hide'}
library(fixest)
```
[This](https://cran.r-project.org/web/packages/fixest/vignettes/fixest_walkthrough.html) tutorial is a good reference for introducing `fixest` functions.

### Data
Data for this exercise was processed and stored in `pmgsy_trees_rural_panel.csv`, which you can find the through the link to the CSV data in the [github repo](https://github.com/devdatalab/paper-agn-forests-roads). Each row of the data frame presents a village in a specific year. 
```{r, results='hide'}
rural.data = data.table::fread("./data/pmgsy_trees_rural_panel.csv")
```

## Replication
The paper estimated the following equation:
$$
Forest_{vdt} = \beta_1\cdot Award_{vdt} + \beta_2\cdot Complete_{vdt} + \alpha_v + \gamma_{dt} + X_v\cdot V_t + \eta_{vdt} 
$$
where $Forest_{vdt}$ is forest cover of village $v$ in district $d$ in year $t$. $Award_{vdt}$ is a dummy variable which takes one during the period when the new road is awarded to the village but has not been built. $Complete_{vdt}$ is also a dummy variable which takes one for all years following the completion of a new road to village $v$. $\alpha_v$ are village fixed effects, while $\gamma_{dt}$ are the district-year fixed effects. $X_v$ controls some baseline characteristics (e.g. forest cover in 2000, total population) and is interacted with year fixed effects $V_t$. For more details regarding the model specification, please refer to section 3.3 of the paper. Note that the paper clustered standard error at the village level. 

There is one more step before we run the regressions. In Stata, which the authors used for their regression, `reghdfe` removed singleton groups automatically. However, the `fixest` package currently doesn't possess this functionality, so for now, we will manually remove these observations.

```{r, results='hide'}
# detect singleton groups: check village fixed effects and district-year fixed effects
index = lapply(list(rural.data$svgroup, rural.data$sdygroup), function(x) x[!(x %in% x[duplicated(x)])])

# how many observations need to be dropped
lapply(index, function(x) length(x))

# exclude singleton groups
rural.data = rural.data %>% filter(!(sdygroup %in% index[[2]]))
```

Finally, we can run our regressions. Following the authors, we test the effect of being awarded a new road and receiving the road on the log forest cover as well as on the average forest cover. 

```{r}
# Table 3
# Column (1)
log.forest.main = feols(ln_forest ~ award_only + treatment_comp|svgroup + sdygroup + year[ln_forest_2000, pc01_pca_tot_p], 
             cluster = "svgroup", 
             data = rural.data)
# Column (2)
log.forest.test = feols(ln_forest ~              treatment_comp|svgroup + sdygroup + year[ln_forest_2000, pc01_pca_tot_p], 
             cluster = "svgroup", 
             data = rural.data)
# Column (3)
avg.forest.main = feols(avg_forest ~ award_only + treatment_comp|svgroup + sdygroup + year[ln_forest_2000, pc01_pca_tot_p], 
             cluster = "svgroup", 
             data = rural.data)
# Column (4)
avg.forest.test = feols(avg_forest ~              treatment_comp|svgroup + sdygroup + year[avg_forest_2000, pc01_pca_tot_p], 
             cluster = "svgroup", 
             data = rural.data)
etable(log.forest.main, log.forest.test, avg.forest.main, avg.forest.test,
       signifCode = c("***"=0.01,"**"=0.05,"*"=0.10),
       drop.section = "slopes")
```

Our results align with the authors' findings presented in Table 3 which show that being awarded a road has a negative impact on forest cover (approximately 0.5% loss in the construction period between being awarded a road and its completion), but after the road is constructed, forest cover appears to return. This could incorrectly be interpreted as a positive effect of roads on tree cover if the award term is left out. This determination that rural roads have no effect on forest loss, in combination with the authors' additional findings of substantial forest loss due to highway construction, have important policy implications for governments considering similar infrastructure expansion. The use of VCF data in this study enabled significant insights, and the potential use cases for VCF data remain numerous.
