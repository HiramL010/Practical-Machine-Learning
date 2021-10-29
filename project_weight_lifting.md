---
title: "Weight Lifting Exercises Project"
output: 
  html_document: 
    keep_md: true
    fig_caption: true 
---

This document shows a Machine Learning model that can predict the manner that subjects in an experiment did a lifting exercise by using the data in: 
http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har.




## Packages
This are the packages that we are going to use in the project:


```r
library(readr)
library(caret)
library(tidyverse)
library(randomForest)
```
## Reading
First, we have to read the data:

```r
testing <- read.csv("pml-testing.csv")
training <- read.csv("pml-training.csv") 
```

# Cleaning
By inspecting the data, we can notice that there are several columns that do not have values or are NAs.

To identify easily these columns, we store their names in a variable: 

```r
lab <- names(training)
```

By visual identification, we select the columns in the data set that have valid values

```r
df <- subset(training, select = c(lab[7:11], lab[37:49],   lab[60:68],
                                  lab[84:86], lab[102], lab[113:124],
                                  lab[140], lab[151:160]))
```
Once the valid data set is defined, the classe column (which is what we want to predict) is changed from the characters A to E to an ID from 1 to 5, respectively:



```r
df$classe[df$classe == "A"] = 1
df$classe[df$classe == "B"] = 2
df$classe[df$classe == "C"] = 3
df$classe[df$classe == "D"] = 4
df$classe[df$classe == "E"] = 5
```

Then, we transform all the variables to numeric. And in the case of the classe column, we transform it as a factor since we want to do a classification.

```r
df[] <- lapply(df, function(x) as.numeric(as.character(x)))
df$classe <- as.factor(df$classe)
```

# Test and train data set
The "df" data set now is divided in two: train (70%) and test (30%). This partition is created to be able to train our ML model and, then, determine its accuracy using the predictions on the test data set.


```r
inTrain <- createDataPartition(df$classe, p = 0.7, list = FALSE)
train <- df[ inTrain,]
test <- df[-inTrain,]
```

# Training the model
For this classification problem, we decided to use a randomForest as the ML method. And as a preprocess, we standardize all the variables by "center" and "scale". 

```r
mod1 <- randomForest(classe ~ ., data = train,
              preProc = c("center", "scale"), verbose = TRUE)
```

# Prediction
Once the model is trained, we use the test data set to predict the classe given the test set:

```r
pred1 <- predict(mod1, test)
```

# Accuracy
To determine the accuracy of our model, we count and sum up the classe correctly predicted by "mod1" and we divided by the total of elements in the test classe column.

```r
#model 1 accuracy:
sum(pred1==test$classe)/length(test$classe)
```

```
## [1] 0.9986406
```
As the above shows, the ML model defined in this project has an accuracy of 0.99 which shows that the model can predict with a high precision the manner in which the subjects did the exercise.


# Code for quiz
To solve the quiz, we used our model together with the testing data set: 

```r
x <- predict(mod1, testing)
```
This shows the manner in which the exercise was done in all 20 test cases.


