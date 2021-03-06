Final Project for Practical Machine Learning
================

Set up
------

``` r
library(dplyr)
library(plyr)
library(caret)
library(randomForest)
library(randomForestSRC)
```

Report
------

``` r
# Read data
# Before reading data, we copy test data to the bottom of train data.
Raw <- read.csv("pml-training.csv")
trainRaw <- Raw[1:19623,]
testRaw <- Raw[19623:19643,]

# use data collected from accelerometers on the belt, forearm, arm, and dumbell
filter = grepl("belt|arm|dumbell|class", names(trainRaw))
train_1 <- trainRaw[, filter]

# remove variables with more than 80% missing values
na <- sapply(colnames(train_1), function(x) if(sum(is.na(train_1[, x]))
                                                > 0.8*nrow(train_1))
             {return(1)}else{return(0)})
train_clean <- train_1[, !na]

# Perform the same steps to testing data
selected_variables <- colnames(train_clean)
test_clean <- testRaw[,selected_variables]

# divide the training data into pure training data (70%) and 
# cross-validation data (30%)
set.seed(123) # For reproducibile purpose
inTrain <- createDataPartition(train_clean$classe, p=0.70, list=F)
train_clean <- train_clean[inTrain, ]
crossval_clean <- train_clean[-inTrain, ]

# calculate correlations between each x-variables to classe
cor <- abs(sapply(colnames(train_clean[, 1:(ncol(train_clean)-1)]), 
                  function(x) cor(as.numeric(train_clean[, x]), 
                                  as.numeric(train_clean$classe), method = "spearman")))

summary(cor)
```

    ##      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
    ## 0.0001082 0.0091110 0.0147400 0.0627800 0.0974900 0.3171000

``` r
# As indicated above, the maximum correlation between x-variables and classe is
# 0.3171, which indicates linear regression is not suitable. We thus build model
# using random forest.

##################################################################################
# Build models
fitrf <-  rfsrc(classe ~ ., 
                 data = train_clean, 
                 na.action = "na.impute",
                 nsplit = 5,
                 ntree = 200
)
fitrf
```

    ##                          Sample size: 13738
    ##            Frequency of class labels: 3907, 2658, 2396, 2252, 2525
    ##                      Number of trees: 200
    ##           Minimum terminal node size: 1
    ##        Average no. of terminal nodes: 1461.15
    ## No. of variables tried at each split: 8
    ##               Total no. of variables: 63
    ##                             Analysis: RF-C
    ##                               Family: class
    ##                       Splitting rule: gini *random*
    ##        Number of random split points: 5
    ##               Normalized Brier score: 8.91 
    ##                           Error rate: 0.01, 0, 0.01, 0.02, 0.01, 0
    ## 
    ## Confusion matrix:
    ## 
    ##           predicted
    ##   observed    A    B    C    D    E class.error
    ##          A 3899    3    4    1    0      0.0020
    ##          B    8 2638   12    0    0      0.0075
    ##          C    0   31 2347   18    0      0.0205
    ##          D    2    0   22 2225    3      0.0120
    ##          E    0    1    2    6 2516      0.0036
    ## 
    ##  Overall error rate: 0.84%

``` r
# We test the model on cross-validation data
pred <- predict(fitrf, crossval_clean)
pred
```

    ##   Sample size of test (predict) data: 4109
    ##                 Number of grow trees: 200
    ##   Average no. of grow terminal nodes: 1461.15
    ##          Total no. of grow variables: 63
    ##                             Analysis: RF-C
    ##                               Family: class
    ##        Test set Normalized Brier score: 1.25 
    ##                  Test set error rate: 0, 0, 0, 0, 0, 0
    ## 
    ## Confusion matrix:
    ## 
    ##           predicted
    ##   observed    A   B   C   D   E class.error
    ##          A 1176   0   0   0   0           0
    ##          B    0 786   0   0   0           0
    ##          C    0   0 726   0   0           0
    ##          D    0   0   0 673   0           0
    ##          E    0   0   0   0 748           0
    ## 
    ##  Overall error rate: 0%

``` r
# We predict the test result
predtest <- predict(fitrf, test_clean)
```
