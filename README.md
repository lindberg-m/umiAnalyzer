
# umiAnalyzer v0.7.1 (03-11-2019)

Tools for analyzing sequencing data containing UMI generated by umierrorcorrect using comprehensive easy-to-use methods. The package allows merging of multiple samples into a single UMIexperiment object which can be easily manipulated using build-in functions to generate tabular and graphical output.

This readme serves as a basic introduction, for more complete informations see the R vignette using:
```r
browseVignettes("umiAnalyzer")
```

Requirements
------------

- R (>= 3.6.0), which can be downloaded and installed via The Comprehensive R Archive Network [CRAN](https://cran.r-project.org/).
- Installation from R using install_github requires the devtools package
- Using the shiny app requires tidyverse, shiny, shinyFiles, shinyWidgets, 
shinydashboard and DT

Running the visualization app
------------

```r
umiAnalyzer::runUmiVisualiser()
```


Installation from R using devtools
------------

In the R studio console, type:

```r
devtools::install_github("ozimand1as/umiAnalyzer")
```

How to make your own umiExperiment object
---------------------

Define a variable containing the path to the directory with all the umierrorcorrect output folders 
belonging to your experiment. umiAnalyzer comes with raw test data generated with umierrorcorrect that 
you can import if you don't have any of your own.

Call the createUmiExperiment to create your umiExperiment object.


The UMIexperiment object always maintains your raw data, however you can create as many filters as you
like, which will be saved as separate objects to access. You can filter the consensus table of
UMIexperiment object with filterUMIobject. The only mandatory arguments are the object to be filtered
and a user defined name. You can use that name to retrieve a filtered table using getFilter. 

```r
library(umiAnalyzer)

main <- system.file("extdata", package = "umiAnalyzer")

samples <- list.dirs(
  path = main,
  full.names = FALSE,
  recursive = FALSE
)

simsen <- createUmiExperiment(
  experimentName = 'simsen',
  mainDir = main,
  sampleNames = samples
)

reads <- parseBamFiles(
  mainDir = main,
  sampleNames = samples,
  consDepth = 10
)

plotFamilyHistogram(reads)
```

A basic example
-----------------

umiAnalyzer comes with a build-in umiExperiment object to explore, which was generated using the code 
above, so it can be used without creating the it first, if so desired.

In order to call variants using the umiAnalyzer variant caller simply load the package and test data
and use the callVariants function. You can then filter the resulting consensus data (cons.data) within
the object, e.g. for significant variants.

```r
data <- simsen
data <- callVariants(data)

data <- filterVariants(
  object = data,
  p.adjust = 0.2,
  minDepth = 5
)
```

## Importing experimental designs and statistics
### Experimental design
umiAnalyzer supports adding meta data to a UMIsample or UMIexperiment object, such as experimental 
design matrices or clinical parameters. This is done using the importDesign function and requires a 
simple formatted table supplied by the user as a tab separated file. It is important that the order 
of the samples in the meta data file is the same as when building the UMIexperiment object.

```r
data <- simsen
data <- callVariants(data)

metaData <- system.file('extdata', 'metadata.txt', package = 'umiAnalyzer')

data <- importDesign(object = data, file = metaData)
```

## Merging data
### Merge technical replicates for statistics
The mergeTechnicalReplicates function will result in a merged data set accessible from the UMIexperiment object using 
object<at>merged.data. This is meant to provide statistical information across multiple replicates. If you want to merge 
multiple sequencing runs of the sample into a single sample using the collapseReplicates function instead.

```r
mergeTechnicalReplicates(
  object = object,
  filter = 'myfilter'
)
```

### Working with meta data

It is also possible to add meta data to an object and to retrieve metadata if needed. The design 
matrix loaded with importDesign can be retrieved as follows:

```r
design <- getMetaData(
  object = data,
  attributeName = 'design'
)

design
```

Similarly, any kind of meta data can be added and retrieved from an object using addMetaData:

```r
comment <- "fix this"
data <- addMetaData(
  object = data,
  attributeName = 'my-comment',
  attributeValue = comment
)

myattribute <- getMetaData(
  object = data,
  attributeName = 'my-comment'
)

myattribute
```

### Generating VCF output (beta)

Generates a VCF file in the current working directory, another output directory can be speciied using the outDir parameter.
The printAll parameter specifies whether all variants should be printed or only those with at least 5 reads as a support (default = FALSE).

```r
generateVCF(object = exp1, outFile = 'myVCF')
```

### Functions for analyzing [Debarcer](https://github.com/oicr-gsi/debarcer) output

```r
DebarcerExperiment <- createUMIexperiment_Debarcer(
  experiment.name = 'debarcer',
  main.dir = main,
  dir.names = sample.names
)
```
