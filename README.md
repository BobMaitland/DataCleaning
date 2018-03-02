# DataCleaning
Week 4 assignment submission "Getting and Cleaning Data"
#Project - Cleaning & Tidying Data
#
library("dplyr")
library("tidyr")
#
#Step 1 - Reading data files to R
s_test<-read.table("test/subject_test.txt")
x_test<-read.table("test/X_test.txt")
y_test<-read.table("test/y_test.txt")
#
#s_test has 2,947 obs of 1 vbl (subject identifier, integer, 1 - 30); x_test has 2,947 obs of 561 vbls; 
#y_test has 2,947 obs. of 1 vbl (test labels, integer, 1 - 6).
#
s_train<-read.table("train/subject_train.txt")
x_train<-read.table("train/X_train.txt")
y_train<-read.table("train/y_train.txt")
#
# s_train has 7,352 obs of 1 vbl(subject identifier, integer, 1 - 30); x_test has 7,352 obs of 561 vbls; 
# y_test has 7,352 obs. of 1 vbl(test labels, integer, 1 - 6).
#
# Step 1 - Reading features and activity labels
#
activity_labels<-read.table("activity_labels.txt")
features<-read.table("features.txt", stringsAsFactors=FALSE)
#
# 'activity_labels' are the 6 labels (e.g. 'WALKING') associated with the test_labels shown
# in y_test and y_train
#
# 'features' contain the 561 feature labels associated with the columns in x_test and x_train
# Three groups of 42 labels (303-344 (fBodyAcc-bandsEnergy); 384-423(fBodyAccJerk-bandsEnergy); 461-502(fBodyGyro-bandsEnergy) 
# are 3 repetitions of 14 unique labels.'features' therefore contains 477 unique labels.
#
# Step 2 - Merge data sets into 1 data set, containing 10,299 observations of just the mean and std measures
# Method: 
# 2.0) rename columns of s_test and s_train - SUB; rename cols of y_test and y_train - ACT
#
colnames(s_test)[1]<-"SUB"
colnames(s_train)[1]<-"SUB"
colnames(y_test)[1]<-"ACT"
colnames(y_train)[1]<-"ACT"
#
# 2.1) merge x_test and x_train, to create a new table resultsAll (10,299 by 561)
#
resultsAll<-rbind(x_test, x_train)
#
# 2.2) merge s_test and s_train to create a new vector subjectLabelsAll (10,299 by 1)
#      merge y_test and y_train to create a new vector activityLabelsAll (10,299 by 1)
#
subjectLabelsAll<-rbind(s_test, s_train)
activityLabelsAll<-rbind(y_test, y_train)
#
# 2.3)select columns for mean and st. dev. from resultsAll, to create resultsMeanStd (10,299 by 79)
#    	note that 46 variables contain the string "-mean", and 33 contain the string "-std"
#    	obtain the vector of features that contain the string "-mean" or "-std"
#    	filter 'features' to contain just these variables, select the columns of values associated with these vbls from resultsAll
#     rename the columns of resultsMeanStd, using the same filter
#     use the labels from the filtered 'features' to provide descriptive column names for resultsMeanStd
#
mean_std_col_filter<-filter(features, grepl("-mean()|-std()", V2))
resultsMeanStd<-resultsAll[,mean_std_col_filter[,1]]
colnames(resultsMeanStd)<-mean_std_col_filter[,2]
#
# 2.4) merge subjectLabelsAll, activityLabelsAll and resultsMeanStd, to create a new table, resultsLabelled, with 10,299 rows and 81            variables
#
resultsLabelled<-cbind(subjectLabelsAll, activityLabelsAll, resultsMeanStd)
#
# 2.5)  merge 'activity_labels' with resultsLabelled using the activity number in each
#       remove the first column of the merged table (activity number), and rename the activity colum 'Activity', to form summResults
#
summResults<-merge(activity_labels, resultsLabelled, by.x="V1", by.y="ACT")
summResults[1]<-NULL
names(summResults)[1]<-"Activity"
#
# 2.6) Group the data by 'Activity' and 'SUB', and summarize_all to form the required 'tidy' data set.
#
tidyResults<-group_by(summResults, Activity, SUB) 
