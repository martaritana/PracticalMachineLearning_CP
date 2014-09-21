Predicting quality of barrel lifts
========================================================

Introduction
-----

The goal of the analysis is to predict how well the excersice is done using the data from accelerometers.

This analysis uses data from http://groupware.les.inf.puc-rio.br/har and caret R package.


Preparation
-----

The first column of datasets containted row numbers. It was skipped by subsetting datasets after loading.


```r
library(caret)
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", "pml-training.csv", method="curl")
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", "pml-testing.csv", method="curl")
training <- read.csv("pml-training.csv")
testing <- read.csv("pml-testing.csv")
training <- training[, 2:ncol(training)]
testing <- testing[, 2:ncol(testing)]
```

In purpose of speed up model traing the columns from traning set that contained less information was selected by nearZeroVar function and cut out. Columns that contained data not related to accelerometers (such as subject name and timestamps) were also subsetted out.

It were feagured out that some of the remaining columns in training set contained large number of NA's, but the others contained zero missing values. Such columns with large number of missing values were cutted off.


```r
nzv <- nearZeroVar(training, saveMetrics=TRUE)
trainFeatures <- training[, !nzv$nzv]
trainFeatures <- trainFeatures[, 6:ncol(trainFeatures)]
trainFeatures <- trainFeatures[, colSums(is.na(trainFeatures)) / nrow(trainFeatures) < 0.25]
```

Lastely the training dataset were partitioned to training and testing parts by ratio 75%/25%. Training part used in model training, testing part was used to determine quality of fitted model.


```r
inTrain <- createDataPartition(y = trainFeatures$classe, p=0.75, list=F)
training_set <- trainFeatures[inTrain, ]
testing_set <- trainFeatures[-inTrain, ]
```

Model fitting
-----

To fit a model, random forest algorithm were used. The algorithm was chosen for its high accuracy.


```r
modelFit2 <- randomForest(classe ~ ., data=training_set)
```

Model quality
-----

After predicting values for test set we see, that we have a model with 99.96% accuracy. We expect quiet low out of sample error rate of 0.04%.


```r
predictions <- predict(modelFit2, newdata=testing_set)
confusionMatrix(predictions, testing_set$classe)
```

```
Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 1395    3    0    0    0
         B    0  944    2    0    0
         C    0    2  853    6    0
         D    0    0    0  797    1
         E    0    0    0    1  900

Overall Statistics
                                         
               Accuracy : 0.9969         
                 95% CI : (0.995, 0.9983)
    No Information Rate : 0.2845         
    P-Value [Acc > NIR] : < 2.2e-16      
                                         
                  Kappa : 0.9961         
 Mcnemar's Test P-Value : NA             

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            1.0000   0.9947   0.9977   0.9913   0.9989
Specificity            0.9991   0.9995   0.9980   0.9998   0.9998
Pos Pred Value         0.9979   0.9979   0.9907   0.9987   0.9989
Neg Pred Value         1.0000   0.9987   0.9995   0.9983   0.9998
Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
Detection Rate         0.2845   0.1925   0.1739   0.1625   0.1835
Detection Prevalence   0.2851   0.1929   0.1756   0.1627   0.1837
Balanced Accuracy      0.9996   0.9971   0.9978   0.9955   0.9993
```
