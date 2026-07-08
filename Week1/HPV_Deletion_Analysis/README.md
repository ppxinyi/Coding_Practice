# R Programming & Bioinformatics Beginner Guide: Creating an HPV Deletion Heatmap

Welcome to your first coding project! In this tutorial, you will learn how to:

* Read biological data into R
* Normalize sequencing coverage data
* Generate a heatmap
* Interpret the results

No previous programming experience is required!

---

# Phase 1: Environment Setup

## Step 1. Install R

Download and install R:

https://cran.r-project.org/

---

## Step 2. Install RStudio

Download and install RStudio Desktop:

https://posit.co/download/rstudio-desktop/

---

## Step 3. Set Your Working Directory

In RStudio:

```text
Session
  └── Set Working Directory
         └── Choose Directory...
```

Choose the folder containing your data files.

---

# Phase 2: Installing Packages

Open the R console and run:

```r
install.packages(c(
  "dplyr",
  "data.table",
  "gplots",
  "RColorBrewer",
  "tidyverse",
  "viridisLite",
  "circlize"
))

if (!require("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

BiocManager::install("ComplexHeatmap")
```

Then load the packages:

```r
library(dplyr)
library(data.table)
library(ComplexHeatmap)
library(gplots)
library(RColorBrewer)
library(tidyverse)
library(viridisLite)
library(circlize)
```

---

# Project Goal

The goal of this project is to visualize **relative read depth changes** across HPV genomic positions and samples.

The final heatmap will look like this:

```text
Rows    → HPV genome positions
Columns → Samples
Colors  → Normalized read depth
```

---

# Input File Format

The input CSV file should look like this:

```text
HPV_position,Sample1,Sample2,Sample3
1,550,278,415
2,971,543,761
3,1320,670,1025
4,1538,729,1176
```

* `HPV_position` = HPV genome coordinates
* `Sample columns` = sequencing coverage at each position

---

# Understanding the Normalization

We perform two normalization steps.

---

## Step 1: Normalize Each Sample

```r
mean_normalized <- function(x) {
  apply(x, 2, function(y) y / mean(y))
}
```

This removes differences in overall sequencing depth between samples.

For example:

```text
Sample A average coverage = 500×

Sample B average coverage = 1000×

After normalization:

Sample A mean = 1
Sample B mean = 1
```

---

## Step 2: Normalize Each HPV Position

```r
new_func2 <- function(x) {
  apply(x, 1, function(y) y / mean(y + 0.001))
}
```

This highlights whether a sample has relatively higher or lower coverage than the other samples at the same HPV position.

The `0.001` avoids dividing by zero.

---

# Understanding the `apply()` Function

The general syntax is:

```r
apply(X, MARGIN, FUN)
```

where:

* `X` = matrix or data frame
* `MARGIN` = direction

  * `1` → rows
  * `2` → columns
* `FUN` = function to apply

---

## Example Matrix

| Position | Sample1 | Sample2 | Sample3 |
| -------- | ------: | ------: | ------: |
| 1        |     100 |     200 |     300 |
| 2        |     150 |     250 |     350 |
| 3        |      50 |     100 |     150 |

---

### Example 1

```r
apply(x, 2, mean)
```

R computes:

```text
mean(Sample1)
mean(Sample2)
mean(Sample3)
```

---

### Example 2

```r
apply(x, 1, mean)
```

R computes:

```text
mean(Position1)
mean(Position2)
mean(Position3)
```

---

### Breaking Down This Code

```r
apply(x, 2, function(y) y / mean(y))
```

means:

```text
Take one column
↓
Calculate its mean
↓
Divide every value by the mean
↓
Move to the next column
```

---

```r
apply(x, 1, function(y) y / mean(y + 0.001))
```

means:

```text
Take one row
↓
Calculate its mean
↓
Divide every value by the mean
↓
Move to the next row
```

---

# Understanding `rm(list = ls())`

```r
rm(list = ls())
```

This command removes everything from memory and starts with a clean workspace.

```text
rm()  = remove
ls()  = list everything

rm(list = ls())
↓
Delete everything and start fresh.
```

---

# Understanding This Code

```r
data <- data[, -1]
data <- as.matrix(data)
```

### Line 1

```r
data <- data[, -1]
```

Remove the first column (`HPV_position`) because it is not numerical data.

### Line 2

```r
data <- as.matrix(data)
```

Convert the table into a matrix because `Heatmap()` requires a matrix as input.

---

# Test Run: Fill in the Blanks

```r
rm(list = ls())

library(dplyr)
library(data.table)
library(ComplexHeatmap)
library(gplots)
library(RColorBrewer)
library(tidyverse)
library(viridisLite)
library(circlize)

data <- read.csv("__________", check.names = FALSE)

data <- data %>% select(-__________)

mean_normalized <- function(x) {
  apply(x, __, function(y) y / mean(y))
}

new_func2 <- function(x) {
  apply(x, __, function(y) y / mean(y + 0.001))
}

df1 <- as.data.frame(__________(data))

write.csv(df1, file = "df1.csv", row.names = TRUE)

df2 <- as.data.frame(__________(df1))

df2 <- cbind(__________ = rownames(df2), df2)

write.csv(df2, file = "df2.csv", row.names = FALSE)

data <- read.csv("df2.csv")

position <- data$__________

data <- data[, -1]

data <- as.matrix(data)

rownames(data) <- position

col_fun <- colorRamp2(
  c(0,4),
  c("__________", "__________")
)

pdf(
  file = "__________.pdf",
  width = 7.5,
  height = 4
)

Heatmap(
  data,
  name = "Normalized RD",
  cluster_rows = TRUE,
  cluster_columns = FALSE,
  col = col_fun
)

dev.off()
```

---

# Answer Key

```r
data <- read.csv("HPV16.csv", check.names = FALSE)

data <- data %>% select(-HPV_position)

mean_normalized <- function(x) {
  apply(x, 2, function(y) y / mean(y))
}

new_func2 <- function(x) {
  apply(x, 1, function(y) y / mean(y + 0.001))
}

df1 <- as.data.frame(mean_normalized(data))

write.csv(df1, file = "df1.csv", row.names = TRUE)

df2 <- as.data.frame(new_func2(df1))

df2 <- cbind(HPV_position = rownames(df2), df2)

write.csv(df2, file = "df2.csv", row.names = FALSE)

data <- read.csv("df2.csv")

position <- data$HPV_position

data <- data[, -1]

data <- as.matrix(data)

rownames(data) <- position

col_fun <- colorRamp2(
  c(0,4),
  c("white", "dark blue")
)

pdf(
  file = "heatmap_HPV16.pdf",
  width = 7.5,
  height = 4
)

Heatmap(
  data,
  name = "Normalized RD",
  cluster_rows = TRUE,
  cluster_columns = FALSE,
  col = col_fun
)

dev.off()
```

---

# Interpreting the Heatmap

* Dark blue = relatively higher normalized coverage.
* White = relatively lower normalized coverage.
* Continuous low-coverage regions may indicate possible HPV deletions.
* Similar patterns are grouped together by clustering.

