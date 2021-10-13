---
title: Parallel Processing
author: Xiaochi Liu
date: '2021-09-20'
slug: []
categories:
  - R programming
tags:
  - R programming
summary: Use the parallel version of the purrr package - furrr
---

Load the libraries


```r
library(tidyverse)
library(timetk)
library(lubridate)
library(furrr)
library(tictoc)
```


Get the data

* use `select()` to select columns

* use `set_names()` to update the column names


```r
walmart_sales_weekly %>% 
  select(id, Date, Weekly_Sales) %>% 
  set_names(c("id", "date", "value"))  -> sales_data_tbl
sales_data_tbl
```

```
## # A tibble: 1,001 × 3
##    id    date        value
##    <fct> <date>      <dbl>
##  1 1_1   2010-02-05 24924.
##  2 1_1   2010-02-12 46039.
##  3 1_1   2010-02-19 41596.
##  4 1_1   2010-02-26 19404.
##  5 1_1   2010-03-05 21828.
##  6 1_1   2010-03-12 21043.
##  7 1_1   2010-03-19 22137.
##  8 1_1   2010-03-26 26229.
##  9 1_1   2010-04-02 57258.
## 10 1_1   2010-04-09 42961.
## # … with 991 more rows
```

The output is a “tidy” dataset in long format where there are:

* seven `id`s in total. Each `id` represents a walmart store department.


```r
sales_data_tbl %>% 
  count(id)
```

```
## # A tibble: 7 × 2
##   id        n
##   <fct> <int>
## 1 1_1     143
## 2 1_3     143
## 3 1_8     143
## 4 1_13    143
## 5 1_38    143
## 6 1_93    143
## 7 1_95    143
```

* the `date` and `value` combination represents the sales in a given week.





## Purrr

We use a common sequence of operations to iteratively apply an “expensive” modelling function to each `id` (Store Department) that models the sales data as a function of it’s trend and month of the year.

1. use `nest()` to convert the data to a “Nested” structure, where columns contain our data as a list structure.
This generates a column called “data”, with our nested data.


```r
sales_data_tbl %>% 
  nest(data = c(date, value))
```

```
## # A tibble: 7 × 2
##   id    data              
##   <fct> <list>            
## 1 1_1   <tibble [143 × 2]>
## 2 1_3   <tibble [143 × 2]>
## 3 1_8   <tibble [143 × 2]>
## 4 1_13  <tibble [143 × 2]>
## 5 1_38  <tibble [143 × 2]>
## 6 1_93  <tibble [143 × 2]>
## 7 1_95  <tibble [143 × 2]>
```

2. use `mutate()` to add a column called “model”.

3. use `purrr::map()` to iteratively apply an expensive function.
The function applied is a linear regression using the `lm()` function.
We use `Sys.sleep(1)` to simulate an expensive calculation that takes 1-second during each iteration.


```r
sales_data_tbl %>% 
  nest(data = c(date, value)) %>% 
  mutate(model = map(data, function(df) {
    Sys.sleep(1)
    lm(value ~ month(date) + as.numeric(date), data = df)
  }))
```

```
## # A tibble: 7 × 3
##   id    data               model 
##   <fct> <list>             <list>
## 1 1_1   <tibble [143 × 2]> <lm>  
## 2 1_3   <tibble [143 × 2]> <lm>  
## 3 1_8   <tibble [143 × 2]> <lm>  
## 4 1_13  <tibble [143 × 2]> <lm>  
## 5 1_38  <tibble [143 × 2]> <lm>  
## 6 1_93  <tibble [143 × 2]> <lm>  
## 7 1_95  <tibble [143 × 2]> <lm>
```

The output is our nested data now with a column called “model” that contains the 7 Linear Regression Models we just made.
The calculation time is:





## Furrr

Now, we redo the calculation, this time changing `purrr::map()` out for `furrr::future_map()`, which will let us run each calculation in parallel for a speed boost.

1. Add a `plan()`. This allows you to set the number of CPU cores to use when parallel processing.
I have 4-cores available on my computer, so I use all 4.

2, We swap `purrr::map()` out for `furrr::future_map()`, which makes the iterative process run in parallel.

The output is the same nested data structure as previously.
But the time is much shorter.


```r
plan(multisession, workers = 4)

tic()
sales_data_tbl %>% 
  nest(data = c(date, value)) %>% 
  mutate(model = future_map(data, function(df) {
    Sys.sleep(1)
    lm(value ~ month(date) + as.numeric(date), data = df)
  }))
```

```
## # A tibble: 7 × 3
##   id    data               model 
##   <fct> <list>             <list>
## 1 1_1   <tibble [143 × 2]> <lm>  
## 2 1_3   <tibble [143 × 2]> <lm>  
## 3 1_8   <tibble [143 × 2]> <lm>  
## 4 1_13  <tibble [143 × 2]> <lm>  
## 5 1_38  <tibble [143 × 2]> <lm>  
## 6 1_93  <tibble [143 × 2]> <lm>  
## 7 1_95  <tibble [143 × 2]> <lm>
```

```r
toc()
```

```
## 2.809 sec elapsed
```
