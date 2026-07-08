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

Download and install R (the programming language):

https://cran.r-project.org/

---

## Step 2. Install RStudio

Download and install RStudio Desktop (the coding environment):

https://posit.co/download/rstudio-desktop/

---

## Step 3. Set Your Working Directory

In RStudio:

```text id="7z39r8"
Session
  └── Set Working Directory
         └── Choose Directory...
```

Select the folder containing your data files.

---

# Phase 2: Installing Packages

Open the R console and run:

```r id="slsbqj"
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

```r id="cgcdw6"
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

The final output will be a heatmap like this:

```text id="a9qgdu"
Rows    → HPV genome positions
Columns → Samples
Colors  → Normalized read depth
```

---

# Input File Format

The input file should look like this:

```text id="nndvr2"
HPV_position,Sample1,Sample2,Sample3
1,550,278,415
2,971,543,761
3,1320,670,1025
4,1538,729,1176
```

* **HPV_position**: HPV genome coordinates.
* **Sample columns**: sequencing coverage at each position.

---

# Understanding the Normalization

We perform **two normalization steps**.

---

# Step 1: Normalize Each Sample

```r id="0hbyyt"
mean_normalized <- function(x) {
  apply(x, 2, function(y) y / mean(y))
}
```

This removes differences in overall sequencing depth between samples.

For example:

```text id="h4q9pi"
Sample A average coverage = 500×
Sample B average coverage = 1000×

After normalization:
Both samples have mean = 1
```

---

# Step 2: Normalize Each HPV Position

```r id="t2fz1f"
new_func2 <- function(x) {
  apply(x, 1, function(y) y / mean(y + 0.001))
}
```

This highlights whether a sample has relatively higher or lower coverage than the other samples at the same HPV position.

The `+0.001` avoids dividing by zero.

---

# Understanding the `apply()` Function

The general syntax is:

```r id="9ywpml"
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

## Example 1: Column Operation

```r id="0n2t2x"
apply(x, 2, mean)
```

R computes:

```text id="w5h6s1"
mean(Sample1)
mean(Sample2)
mean(Sample3)
```

---

## Example 2: Row Operation

```r id="64r4pr"
apply(x, 1, mean)
```

R computes:

```text id="ov8vlh"
mean(Position1)
mean(Position2)
mean(Position3)
```

---

## Breaking Down This Code

```r id="5i6n4r"
apply(x, 2, function(y) y / mean(y))
```

means:

```text id="m7a08z"
Take one column (y)
↓
Compute its mean
↓
Divide every value by that mean
↓
Move to the next column
```

---

```r id="4kpt7q"
apply(x, 1, function(y) y / mean(y + 0.001))
```

means:

```text id="jnknx8"
Take one row (y)
↓
Compute its mean
↓
Divide every value by that mean
↓
Move to the next row
```

---

# Quick Summary

| Code             | Meaning                         |
| ---------------- | ------------------------------- |
| `apply(x,1,FUN)` | Apply a function to each row    |
| `apply(x,2,FUN)` | Apply a function to each column |
| `function(y)`    | Temporary function              |
| `mean(y)`        | Average value                   |

---

# Test Run: Fill in the Blanks

Complete the script below.

```r id="cd8qf5"
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

data <- read.csv("df2.csv", check.names = FALSE)

rownames <- data$__________

data <- data %>%
  select(-__________) %>%
  as.matrix()

rownames(data) <- rownames

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

```r id="b0h0lf"
##change the file name
data <- read.csv("HPV16.csv", check.names = FALSE)

data <- data %>% select(-HPV_position)

mean_normalized <- function(x) {
  apply(x, 2, function(y) y / mean(y))
}

new_func2 <- function(x) {
  apply(x, 1, function(y) y / mean(y + 0.001))
}

df1 <- as.data.frame(mean_normalized(data))

df2 <- as.data.frame(new_func2(df1))

df2 <- cbind(HPV_position = rownames(df2), df2)

rownames <- data$HPV_position

data <- data %>%
  select(-HPV_position) %>%
  as.matrix()

col_fun <- colorRamp2(
  c(0,4),
  c("white", "dark blue")
)

pdf(
  file = "heatmap_HPV16.pdf",
  width = 7.5,
  height = 4
)
```

---

# Interpreting the Heatmap

* Dark blue = relatively higher normalized coverage.
* White = relatively lower normalized coverage.
* Continuous low-coverage regions may indicate possible HPV deletions.
* Similar patterns are grouped together by clustering.
