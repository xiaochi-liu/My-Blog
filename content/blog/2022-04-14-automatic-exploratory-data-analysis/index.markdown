---
title: Automatic Exploratory Data Analysis
author: Xiaochi Liu
date: '2022-04-14'
slug: []
categories:
  - R programming
tags:
  - Scientific Writing
summary: Create an EDA report in one line of code
---


```r
knitr::opts_chunk$set(warning = FALSE, message = FALSE)
library(DataExplorer)
library(tidyverse)
```

```
## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──
```

```
## ✓ ggplot2 3.3.5          ✓ purrr   0.3.4     
## ✓ tibble  3.1.6          ✓ dplyr   1.0.8     
## ✓ tidyr   1.2.0          ✓ stringr 1.4.0.9000
## ✓ readr   2.1.2          ✓ forcats 0.5.1
```

```
## Warning: package 'tidyr' was built under R version 4.0.5
```

```
## Warning: package 'readr' was built under R version 4.0.5
```

```
## Warning: package 'dplyr' was built under R version 4.0.5
```

```
## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```


```r
gss_cat
```

```
## # A tibble: 21,483 × 9
##     year marital         age race  rincome        partyid    relig denom tvhours
##    <int> <fct>         <int> <fct> <fct>          <fct>      <fct> <fct>   <int>
##  1  2000 Never married    26 White $8000 to 9999  Ind,near … Prot… Sout…      12
##  2  2000 Divorced         48 White $8000 to 9999  Not str r… Prot… Bapt…      NA
##  3  2000 Widowed          67 White Not applicable Independe… Prot… No d…       2
##  4  2000 Never married    39 White Not applicable Ind,near … Orth… Not …       4
##  5  2000 Divorced         25 White Not applicable Not str d… None  Not …       1
##  6  2000 Married          25 White $20000 - 24999 Strong de… Prot… Sout…      NA
##  7  2000 Never married    36 White $25000 or more Not str r… Chri… Not …       3
##  8  2000 Divorced         44 White $7000 to 7999  Ind,near … Prot… Luth…      NA
##  9  2000 Married          44 White $25000 or more Not str d… Prot… Other       0
## 10  2000 Married          47 White $25000 or more Strong re… Prot… Sout…       3
## # … with 21,473 more rows
```

## Create the Automatic EDA Report




## Data Introduction


```r
gss_cat %>% introduce()
```

```
## # A tibble: 1 × 9
##    rows columns discrete_columns continuous_columns all_missing_columns
##   <int>   <int>            <int>              <int>               <int>
## 1 21483       9                6                  3                   0
## # … with 4 more variables: total_missing_values <int>, complete_rows <int>,
## #   total_observations <int>, memory_usage <dbl>
```


```r
gss_cat %>% plot_intro()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

## Missing Values


```r
gss_cat %>% plot_missing()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />


```r
gss_cat %>% profile_missing()
```

```
## # A tibble: 9 × 3
##   feature num_missing pct_missing
##   <fct>         <int>       <dbl>
## 1 year              0     0      
## 2 marital           0     0      
## 3 age              76     0.00354
## 4 race              0     0      
## 5 rincome           0     0      
## 6 partyid           0     0      
## 7 relig             0     0      
## 8 denom             0     0      
## 9 tvhours       10146     0.472
```

## Continuous Features


```r
gss_cat %>% plot_density()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />


```r
gss_cat %>% plot_histogram()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

## Categorical Features


```r
gss_cat %>% plot_bar()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

## Relationships


```r
gss_cat %>% plot_correlation(maxcat = 15)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

