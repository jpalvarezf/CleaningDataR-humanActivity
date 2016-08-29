# CleaningDataR-humanActivity
Project for the R course in Coursera: Getting and Cleaning Data - August 2016

## Description
This document describes the procedures and scripts of how the Human Activity datasets can be cleaned, joined and transformed.

The goal is to make a tidy dataset that can be used by a broad data analysis audience.

**_GO TO the `codebook.md` file to read an extended explanation of the project_**

## Scripts
Refer to the comments in the code to read the explanation of each step

```{r}
# Title: Run Analysis Scripts
# Autor: Juan Pablo Alvarez
# Date: 2016-08-26

# Install packages if necessary
# install.packages("dplyr")
# install.packages("reshape2")

# Libraries
library(dplyr)
library(reshape2)


# Obtain TRAIN DATA
trainFile <- "./train/X_train.txt"
trainLabelsFile <- "./train/y_train.txt"
trainSubjectFile <- "./train/subject_train.txt"
# Main TRAIN DATA file
train <- read.table(trainFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
# TRAIN labels file
trainLabels <- read.table(trainLabelsFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
names(trainLabels) <- c("Label")
train <- bind_cols(trainLabels, train)
# TRAIN subject file
trainSubject <- read.table(trainSubjectFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
names(trainSubject) <- c("Subject")
train <- bind_cols(trainSubject, train)

# Obtain TEST DATA
testFile <- "./test/X_test.txt"
testLabelsFile <- "./test/y_test.txt"
testSubjectFile <- "./test/subject_test.txt"
# Main TEST DATA file
test <- read.table(testFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
# TEST labels file
testLabels <- read.table(testLabelsFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
names(testLabels) <- c("Label")
test <- bind_cols(testLabels, test)
# TEST subject file
testSubject <- read.table(testSubjectFile, header = FALSE, sep = "", quote = "", dec = ".", comment.char = "", colClasses = "numeric")
names(testSubject) <- c("Subject")
test <- bind_cols(testSubject, test)

# Bind by rows TRAIN and TEST DATA
humanActivity <- bind_rows(train, test)

# Obtain features
haColNamesFile <- "./features.txt"
haColNames <- read.table(haColNamesFile, header = FALSE, sep = "", quote = "", comment.char = "", stringsAsFactors = FALSE)

# Clean and make more readable the dataset columns names
haColNames$V2 <- gsub("\\(", "", haColNames$V2)
haColNames$V2 <- gsub("\\)", "", haColNames$V2)
haColNames$V2 <- gsub("\\,", "_", haColNames$V2)
haColNames$V2 <- gsub("\\-", "_", haColNames$V2)
# Make a unique name for each feature
haColNames$V3 <- paste0(haColNames$V2, "-", haColNames$V1)
names(humanActivity) <- c("Subject", "Labels", haColNames$V3)

# Replaces Labels codes with readable labels names and Extracts only the measurements on the mean and standard deviation
activityNamesFile <- "./activity_labels.txt"
activityNames <- read.table(activityNamesFile, header = FALSE, sep = "", quote = "", comment.char = "", stringsAsFactors = FALSE)
humanActivity_use <- humanActivity %>% select(Subject, Labels, contains("mean"), contains("std")) %>% mutate(Labels = plyr::mapvalues(Labels, activityNames$V1, activityNames$V2))

# Obtain the mean of features for each label and each subject
humanActivity_summarize <- humanActivity_use %>% group_by(Subject, Labels) %>% summarise_each(funs(mean))

# Write to a file the resulting summarized dataset to evaluate the final output
write.table(humanActivity_summarize, "./humanActivitySummarize.txt", row.names = FALSE)

```
