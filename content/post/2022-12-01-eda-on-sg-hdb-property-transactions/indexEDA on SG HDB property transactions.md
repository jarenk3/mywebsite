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


EDA stands for "Exploratory Data Analysis". It's like exploring a treasure map to find the treasure! In this case, the treasure is information that will help us make predictions with machine learning.

Imagine you're going on a treasure hunt in a big forest. Before you start, you want to look at the map to see where the treasure might be hidden. This is like EDA - we want to look at our data to see what it looks like and what kind of information it has.

When we look at our data, we might see things like numbers or pictures. We can use different tools to help us explore the data, like graphs or charts. We can also ask questions about the data, like "What's the biggest number?" or "How many red dots are there?".

Lets start our project. 
Step 1 - Load data in my own internal environment, data can be sourced here
#https://data.gov.sg/dataset/resale-flat-prices


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

Step 2 - Without cleaning the data, attempt to plot a resale price against a floor area per sqm 


```r
library(ggplot2)
plot(hdbresale$floor_area_sqm,hdbresale$resale_price)
```

<img src="/post/2022-12-01-eda-on-sg-hdb-property-transactions/indexEDA on SG HDB property transactions_files/figure-html/unnamed-chunk-2-1.png" width="672" />

Step 3 - Cleaning the data. Ie check if there is nil values. Convert character string to numeric and etc

```r
library(tidyr)
library(dplyr)
## 
## Attaching package: 'dplyr'
## The following objects are masked from 'package:stats':
## 
##     filter, lag
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
#split column for month into registration year and registration month
hdbresale2<-hdbresale%>%
  separate(month,c("reg_year","reg_month"),"-")
#split remaining_lease to its remaining year and remaining month
hdbresale2<-hdbresale2%>%
  separate(remaining_lease,c("rem_lease_yr","delete1","rem_lease_month","delete2")," ")
## Warning: Expected 4 pieces. Missing pieces filled with `NA` in 9976 rows [6, 16,
## 19, 23, 27, 29, 30, 32, 37, 51, 59, 61, 71, 72, 81, 84, 86, 88, 93, 94, ...].
hdbresale2<-hdbresale2[,c(-12,-14)]

#number of records
nrow(hdbresale2)
## [1] 121475

#checking null values
sum(is.na(hdbresale2)) #only rem_lease_month is with nil values, this is because of nil month
## [1] 9976
sum(is.na(hdbresale2$rem_lease_yr))
## [1] 0

#converting columns to its numeric and factor to apply to machine learning algorithm
hdbresale2 <- hdbresale2 %>%
  mutate(reg_year = as.numeric(reg_year),
         reg_month = as.numeric(reg_month),
         flat_type = as.factor(flat_type),
         flat_model = as.factor(flat_model),
         rem_lease_yr = as.numeric(rem_lease_yr),
         storey_range = as.factor(storey_range),
         town = as.factor(town))
```

Plotting some histogram, charts, ...

```r
#Avg Price vs Flat Type
hdbresale2 %>%
  select(`flat_type`, `resale_price`) %>% 
  group_by(`flat_type`) %>% 
  dplyr::summarize(cases = n(),
                   totalprice = sum(`resale_price`)) %>% 
  mutate(avgprice = totalprice/cases) %>% 
  drop_na() %>% 
  ggplot(aes(x = reorder(`flat_type`, avgprice), y = avgprice)) +
  geom_col() +
  geom_text(aes(label = round(avgprice)), size = 4, hjust = 1) +
  scale_y_continuous(breaks = seq(0, 12e6, 2e6),
                     labels = scales::dollar_format(scale = 1)) +
  labs(title = "A Comparison of Average Resale Prices by flat type.",
       caption = "Source: HDB resale transactions dataset",
       x = "flat type",
       y = "Average HDB resale price") +
  coord_flip()
```

<img src="/post/2022-12-01-eda-on-sg-hdb-property-transactions/indexEDA on SG HDB property transactions_files/figure-html/Plotting average price by HDB model type-1.png" width="672" />

Further plot a average price by town area

```r
hdbresale2 %>%
  select(`town`, `resale_price`) %>% 
  group_by(`town`) %>% 
  dplyr::summarize(
                   medianprice = median(`resale_price`)) %>% 
  drop_na() %>% 
  ggplot(aes(x = reorder(`town`, medianprice), y = medianprice)) +
  geom_col() +
  geom_text(aes(label = round(medianprice)), size = 4, hjust = 1) +
  scale_y_continuous(breaks = seq(0, 12e6, 2e6),
                     labels = scales::dollar_format(scale = 1)) +
  labs(title = "A Comparison of Median Resale Prices by township area.",
       caption = "Source: HDB resale transactions dataset",
       x = "township",
       y = "Median HDB resale price") + 
  coord_flip()
```

<img src="/post/2022-12-01-eda-on-sg-hdb-property-transactions/indexEDA on SG HDB property transactions_files/figure-html/unnamed-chunk-4-1.png" width="672" />

Lets call it a day. See you in next post when i share with you the machine learning algorithm.
