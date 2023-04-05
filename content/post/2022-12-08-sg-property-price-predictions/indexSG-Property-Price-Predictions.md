---
title: SG Property Price Predictions
author: R package build
date: '2022-12-08'
slug: 2022 sg property price predictions
categories: R
tags: 
- R Markdown
- random forest
- linear regression
- machine learning
---

Continuing from the previous EDA exercise, with same data object. Let us recall back the data structure in our project.


```r
salesfile<-"resale-flat-prices-based-on-registration-date-from-jan-2017-onwards.csv"
hdbresale<-read.csv(file=salesfile,header=TRUE)
summary(hdbresale)
```

```
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

```r
library(tidyr)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
#split column for month into registration year and registration month
hdbresale2<-hdbresale%>%
  separate(month,c("reg_year","reg_month"),"-")
#split remaining_lease to its remaining year and remaining month
hdbresale2<-hdbresale2%>%
  separate(remaining_lease,c("rem_lease_yr","delete1","rem_lease_month","delete2")," ")
```

```
## Warning: Expected 4 pieces. Missing pieces filled with `NA` in 9976 rows [6, 16,
## 19, 23, 27, 29, 30, 32, 37, 51, 59, 61, 71, 72, 81, 84, 86, 88, 93, 94, ...].
```

```r
hdbresale2<-hdbresale2[,c(-12,-14)]

#number of records
nrow(hdbresale2)
```

```
## [1] 121475
```

```r
#checking null values
sum(is.na(hdbresale2)) #only rem_lease_month is with nil values, this is because of nil month
```

```
## [1] 9976
```

```r
sum(is.na(hdbresale2$rem_lease_yr))
```

```
## [1] 0
```

```r
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



```r
#Display structure of R object. Its indicated that the object is a data frame with 13 variables, and 121,475 rows of transactions.  
str(hdbresale2)
```

```
## 'data.frame':	121475 obs. of  13 variables:
##  $ reg_year           : num  2017 2017 2017 2017 2017 ...
##  $ reg_month          : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ town               : Factor w/ 26 levels "ANG MO KIO","BEDOK",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ flat_type          : Factor w/ 7 levels "1 ROOM","2 ROOM",..: 2 3 3 3 3 3 3 3 3 3 ...
##  $ block              : chr  "406" "108" "602" "465" ...
##  $ street_name        : chr  "ANG MO KIO AVE 10" "ANG MO KIO AVE 4" "ANG MO KIO AVE 5" "ANG MO KIO AVE 10" ...
##  $ storey_range       : Factor w/ 17 levels "01 TO 03","04 TO 06",..: 4 1 1 2 1 1 2 2 2 1 ...
##  $ floor_area_sqm     : num  44 67 67 68 67 68 68 67 68 67 ...
##  $ flat_model         : Factor w/ 20 levels "2-room","Adjoined flat",..: 5 12 12 12 12 12 12 12 12 12 ...
##  $ lease_commence_date: int  1979 1978 1980 1980 1980 1981 1979 1976 1979 1979 ...
##  $ rem_lease_yr       : num  61 60 62 62 62 63 61 58 61 61 ...
##  $ rem_lease_month    : chr  "04" "07" "05" "01" ...
##  $ resale_price       : num  232000 250000 262000 265000 265000 275000 280000 285000 285000 285000 ...
```



```r
#plot correlation of the numeric variables
#ggcorr part of GGally packages
#install.packages("GGally") 
library(GGally) #extended version of ggplot2 library
```

```
## Loading required package: ggplot2
```

```
## Registered S3 method overwritten by 'GGally':
##   method from   
##   +.gg   ggplot2
```

```r
ggcorr(hdbresale2, label = T, hjust = 1, layout.exp = 3)
```

```
## Warning in ggcorr(hdbresale2, label = T, hjust = 1, layout.exp = 3): data
## in column(s) 'town', 'flat_type', 'block', 'street_name', 'storey_range',
## 'flat_model', 'rem_lease_month' are not numeric and were ignored
```

<img src="/post/2022-12-08-sg-property-price-predictions/indexSG-Property-Price-Predictions_files/figure-html/unnamed-chunk-3-1.png" width="672" />

Testing out linear regression model, to show the difference when attempt to overfit the model.


```r
#multivariate linear regression on selected variables only
hdb_linear_model=lm(resale_price~reg_year+floor_area_sqm+rem_lease_yr+flat_model, data=hdbresale2)
summary(hdb_linear_model) #rsquare at 0.5689
```

```
## 
## Call:
## lm(formula = resale_price ~ reg_year + floor_area_sqm + rem_lease_yr + 
##     flat_model, data = hdbresale2)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -336745  -66549  -21961   38003  644422 
## 
## Coefficients:
##                                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)                      -3.250e+07  4.051e+05 -80.211  < 2e-16 ***
## reg_year                          1.599e+04  2.000e+02  79.943  < 2e-16 ***
## floor_area_sqm                    3.503e+03  1.684e+01 207.958  < 2e-16 ***
## rem_lease_yr                      3.233e+03  3.079e+01 104.992  < 2e-16 ***
## flat_modelAdjoined flat           2.178e+05  2.810e+04   7.751 9.20e-15 ***
## flat_modelApartment               1.015e+05  2.723e+04   3.727 0.000194 ***
## flat_modelDBSS                    3.229e+05  2.723e+04  11.858  < 2e-16 ***
## flat_modelImproved                9.516e+04  2.716e+04   3.504 0.000459 ***
## flat_modelImproved-Maisonette     2.269e+05  3.678e+04   6.171 6.81e-10 ***
## flat_modelMaisonette              1.702e+05  2.725e+04   6.247 4.20e-10 ***
## flat_modelModel A                 6.995e+04  2.715e+04   2.577 0.009973 ** 
## flat_modelModel A-Maisonette      2.329e+05  2.809e+04   8.290  < 2e-16 ***
## flat_modelModel A2                7.225e+03  2.728e+04   0.265 0.791151    
## flat_modelMulti Generation        2.352e+05  3.026e+04   7.773 7.70e-15 ***
## flat_modelNew Generation          8.403e+04  2.716e+04   3.094 0.001978 ** 
## flat_modelPremium Apartment       6.790e+04  2.716e+04   2.500 0.012427 *  
## flat_modelPremium Apartment Loft  4.370e+05  2.964e+04  14.743  < 2e-16 ***
## flat_modelPremium Maisonette      1.676e+05  3.986e+04   4.204 2.62e-05 ***
## flat_modelSimplified              7.510e+04  2.719e+04   2.762 0.005739 ** 
## flat_modelStandard                1.638e+05  2.722e+04   6.018 1.78e-09 ***
## flat_modelTerrace                 4.879e+05  2.980e+04  16.372  < 2e-16 ***
## flat_modelType S1                 5.457e+05  2.805e+04  19.457  < 2e-16 ***
## flat_modelType S2                 6.183e+05  2.867e+04  21.568  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 105100 on 121452 degrees of freedom
## Multiple R-squared:  0.5689,	Adjusted R-squared:  0.5689 
## F-statistic:  7286 on 22 and 121452 DF,  p-value: < 2.2e-16
```

```r
#multivariate linear regression on all variables
#hdb_linear_model2<-lm(resale_price~.,data=hdbresale2) #  What is considered optimum? Rsquare at 0.9358, 
#summary(hdb_linear_model2)
```

#From doing a split piece. I learnt that when rendering the Rmd file, i cannot allow for a "Install.packages()" code. And also need to source again the raw csv data file. which is why i recopied the earlier few steps from my earlier EDA post.
Also the hdb_linear_model2 was meant to show the result of overfitting the model, but bear in mind the run time for the code is quite long as well, hence for the purpose of quickly saving my render and post it, i simply mark as a comment only. 

I will quickly do a split of data by training data and testing data. After that compare the linear model against the random forest model and compare by root mean square error


```r
#split data
set.seed(123)
idx<-sample(nrow(hdbresale2),nrow(hdbresale2)*0.8)
housing_train<-hdbresale2[idx,]
housing_test<-hdbresale2[-idx,]
```


```r
#make predictions

housing_test$predict_lm<-predict(hdb_linear_model,housing_test)
```

Random forest model


```r
library(randomForest) # for new user of this library, remember to install.packages("randomForest")
```

```
## randomForest 4.7-1.1
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```
## The following object is masked from 'package:dplyr':
## 
##     combine
```

```r
hdb_rf_model<- randomForest(resale_price ~reg_year+floor_area_sqm+rem_lease_yr+flat_model, data=housing_train, ntree=4) # try this ntree variable. careful when using AI to solve as it causes lagness or jam
summary(hdb_rf_model)
```

```
##                 Length Class  Mode     
## call                4  -none- call     
## type                1  -none- character
## predicted       97180  -none- numeric  
## mse                 4  -none- numeric  
## rsq                 4  -none- numeric  
## oob.times       97180  -none- numeric  
## importance          4  -none- numeric  
## importanceSD        0  -none- NULL     
## localImportance     0  -none- NULL     
## proximity           0  -none- NULL     
## ntree               1  -none- numeric  
## mtry                1  -none- numeric  
## forest             11  -none- list     
## coefs               0  -none- NULL     
## y               97180  -none- numeric  
## test                0  -none- NULL     
## inbag               0  -none- NULL     
## terms               3  terms  call
```

```r
housing_test$predict_rf<-predict(hdb_rf_model,housing_test)
```


```r
# Calculate the root mean squared error for each model
linear_rmse <- sqrt(mean((housing_test$resale_price - housing_test$predict_lm)^2))
rf_rmse <- sqrt(mean((housing_test[, "resale_price"] - housing_test[,"predict_rf"])^2)) #this one is smaller
linear_rmse
```

```
## [1] 104962.9
```

```r
rf_rmse
```

```
## [1] 96286.66
```

Conclusion, you can see that random forest is better coming from the result of lower rmse. However, i note that my predictive modeling here need further refinement, as in to show hyperparameter tuning of a random forest model. I would create a different post to have a better structure of managing a predictive model. (Ie. Setting a recipe, baking, fit, analyze result)
