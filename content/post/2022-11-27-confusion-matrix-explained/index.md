---
title: Confusion matrix explained
author: R package build
date: '2022-11-27'
slug: confusion-matrix-explained
categories: []
tags: []
---

A confusion matrix is a table to define the performance of a classification algorithm.

|         |           | Predicted|Predicted|
|:--------:|:----------:|:--------:|:--------:|
|         |           |True|False|
|**Observed** |True|True Positive(TP)|False Negative(FN)|
|**Observed** |False|False Positive(FP)|True Negative(TN)|

Confusion matrix is easy to understand. “I am only confused by its name”

Three useful metrics are derived from this confusion matrix : Accuracy , Sensitivity and Specificity.

Accuracy = (TP+TN)/Total Predictions

Sensitivity = TP/(TP+FN)

Specificity = TN/(TN+FP)

Lets use the Covid-19 Antigen Rapid Test Kit as an example. For those uninitiated, or reading this many years in the future, Covid-19 is an infectious disease which causes pandemic outbreak since early 2020 till date. Rapid antigen test kit can screen for Covid-19 virus effectively and have become an effective screening tool for business, schools, and homes to use instead of going to a clinic to verify one as Covid-19 positive.

“This test kit XYZ has a sensitivity of 97% and specificity of 100%” says Adam.

What do Adam mean? If a positive person infected by Covid-19 did the test 100 times, turns out that the test kit indicates a negative result 3 times. And if a healthy person without Covid-19 infections did the test for 100 times, the test kit would never indicate a positive result, not even once.

“This test kit 123 has a sensitivity of 100% and specificity of 97%” says Michelle.

What do Michelle mean? If a positive person infected by Covid-19 did the test 100 times, turns out that the test kit indicates a positive Covid-19 result in all 100 tests. And if a healthy person without Covid-19 infections did the test for 100 times, the test kit would probably indicate a positive result 3 times.

Would one prefer to use test kit 123 or XYZ?

