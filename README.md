## This file (run_analysis.R) is created to do Getting and Cleaning Data Course Project:
## 1. Merges the training and the test sets to create one data set.
## 2. Extracts only the measurements on the mean and standard deviation for each measurement.
## 3. Uses descriptive activity names to name the activities in the data set
## 4. Appropriately labels the data set with descriptive activity names.
## 5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

if (!require("data.table")) {
  install.packages("data.table")
	}
if (!require("reshape2")) {
  install.packages("reshape2")
	}
require("data.table")
require("reshape2")

## It loads data from the file saved in the working Directory: activity labels
activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")[,2]

## It Loads data from the file saved in the working Directory: data column names
features <- read.table("./UCI HAR Dataset/features.txt")[,2]

## Extract the measurements for the mean and standard deviation
extract_features <- grepl("mean|std", features)

## It loads and processes X_test and y_test data from the saved file
X_test <- read.table("./UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./UCI HAR Dataset/test/y_test.txt")
subject_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")

names(X_test) = features

## Extract the measurements for the mean and standard deviation
X_test = X_test[,extract_features]

## It loads activity labels
y_test[,2] = activity_labels[y_test[,1]]
names(y_test) = c("Activity_ID", "Activity_Label")
names(subject_test) = "subject"

## It binds the data labeled and measured previously 
test_data <- cbind(as.data.table(subject_test), y_test, X_test)

## It loads and processes X_train & y_train data
X_train <- read.table("./UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./UCI HAR Dataset/train/y_train.txt")

subject_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")

names(X_train) = features

## It extracted the measurements on the mean and standard deviation
X_train = X_train[,extract_features]

## It loads activity data
y_train[,2] = activity_labels[y_train[,1]]
names(y_train) = c("Activity_ID", "Activity_Label")
names(subject_train) = "subject"

## It binds the data labeled and measured previously 
train_data <- cbind(as.data.table(subject_train), y_train, X_train)

## It merges test and train data
data = rbind(test_data, train_data)

id_labels   = c("subject", "Activity_ID", "Activity_Label")
data_labels = setdiff(colnames(data), id_labels)
melt_data      = melt(data, id = id_labels, measure.vars = data_labels)

## It Applies mean function to dataset using dcast function
tidy_data   = dcast(melt_data, subject + Activity_Label ~ variable, mean)
write.table(tidy_data, file = "./tidy data.txt")
