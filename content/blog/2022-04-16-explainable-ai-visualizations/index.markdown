---
title: Explainable AI Visualizations
author: Xiaochi Liu
date: '2022-12-07'
slug: []
categories:
  - R programming
tags:
  - R programming
summary: Machine learning is great, until it is explained.
---



```r
knitr::opts_chunk$set(
	echo = TRUE,
	message = FALSE,
	warning = FALSE
)
```



```r
library(modelStudio)
library(DALEX)
library(tidyverse)
library(tidymodels)
```




```r
data_tbl <- mpg %>%
    select(hwy, manufacturer:drv, fl, class)
data_tbl
```

```
## # A tibble: 234 × 10
##      hwy manufacturer model      displ  year   cyl trans      drv   fl    class 
##    <int> <chr>        <chr>      <dbl> <int> <int> <chr>      <chr> <chr> <chr> 
##  1    29 audi         a4           1.8  1999     4 auto(l5)   f     p     compa…
##  2    29 audi         a4           1.8  1999     4 manual(m5) f     p     compa…
##  3    31 audi         a4           2    2008     4 manual(m6) f     p     compa…
##  4    30 audi         a4           2    2008     4 auto(av)   f     p     compa…
##  5    26 audi         a4           2.8  1999     6 auto(l5)   f     p     compa…
##  6    26 audi         a4           2.8  1999     6 manual(m5) f     p     compa…
##  7    27 audi         a4           3.1  2008     6 auto(av)   f     p     compa…
##  8    26 audi         a4 quattro   1.8  1999     4 manual(m5) 4     p     compa…
##  9    25 audi         a4 quattro   1.8  1999     4 auto(l5)   4     p     compa…
## 10    28 audi         a4 quattro   2    2008     4 manual(m6) 4     p     compa…
## # … with 224 more rows
```

How Highway Fuel Economy (miles per gallon, `hwy`) can be estimated based on the remaining 9 columns?

## Make a Predictive Model

The best way to understand what affects `hwy` is to build a predictive model (and then explain it).


```r
boost_tree(learn_rate = 0.3) %>%
  set_mode("regression") %>%
  set_engine("xgboost") %>%
  fit(hwy ~ ., data = data_tbl) -> fit_xgboost

fit_xgboost
```

```
## parsnip model object
## 
## ##### xgb.Booster
## raw: 30.1 Kb 
## call:
##   xgboost::xgb.train(params = list(eta = 0.3, max_depth = 6, gamma = 0, 
##     colsample_bytree = 1, colsample_bynode = 1, min_child_weight = 1, 
##     subsample = 1), data = x$data, nrounds = 15, watchlist = x$watchlist, 
##     verbose = 0, nthread = 1, objective = "reg:squarederror")
## params (as set within xgb.train):
##   eta = "0.3", max_depth = "6", gamma = "0", colsample_bytree = "1", colsample_bynode = "1", min_child_weight = "1", subsample = "1", nthread = "1", objective = "reg:squarederror", validate_parameters = "TRUE"
## xgb.attributes:
##   niter
## callbacks:
##   cb.evaluation.log()
## # of features: 81 
## niter: 15
## nfeatures : 81 
## evaluation_log:
##     iter training_rmse
##        1     16.854722
##        2     12.037528
## ---                   
##       14      0.744647
##       15      0.678804
```


## Make an Explainer

With a predictive model in hand, we are ready to create an explainer.


```r
DALEX::explain(
  model = fit_xgboost,
  data  = data_tbl,
  y     = data_tbl$hwy,
  label = "XGBoost"
) -> explainer
```

```
## Preparation of a new explainer is initiated
##   -> model label       :  XGBoost 
##   -> data              :  234  rows  10  cols 
##   -> data              :  tibble converted into a data.frame 
##   -> target variable   :  234  values 
##   -> predict function  :  yhat.model_fit  will be used (  default  )
##   -> predicted values  :  No value for predict function target column. (  default  )
##   -> model_info        :  package parsnip , ver. 1.0.2.9000 , task regression (  default  ) 
##   -> predicted values  :  numerical, min =  11.74135 , mean =  23.28085 , max =  42.29173  
##   -> residual function :  difference between y and yhat (  default  )
##   -> residuals         :  numerical, min =  -1.251106 , mean =  0.1593225 , max =  2.348537  
##   A new explainer has been created!
```

```r
explainer
```

```
## Model label:  XGBoost 
## Model class:  _xgb.Booster,model_fit 
## Data head  :
##   hwy manufacturer model displ year cyl      trans drv fl   class
## 1  29         audi    a4   1.8 1999   4   auto(l5)   f  p compact
## 2  29         audi    a4   1.8 1999   4 manual(m5)   f  p compact
```

## Run modelStudio

The last step is to run `modelStudio`.
This opens up the `modelStudio` app - an interactive tool for exploring predictive models.



### Feature Importance


```r
knitr::include_graphics("importance.png")
```

<img src="importance.png" width="406" />

The feature importance plot is a global representation.
This means that it looks all of your observations and tells you which features (columns that help you predict) have in-general the most predictive value for your model.

1. `displ` - The Engine Displacement (Volume) has the most predictive value in general for this dataset.
It’s an important feature.
In fact, it’s 5X more important than the `model` feature.
And 100X more important than `cyl`.
So I should DEFINITELY investigate it more.


2. `drv` is the second most important feature.
Definitely want to review the Drive Train too.

3. Other features - The Feature importance plot shows the the other features have some importance, but the 80/20 rule tells me to focus on `displ` and `drv`.


### Break Down Plot


```r
knitr::include_graphics("break down.png")
```

<img src="break down.png" width="398" />

The Breakdown plot is a local representation that explains one specific observation.
The plot then shows a intercept (starting value) and the positive or negative contribution that each feature has to developing the prediction.

1. For Observation ID 38, that has an actual `hwy` of 24 Miles Per Gallon (MPG)

2. The starting point (intercept) for all observations is 23.281 MPG.

3. The `displ = 2.4` which boosts the model’s prediction by +3.165 MPG.

4. The `drv = 'f'` which increases the model’s prediction another +1.398 MPG.

5. The `manufacturer = 'dodge'` which decreases the MPG prediction by -1.973

6. And we keep going until we reach the prediction.
Notice that the first features tend to be the most important because they move the prediction the most.


### Shapley Values


```r
knitr::include_graphics("shapley values.png")
```

<img src="shapley values.png" width="391" />

Shapley values are a local representation of the feature importance.
Instead of being global, the shapley values will change by observation telling you again the contribution.

The shapley values are related closely to the Breakdown plot, however you may seem slight differences in the feature contributions.
The order of the shapley plot is always in the most important magnitude contribution.
We also get positive and negative indicating if the feature decreases or increases the prediction.



1. The centerline is again the intercept (23.28 MPG)

2. The `displ` feature is the most important for the observation (ID = 38). The `displ` increases the prediction by 2.715 MPG.

3. We can keep going for the rest of the features.


### Partial Dependence Plot


```r
knitr::include_graphics("partial dependence.png")
```

<img src="partial dependence.png" width="368" />

The partial dependence plot helps us examine one feature at a time.
Above we are only looking at `displ`.
The partial dependence is a global representation, meaning it will not change by observation, but rather helps us see how the model predicts over a range of values for the feature being examined.

We are investigating only `displ`.
As values go from low (smaller engines are 1.6 liter) to high (larger engines are 7 liter), the average prediction for highway fuel becomes lower going from 30-ish Highway MPG to under 20 Highway MPG.
