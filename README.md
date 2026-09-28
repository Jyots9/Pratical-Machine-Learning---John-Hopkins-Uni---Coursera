# Practical-Machine-Learning---John-Hopkins-Uni---Coursera


R version 4.0.0 (2020-04-24) -- "Arbor Day"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-pc-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

Welcome to your Lab Sandbox! To get started, please reference the README.rd file on the right side of your screen.
> #load the libraries
> library(rattle)
Loading required package: tibble
Loading required package: bitops
Rattle: A free graphical interface for data science with R.
Version 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.
Type 'rattle()' to shake, rattle, and roll your data.
> library(caret)
Loading required package: ggplot2
Keep up to date with changes at https://www.tidyverse.org/blog/
Loading required package: lattice
> library(rpart)
> library(rpart.plot)
> library(corrplot)
corrplot 0.90 loaded
> library(randomForest)
randomForest 4.6-14
Type rfNews() to see new features/changes/bug fixes.

Attaching package: ‘randomForest’

The following object is masked from ‘package:ggplot2’:

    margin

The following object is masked from ‘package:rattle’:

    importance

> library(RColorBrewer)
> #download data
> # training set
> download.file(
+     "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv",
+     destfile = "pml-training.csv",
+     method = "libcurl" )
trying URL 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv'
Content type 'text/csv' length 12202745 bytes (11.6 MB)
==================================================
downloaded 11.6 MB

> #test set
> download.file(
+     "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv",
+     destfile = "pml-testing.csv",
+     method = "libcurl")
trying URL 'https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv'
Content type 'text/csv' length 15113 bytes (14 KB)
==================================================
downloaded 14 KB

> library(caret)
> training <- read.csv("pml-training.csv",na.strings = c("NA", "", "#DIV/0!"))
> testing <- read.csv("pml-testing.csv",na.strings = c("NA", "", "#DIV/0!"))
> 
> # Near-zero variance predictors
> NZV <- nearZeroVar(training, saveMetrics = TRUE)
> 
> # Remove columns with many NAs
> naCols <- colSums(is.na(training)) > 0.95 * nrow(training)
> training_clean <- training[, !naCols]
> testing_clean  <- testing[, !naCols]
> # Remove NZV columns
> nzvCols <- nearZeroVar(training_clean)
> training_clean <- training_clean[, -nzvCols]
> testing_clean  <- testing_clean[, -nzvCols]
> regex <- grepl("^X|timestamp|user_name", names(training_clean))
> training_reg <- training_clean[, !regex]
> testing_reg <- testing_clean[, !regex]
> cond <- (colSums(is.na(training_reg)) == 0)
> training_cond <- training_reg[, cond]
> testing_cond <- testing_reg[, cond]
> corrplot(cor(training_cond[, -length(names(training_cond))]), method = "color", tl.cex = 0.5)
> inTrain <- createDataPartition(training_cond$classe, p = 0.70, list = FALSE)
> validation <- training[-inTrain, ]
> training <- training[inTrain, ]
> rm(inTrain)
> 
> table(validation$classe)

   A    B    C    D    E 
1674 1139 1026  964 1082 
> 
> table(predictTree)
Error in table(predictTree) : object 'predictTree' not found
> predictTree
Error: object 'predictTree' not found
> class(validation$classe)
[1] "character"
> class(predictTree)
Error: object 'predictTree' not found
> levels(validation$classe)
NULL
> levels(predictTree)
Error in levels(predictTree) : object 'predictTree' not found
> validation$classe <- factor( validation$classe,levels = c("A", "B", "C", "D", "E"))
> modelTree <- rpart(classe ~ ., data = training, method = "class")
> prp(modelTree)
> predictTree <- predict(modelTree, validation, type = "class")
> table(predictTree)
predictTree
   A    B    C    D    E 
1674 1139 1026  964 1082 
> 
> cm <- confusionMatrix(predictTree,validation$classe)
> accuracy <- postResample(predictTree, validation$classe)
> ose <- 1 - as.numeric(confusionMatrix(validation$classe, predictTree)$overall[1])
> rm(predictTree)
> rm(modelTree)
> modelRF <- train(classe ~ ., data = training, method = "rf", trControl = trainControl(method = "cv", 5), ntree = 250)
Error in na.fail.default(list(classe = c("A", "A", "A", "A", "A", "A",  : 
  missing values in object
> modelRF
Error: object 'modelRF' not found
> training <- training[, colSums(is.na(training)) == 0]
> testing  <- testing[, colSums(is.na(testing)) == 0]
> anyNA(training)
[1] FALSE
> modelRF <- train(classe ~ ., data = training, method = "rf", trControl = trainControl(method = "cv", 5), ntree = 250)
> modelRF
Random Forest 

13737 samples
   59 predictor
    5 classes: 'A', 'B', 'C', 'D', 'E' 

No pre-processing
Resampling: Cross-Validated (5 fold) 
Summary of sample sizes: 10990, 10989, 10989, 10991, 10989 
Resampling results across tuning parameters:

  mtry  Accuracy   Kappa    
   2    0.9949769  0.9936462
  41    0.9998544  0.9998159
  81    0.9997816  0.9997238

Accuracy was used to select the optimal model using the largest value.
The final value used for the model was mtry = 41.
> predictRF <- predict(modelRF, validation)
> confusionMatrix(validation$classe, predictRF)
Confusion Matrix and Statistics

          Reference
Prediction    A    B    C    D    E
         A 1674    0    0    0    0
         B    0 1139    0    0    0
         C    0    1 1025    0    0
         D    0    0    0  964    0
         E    0    0    0    0 1082

Overall Statistics
                                     
               Accuracy : 0.9998     
                 95% CI : (0.9991, 1)
    No Information Rate : 0.2845     
    P-Value [Acc > NIR] : < 2.2e-16  
                                     
                  Kappa : 0.9998     
                                     
 Mcnemar's Test P-Value : NA         

Statistics by Class:

                     Class: A Class: B Class: C Class: D Class: E
Sensitivity            1.0000   0.9991   1.0000   1.0000   1.0000
Specificity            1.0000   1.0000   0.9998   1.0000   1.0000
Pos Pred Value         1.0000   1.0000   0.9990   1.0000   1.0000
Neg Pred Value         1.0000   0.9998   1.0000   1.0000   1.0000
Prevalence             0.2845   0.1937   0.1742   0.1638   0.1839
Detection Rate         0.2845   0.1935   0.1742   0.1638   0.1839
Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
Balanced Accuracy      1.0000   0.9996   0.9999   1.0000   1.0000
> accuracy <- postResample(predictRF, validation$classe)
> ose <- 1 - as.numeric(confusionMatrix(validation$classe, predictRF)$overall[1])
> rm(predictRF)
> rm(accuracy)
> rm(ose)
> predict(modelRF, testing[, -length(names(testing))])
 [1] A A A A A A A A A A A A A A A A A A A A
Levels: A B C D E
> pml_write_files = function(x){
+     n = length(x)
+     for(i in 1:n){
+         filename = paste0("./Assignment_Solutions/problem_id_",i,".txt")
+         write.table(x[i], file = filename, quote = FALSE, row.names = FALSE, col.names = FALSE)
+     }
+ }
