#Getting and Cleaning Data Course Project at Coursera

First, environment is set for the data processing

## 0. Prepare environment
rm(list=ls())
if(dir.exists("./project")) {
  unlink("./project", recursive=TRUE)
}
dir.create("./project")

setwd("./project")
print("Warning: changing working directory.")
getwd()

### Load data.table package
library("data.table")

### Download dataset
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl, destfile="projectDataset.zip")

### Unzip dataset
unzip("projectDataset.zip")

## 1. Merge the training and the test sets to create one data set
train <- read.csv("UCI HAR Dataset/train/X_train.txt", sep="", header=FALSE)
train[,ncol(train)+1] <- read.csv("UCI HAR Dataset/train/y_train.txt", sep="", header=FALSE)
train[,ncol(train)+1] <- read.csv("UCI HAR Dataset/train/subject_train.txt", sep="", header=FALSE)

test <- read.csv("UCI HAR Dataset/test/X_test.txt", sep="", header=FALSE)
test[,ncol(test)+1] <- read.csv("UCI HAR Dataset/test/y_test.txt", sep="", header=FALSE)
test[,ncol(test)+1] <- read.csv("UCI HAR Dataset/test/subject_test.txt", sep="", header=FALSE)

merged <- rbind(train, test)

## 2. Extract only the measurements on the mean and standard deviation for each measurement
### Get column names
colNames = c(read.csv("UCI HAR Dataset/features.txt", sep="", header=FALSE, colClasses="character")[,2], "label", "subject")
colnames(merged) <- colNames

### Create a filter for column names
colFilter <- grep(".*mean\\(\\)|.*std\\(\\)", colNames)

### Subset merged table based on the col_filter_list to keep only the desired columns
extractData <- merged[,c(colFilter,ncol(merged),ncol(merged)-1)]

## 3. Use descriptive activity names to name the activities in the data set
### Get activity labels
activityLabels <- read.csv("UCI HAR Dataset/activity_labels.txt", sep="", header=FALSE,colClasses="character", col.names=c("id","label"))

## 4. Appropriately label the data set with descriptive variable names
extractData$label <- factor(extractData$label,levels=activityLabels$id,labels=activityLabels$label)

## 5. From the data set in step 4, create a second, independent tidy data set with the average of each variable for each activity and each subject.
dataTable <- data.table(extractData)
tidy <- dataTable[,lapply(.SD,mean),by="label,subject"]
write.table(tidy, file="tidy.txt",sep=",",row.names = FALSE)
