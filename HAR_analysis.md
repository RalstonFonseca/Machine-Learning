---
title: "Human Activity Recognition Project"
author: "Ralston Fonseca"
date: "October 3, 2018"
output: 
  html_document: 
    keep_md: yes
---



# Overview

People regularly quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, we will use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: (http://groupware.les.inf.puc-rio.br/har) _(see the section on the Weight Lifting Exercise Dataset)_.

# Strategy
#### I) Data
We have 2 sets of data:

- The training set (https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv) is partioned into 2 sets:
    1) Training data
    2) Test data, used to check the Accuracy.
- The Test data (https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv), which will be used for cross-validation.

#### II) Model Selection
We will build different models (using training data) and check their accuracy with test data. Based on their accuracy and performance we will select an appropriate model for our prediction. We will use _caret_ package in R for building our models.
 
We will use the following methods for model creation:

- MODEL1: _Bagging_, using _treebag_ method
- MODEL2: _Decision Tree_ using _rpart_ method
- MODEL3: _Random Forrest_, using _rf_ method
- MODEL4: _Boosting_, using _gbm_ method

If we do not get satisfactory results from above methods then we will explore combination of models.

#### III) Prediction based on selected model
Finally, based on the model selected we will predict the values for cross-validation data.

## I) Data
Let's download and load the data in R.
The original training data _(pml-training.csv)_ has **19622** observations and **160** variables (columns). The original testing data used from cross-validation has **20** observations and **160** variables. 


```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## [1] 19622   160
```

```
## [1]  20 160
```

We observe that many variables are have having no data i.e NAs or are empty (""). Our strategy will be to eliminate variables where more than _50%_ data is NOT available. We won't be getting good model fit with such cases. Following **100** variables were found with this condition.


```
##   [1] "kurtosis_roll_belt"       "kurtosis_picth_belt"     
##   [3] "kurtosis_yaw_belt"        "skewness_roll_belt"      
##   [5] "skewness_roll_belt.1"     "skewness_yaw_belt"       
##   [7] "max_roll_belt"            "max_picth_belt"          
##   [9] "max_yaw_belt"             "min_roll_belt"           
##  [11] "min_pitch_belt"           "min_yaw_belt"            
##  [13] "amplitude_roll_belt"      "amplitude_pitch_belt"    
##  [15] "amplitude_yaw_belt"       "var_total_accel_belt"    
##  [17] "avg_roll_belt"            "stddev_roll_belt"        
##  [19] "var_roll_belt"            "avg_pitch_belt"          
##  [21] "stddev_pitch_belt"        "var_pitch_belt"          
##  [23] "avg_yaw_belt"             "stddev_yaw_belt"         
##  [25] "var_yaw_belt"             "var_accel_arm"           
##  [27] "avg_roll_arm"             "stddev_roll_arm"         
##  [29] "var_roll_arm"             "avg_pitch_arm"           
##  [31] "stddev_pitch_arm"         "var_pitch_arm"           
##  [33] "avg_yaw_arm"              "stddev_yaw_arm"          
##  [35] "var_yaw_arm"              "kurtosis_roll_arm"       
##  [37] "kurtosis_picth_arm"       "kurtosis_yaw_arm"        
##  [39] "skewness_roll_arm"        "skewness_pitch_arm"      
##  [41] "skewness_yaw_arm"         "max_roll_arm"            
##  [43] "max_picth_arm"            "max_yaw_arm"             
##  [45] "min_roll_arm"             "min_pitch_arm"           
##  [47] "min_yaw_arm"              "amplitude_roll_arm"      
##  [49] "amplitude_pitch_arm"      "amplitude_yaw_arm"       
##  [51] "kurtosis_roll_dumbbell"   "kurtosis_picth_dumbbell" 
##  [53] "kurtosis_yaw_dumbbell"    "skewness_roll_dumbbell"  
##  [55] "skewness_pitch_dumbbell"  "skewness_yaw_dumbbell"   
##  [57] "max_roll_dumbbell"        "max_picth_dumbbell"      
##  [59] "max_yaw_dumbbell"         "min_roll_dumbbell"       
##  [61] "min_pitch_dumbbell"       "min_yaw_dumbbell"        
##  [63] "amplitude_roll_dumbbell"  "amplitude_pitch_dumbbell"
##  [65] "amplitude_yaw_dumbbell"   "var_accel_dumbbell"      
##  [67] "avg_roll_dumbbell"        "stddev_roll_dumbbell"    
##  [69] "var_roll_dumbbell"        "avg_pitch_dumbbell"      
##  [71] "stddev_pitch_dumbbell"    "var_pitch_dumbbell"      
##  [73] "avg_yaw_dumbbell"         "stddev_yaw_dumbbell"     
##  [75] "var_yaw_dumbbell"         "kurtosis_roll_forearm"   
##  [77] "kurtosis_picth_forearm"   "kurtosis_yaw_forearm"    
##  [79] "skewness_roll_forearm"    "skewness_pitch_forearm"  
##  [81] "skewness_yaw_forearm"     "max_roll_forearm"        
##  [83] "max_picth_forearm"        "max_yaw_forearm"         
##  [85] "min_roll_forearm"         "min_pitch_forearm"       
##  [87] "min_yaw_forearm"          "amplitude_roll_forearm"  
##  [89] "amplitude_pitch_forearm"  "amplitude_yaw_forearm"   
##  [91] "var_accel_forearm"        "avg_roll_forearm"        
##  [93] "stddev_roll_forearm"      "var_roll_forearm"        
##  [95] "avg_pitch_forearm"        "stddev_pitch_forearm"    
##  [97] "var_pitch_forearm"        "avg_yaw_forearm"         
##  [99] "stddev_yaw_forearm"       "var_yaw_forearm"
```

```
## [1] 19622    60
```
After eliminating them we are left with **60** variables in training data.

Following variables do not contribute to the outcome so will be eliminating them too

```
## [1] "X"                    "user_name"            "raw_timestamp_part_1"
## [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
## [7] "num_window"
```

```
## [1] 19622    53
```

Performing the equivalent operations on test data to eliminate the variables.

```
## [1] 20 52
```

Dividing training data such that _70%_ is allocated for training model fit and remaining _30%_ for testing and calculating the accuracy.

```
## [1] 13737    53
```

```
## [1] 5885   53
```

## II) Model Selection
Let's explore different methods to find the best model.

#### MODEL1: _Bagging_, using _treebag_ method


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1658   14    2    3    1
##          B   12 1108    9    1    4
##          C    3    9 1011   14    4
##          D    1    8    4  946    5
##          E    0    0    0    0 1068
```

```
##  Accuracy 
## 0.9840272
```

We get _98.40%_ accuracy using _treebag_ method.

#### MODEL2: _Decision Tree_ using _rpart_ method


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1525  464  479  462  155
##          B   21  386   30  163  155
##          C  123  289  517  339  291
##          D    0    0    0    0    0
##          E    5    0    0    0  481
```

```
##  Accuracy 
## 0.4943076
```

![](HAR_analysis_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

We get a low _49.43%_ accuracy using _rpart_ method and is not encouraging.

#### MODEL3: _Random Forrest_, using _rf_ method


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1673    5    0    0    0
##          B    0 1131    6    0    0
##          C    1    2 1018    8    2
##          D    0    1    2  956    2
##          E    0    0    0    0 1078
```

```
##  Accuracy 
## 0.9950722
```

![](HAR_analysis_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

The accuracy hits the peak at around 27 predictors and then starts declining.

We get  _99.51%_ accuracy using _rf_ method.

#### MODEL4: _Boosting_, using _gbm_ method


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1648   40    0    0    3
##          B   18 1062   30    2    9
##          C    4   33  988   28    9
##          D    4    3    8  932   10
##          E    0    1    0    2 1051
```

```
##  Accuracy 
## 0.9653356
```

![](HAR_analysis_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

We get _96.53%_ accuracy using _gbm_ method.


## III) Prediction based on selected model

##### Conclusion
Bagging _(98.40%)_, Random Forest _(99.51%)_ and Boosting models _(96.53%)_ provide above 95% accuracy. I have decided to go with Random Forest as it provides high accuracy and with reasonable performance. 

The Final predicted values from 20 observations from test data is as follows:


```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```





