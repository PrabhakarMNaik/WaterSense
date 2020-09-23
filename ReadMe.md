<center>

    
# Studying Water Bodies using Remote Sensing
    
## Introduction
    
</center>

This project aims at using the opensource tools for studying the changes in surface water bodies that have occured with time due to nature and human intervention. The images acquired from the [LandSat 8](https://www.usgs.gov/land-resources/nli/landsat/landsat-8?qt-science_support_page_related_con=0#qt-science_support_page_related_con) satellite are used for the historical information about the water bodies. The tools we are primarily using are [Google Earth Engine](https://earthengine.google.com/) and Python's image processing library [openCV](https://opencv.org/). LandSat-8 carries an Operational Land Imager and a Thermal infrared Sensor that gives us images in [9 spectral bands]() and [2 infrared bands]() respectively. A subset of these bands along with some of their combinations, popularly known as [Normalized differences](https://gisgeography.com/ndvi-normalized-difference-vegetation-index/) will be used for our computations.  

The LandSat-8 images are in GeoTiff format and usually identified with a _Row_ and _Path_ number as well as date on which they were acquired. The _Row_ and _Path_ number for a particular location can be found out in many ways, one particular being the official [USGS](https://www.usgs.gov/) (United States Geological Survey) website. The procedure is explained in this [document](https://lta.cr.usgs.gov/sites/default/files/PerformaSearch_Updated062020.pdf) item 3. Each _Row_ and _Path_ specifies a square shape region of about 185 km x 180 km and around 8000 x 8000 pixels (which varies from scene to scene) size with the resoulution ranging from 30m to about 100m from band to band. The GeoTiff image is an array of pixel intesities, tagged along with geospacial data as well as some metadata pertaining to Loactions, transformations and time stamps. Which is completely reasonable as the Scene cannot treated as a planar surface at these scales. This makes the problem trickier than any other traditional image processing problems.

<center>


### Methodology

</center>

Our algorithm on a higher level finds the water body in the pixel space and gets a bounding region around it. Then this bounding region is projected back in the geodesic coordinates to clip the GeoTiff file in our Region of interest. Then an unsupervised classification algorithm clusters the pixels in the Region of Interest. The algorithm then identify the cluster that represents the water body and report the its area. The detailed algorithm is as below:

Inputs: Path, Row, timeFrame, cloud cover, ReqX, ReqY
Hyperparameters: offset, numClusters, numPixels, varBound
- Set: $imageCollection \leftarrow$ as a set of Scenes for the given (Path, Row, timeFrame, cloud cover). [1](#Aquiring-Images)
- Set: $minCC :=$ Index of Scene with minimum cloud cover $\in imageCollection$ [2](#Finding-the-minCC)
- Set: $image_{minCC} \leftarrow$ ImageCollection\[minCC\] as numpy.ndarray [3](#Converting-$image_{minCC}$-to-np.ndarray)
- Identify: $cnt \leftarrow$ as a set of contours in $image_{minCC}$ [4](#Helper-function:-detecting-Contours,-Rectangles)
- Set: $contour_{required}:= argmin_{i} Distance((reqX, reqY), centre_{contour_i})$ for $contour_i \in cnt$ [4](#Helper-function:-detecting-Contours,-Rectangles)
- identify: $ rect_{required}:=[x, y, width, height] $ for the bounding rectangle of $contour_{required}$ with offset [4](#Helper-function:-detecting-Contours,-Rectangles)
- Project: $roi := rect_{required} $ as an earth engine object.[5](#Reprojecting-to-the-geographic-space)
- Sample: sampleTraining := points $ \in image_{minCC}$ as set. $\|sampleTraining\| = numPixels$ [6](#Getting-Training-Sasmple)
- Train: classifier on the for numClusters over $sampleTraining$ [6](#Getting-Training-Sasmple)
- For each $image \in imageCollection$ :
    - Clip: $image$ to $Roi$
    - Classify: $points \in image$ into numClusters
    - Set: $kNDWI := \{ Ndwi_i\}_{i = 0}^{numClusters}$ [7](#Helper-Function:-Calculating-the-NDWI-for-each-layer)
    - $waterCluster := max_{i}(kNDWI)$
    - Compute: $area_{image} := PixelArea(waterCluster)$.[8](#Helper-Function:-Cluster-Area-calculator)
    - if $(area_{image} - area_{image_{minCC}}) \leq varBound$:
        - $areaHistory \leftarrow area_{image}$
    - End of if
- End of For
- Plot: $AreaHistory$ [9](#Plotting-the-Results)

<center>


### Conventions

</center>

The local variables identifiers are in camel case with lowercase initials $trainingSample$. Functions are in camel case with uppercase initials $ProjectRect$. Global variables are in All capitals. The program is will work in python3.x and will use following packages:
- [numpy](https://numpy.org/)
- [ee](https://developers.google.com/earth-engine/guides/python_install#install-options) 
  For using any method in Earth engine library, an Earth Engine [account](https://earthengine.google.com/) is needed.  
- [IPython.display](https://ipython.org/)
- [matplotlib](https://matplotlib.org/)
- [geemap](https://github.com/giswqs/geemap)
- [cv2](https://docs.opencv.org/master/d0/de3/tutorial_py_intro.html)
- [math](https://docs.python.org/3/library/math.html)
- [urllib](https://docs.python.org/3/library/urllib.html)

<center>

### Conclusion
</center>    
This script will help you bound a region around the water body of users' interest and analyse the Surface Area occupied by that body for any duration user specifies while creating the image collection. Furthermore, the not only water bodies, the above algorithm can also be scaled to study other geographic features like, Vegetation cover using NDVI (Normalized Difference Vegetation Index); Snow cover using NDSI (Normalized Difference Snow Index), etc. 

Such type of analyses make sense in academic and agricultural settings but also create an awareness about how the combined actions of Humans and Nature are affecting our resources. 
