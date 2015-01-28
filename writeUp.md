# Qualitative Activity Recognition - Reproducing Test Results
****

## Background
****
This project is based on the study of [Qualitative Activity Recognition of Weight Lifting Exercises](http://groupware.les.inf.puc-rio.br/public/papers/2013.Velloso.QAR-WLE.pdf). The basic idea of this particular study was to extend the quantitative approach of Human Activity Recognition(HAR) to a qualitative one. Many of the current approaches are limited to distinguish the human activities from each other such as sleeping, walking and sitting. However, the case can be that it is not just the quantity that is of interest but also the quality of the exercise. The authors of the study do show this in the example of weight training (using the dumbell raises as an example), where:  
1. the quantity of raises not necessarily represent the amount of exercise done  
2. wrong application can lead to injuries

The authors suggested a set to measure the movements with a specific sensor setup. And then instructed the test subject to simulate "good" and "bad" dumbell raises.    

## Goal of this Draft
****
In this draft we work with the sensor data and use them to train and validate two modes based on:  
1. Random Forests
2. Recursive Partitioning and Regression Trees

For this exercise we will be using the "caret" and "randomForest" packages available for r.


```
## Loading required package: lattice
## Loading required package: ggplot2
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```
  
## Data Preparation
****  
The initial data cleaning occurs when loading the data:  
1. Remove all columns that with zero variance  
2. Remove all columns that contain mostly NAs  
3. Remove all columns with non-numeric variables


```r
trainData <- read.csv("pml-training.csv")

nzv <- nearZeroVar(trainData)
trainData <- trainData[, -nzv]
trainData <- trainData[, colSums(is.na(trainData)) < 0.95*nrow(trainData)]
rem_col <- c("user_name", "cvtd_timestamp", "raw_timestamp_part_1", "raw_timestamp_part_2", "new_window", "X")
trainData <- trainData[, !(names(trainData) %in% rem_col)]
```

We will devide the data into 5 folds.The intent is to use 2 folds for each model, Random Forests and Recursive Partitioning and Regression Trees. One fold of each will be used for training purpose and the second set for validation.


```r
set.seed(32323)
folds <- createFolds(y = trainData$classe, k = 5, list = T, returnTrain = F)
```
  
## Model Comparison
****

The first model we choose is Recursive Partitioning and Regression Trees. We use a described part of the training data for training purposes and another part for validation. Note that we kept all parameter for the train function default.

```r
modelRpart <- train(classe~., method = "rpart", data = trainData[folds[[3]], ])
preRpart <- predict(modelRpart, trainData[folds[[4]],])
confuRpart <- confusionMatrix(trainData[folds[[4]], ]$classe, preRpart)
```

Overall Accuracy Estimate for the RPART test:

```
## [1] "Accuracy:"
```

```
## [1] 0.577
```

Table of the Confusion Matrix for a Recursive Partitioning and Regression Trees. Note that the RPART model is particularuly flawed, when it comes to distinguishing the cases A, B and C. 


```
##           Reference
## Prediction    A    B    C    D    E
##          A 1007    9   87    0   13
##          B  216  231  313    0    0
##          C  123   15  546    0    0
##          D   96  103  418    0   26
##          E   21   39  182    0  479
```

The second model we choose is Random Forests. Again we use part of the training data for training purposes and another part for validation. Note that we kept all parameter for the train function default except for the fact that we enable parallel processing.


```r
modelRf <- randomForest(classe~., data = trainData[folds[[1]], ], ntree = 100, importance = FALSE)
preRf <- predict(modelRf, trainData[folds[[2]], ])
confuRf <- confusionMatrix(trainData[folds[[2]], ]$classe, preRf)
```

Overall Accuracy Estimate for the Random Forest test:

```
## [1] "Accuracy:"
```

```
## [1] 0.982
```

Table of the confusionMatrix for the Random Forest run test.

```
##           Reference
## Prediction    A    B    C    D    E
##          A 1114    2    0    0    0
##          B   15  732   11    1    0
##          C    0   21  663    1    0
##          D    0    0   11  632    1
##          E    0    1    4    4  712
```
  
## Testing Data and Submission
****
As part of the exercise we were given a set of test data to varify our models


```r
testData <- read.csv("pml-testing.csv")
colnames(testData)[160] <- "classe"
testData <- testData[, names(testData)%in%names(trainData)]
```

Row one shows the predictive values of Recursive Partitioning and Regression trees and row two for Random Forests

```
##  [1] A A C A A C C C A A C C C A C C A A A C
## Levels: A B C D E
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```
   
**Submission results:**
****
- Recursive partitioning and regression trees get 10/20 correct;
- Random Forests gets 20/20 corret.
