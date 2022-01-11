---
title: Marginal Distributions
author: Xiaochi Liu
date: '2022-01-11'
slug: []
categories:
  - R programming
tags:
  - R programming
summary: Strategies for writing critically
---

**Marginal Distribution (Density) plots** are a way to extend your numeric data with side plots that highlight the density (histogram or boxplots work too).



```r
knitr::opts_chunk$set(warning = FALSE, message = FALSE)
```



```r
library(ggside)
library(tidyverse)
library(tidyquant)
```


```r
mpg
```

```
## # A tibble: 234 × 11
##    manufacturer model      displ  year   cyl trans drv     cty   hwy fl    class
##    <chr>        <chr>      <dbl> <int> <int> <chr> <chr> <int> <int> <chr> <chr>
##  1 audi         a4           1.8  1999     4 auto… f        18    29 p     comp…
##  2 audi         a4           1.8  1999     4 manu… f        21    29 p     comp…
##  3 audi         a4           2    2008     4 manu… f        20    31 p     comp…
##  4 audi         a4           2    2008     4 auto… f        21    30 p     comp…
##  5 audi         a4           2.8  1999     6 auto… f        16    26 p     comp…
##  6 audi         a4           2.8  1999     6 manu… f        18    26 p     comp…
##  7 audi         a4           3.1  2008     6 auto… f        18    27 p     comp…
##  8 audi         a4 quattro   1.8  1999     4 manu… 4        18    26 p     comp…
##  9 audi         a4 quattro   1.8  1999     4 auto… 4        16    25 p     comp…
## 10 audi         a4 quattro   2    2008     4 manu… 4        20    28 p     comp…
## # … with 224 more rows
```

Linear Regression with Marginal Distribution (Density) Side-Plots (Top and Left):


```r
mpg %>% 
  ggplot(aes(hwy, cty, color = class)) +
  geom_point(size = 2, alpha = 0.3) + 
  geom_smooth(aes(color = NULL), se = TRUE) +
  geom_xsidedensity(aes(y = after_stat(density), fill = class),
                    alpha = 0.5,
                    size = 1,
                    position = "stack"
                    ) +
  geom_ysidedensity(aes(x = after_stat(density), fill = class),
                    alpha = 0.5,
                    size = 1,
                    position = "stack"
                    ) + 
  scale_color_tq() +
  scale_fill_tq() +
  theme_tq() +
  labs(title = "Fuel Economy by Vehicle Type",
       subtitle = "ggside density",
       x = "Highway MPG", y = "City MPG") +
  theme(ggside.panel.scale.x = 0.4,
        ggside.panel.scale.y = 0.4)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Facet-Plot with Marginal Box Plots (Top):


```r
mpg %>% 
  ggplot(aes(x = cty, y = hwy, color = class)) +
  geom_point() +
  geom_smooth(aes(color = NULL)) +
  geom_xsideboxplot(alpha = 0.5, size = 1) +
  scale_color_tq() +
  scale_fill_tq() +
  theme_tq() +
  facet_grid(cols = vars(cyl), scales = "free_x") +
  labs(title = "Fuel Economy by Engine Size (Cylinders)") +
  theme(ggside.panel.scale.x = 0.4)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

