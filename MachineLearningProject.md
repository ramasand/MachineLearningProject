---
title: "MachineLearningProject"
output: html_document
---
The goal of this project is to predict if barbell lifts was done correctly (class A) or incorrectly in corresponding to one of four common mistakes: 

* class B - throwing the elbows to the front 

* class C - lifting the dumbbell only halfway

* Class D - lowering the dumbbell only halfway

* class E - throwing the hips to the front

 6 participants were asked to perform barbell lifts correctly and incorrectly according to the four classes. Participants were supervised by an experienced weight lifter to make sure the execution complied to the manner they were supposed to simulate.
 Sensors that provide three-axes acceleration, gyroscope and magnetometer data, was mounted in the users' glove, armband, lumbar belt and dumbbell. 
The collected measurements will be used as predictors to build a prediction model to detect the class of the exercise. In this case we used "Random Forests" on the training set.

As the last part of this project we will use the prediction model in order to predict the classes of 20 different tests.

## Analysis

###1. Download the training and the test data


```r
library(randomForest)
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
if(!file.exists("data")){dir.create("data")}
fileUrl<-"http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
download.file(fileUrl,destfile="./data/pml-training.csv",method="auto")
training<-read.table("./data/pml-training.csv",header=TRUE,sep=",")

fileUrl<-"http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(fileUrl,destfile="./data/pml-testing.csv",method="auto")
testing<-read.table("./data/pml-testing.csv",header=TRUE,sep=",")
```

###2. Cleaning the data

* The downloaded data has some extra columns of unnecessary information (with processes on the raw data). Since we interesting in the raw data  only, we will delete these extra columns.

* The second step will be, to divide the raw data into training and testing set in order to be able to test and evaluating our fitting model.


```r
library(caret)
training<-training[,!(is.na(training[2,]) | training[2,]=="")] #choose only the raw data
training<-training[,8:dim(training)[2]]
set.seed(2412)
inTrain<-createDataPartition(y=training$classe,p=0.7,list=FALSE)
train<-training[inTrain,]; test<-training[-inTrain,]
```

###3. The fitting model and cross validation
We decided  to use "Random Forests" in order to build the prediction model, since we are interesting in high accuracy.  

The Random Forests uses Bootstrap Samples as a cross validation method. 


```r
set.seed(2412)
fit<-train(classe~.,data=train,methood="rf")
```

###4. Accuracy 

After the model was calculated, we can predict the classes of the test set and check the accuracy of the predictive model 


```r
library(knitr)
pred<-predict(fit,test)
cm<-confusionMatrix(pred,test$classe)
CM<-kable(cm$table)
```

#####The accuracy over the independent test set is: 0.9945624

The confusion matrix fot the test set:


|   |    A|    B|    C|   D|    E|
|:--|----:|----:|----:|---:|----:|
|A  | 1671|    4|    0|   0|    0|
|B  |    1| 1132|    5|   0|    1|
|C  |    2|    2| 1019|   8|    2|
|D  |    0|    1|    2| 955|    3|
|E  |    0|    0|    0|   1| 1076|




##Results

###Prediction of the test set

As a final step, we will predict the classes of the 20 test cases (afterthe preprocessing):


```r
library(knitr)
testing<-testing[,!(is.na(testing[2,]) | testing[2,]=="")] #choose only the raw data
testing<-testing[,8:dim(testing)[2]]
testingPred<-predict(fit,testing)
kable(data.frame("Test"=1:20,"Prediction"=testingPred))
```



| Test|Prediction |
|----:|:----------|
|    1|B          |
|    2|A          |
|    3|B          |
|    4|A          |
|    5|A          |
|    6|E          |
|    7|D          |
|    8|B          |
|    9|A          |
|   10|A          |
|   11|B          |
|   12|C          |
|   13|B          |
|   14|A          |
|   15|E          |
|   16|E          |
|   17|A          |
|   18|B          |
|   19|B          |
|   20|B          |


### References 
[1] Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements
