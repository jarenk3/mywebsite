---
title: "EDA on SG HDB property transactions"
author: "R package build"
date: "2022-12-01"
slug: indexEDA on SG HDB property transactions
categories: R
tags:
- R Markdown
- plot
- regression
---


Attempt to load data in my own internal environment, data can be sourced here
#provide data.gov link


```r
salesfile<-"resale-flat-prices-based-on-registration-date-from-jan-2017-onwards.csv"
hdbresale<-read.csv(file=salesfile,header=TRUE)
summary(hdbresale)
##     month               town            flat_type            block          
##  Length:121475      Length:121475      Length:121475      Length:121475     
##  Class :character   Class :character   Class :character   Class :character  
##  Mode  :character   Mode  :character   Mode  :character   Mode  :character  
##                                                                             
##                                                                             
##                                                                             
##  street_name        storey_range       floor_area_sqm    flat_model       
##  Length:121475      Length:121475      Min.   : 31.00   Length:121475     
##  Class :character   Class :character   1st Qu.: 82.00   Class :character  
##  Mode  :character   Mode  :character   Median : 94.00   Mode  :character  
##                                        Mean   : 97.82                     
##                                        3rd Qu.:113.00                     
##                                        Max.   :249.00                     
##  lease_commence_date remaining_lease     resale_price    
##  Min.   :1966        Length:121475      Min.   : 140000  
##  1st Qu.:1985        Class :character   1st Qu.: 345000  
##  Median :1996        Mode  :character   Median : 430000  
##  Mean   :1995                           Mean   : 462444  
##  3rd Qu.:2005                           3rd Qu.: 545000  
##  Max.   :2019                           Max.   :1360000
```

Attempt to plot a resale price against a floor area per sqm


```r
library(ggplot2)
plot(hdbresale$floor_area_sqm,hdbresale$resale_price)
```

<img src="/post/2022-12-01-eda-on-sg-hdb-property-transactions/indexEDA on SG HDB property transactions_files/figure-html/unnamed-chunk-2-1.png" width="672" />

