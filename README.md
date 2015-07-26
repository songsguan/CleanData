# CleanData
Repo for Clean Data


Requirements:
1. You should create one R script called run_analysis.R that does the following. 
2. Merges the training and the test sets to create one data set.
3. Extracts only the measurements on the mean and standard deviation for each measurement. 
4. Uses descriptive activity names to name the activities in the data set
5. Appropriately labels the data set with descriptive variable names. 
6. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

The following R code is to clean up the raw data to the tidy dataset as required


##############################################################################
# 1. Merges the training and the test sets to create one data set.
##############################################################################

# Reading Train dataset
subject_train <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/train/subject_train.txt')
X_train <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/train/X_train.txt')
y_train <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/train/y_train.txt')

# rename the column
colnames(subject_train) <- 'subject'
colnames(y_train) <- 'activity_type'

# combine training dataset of subject, activity_type and measurements
train_data <- cbind(subject_train,y_train,X_train )

# check missing value
colSums(is.na(train_data))
sum(colSums(is.na(train_data)))


# Reading Tran dataset
subject_test <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/test/subject_test.txt')
X_test <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/test/X_test.txt')
y_test <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/test/y_test.txt')

# rename
colnames(subject_test) <- 'subject'
colnames(y_test) <- 'activity_type'

# combine test dataset of subject, activity_type and measurements
test_data <- cbind(subject_test,y_test,X_test )

# check missing value
colSums(is.na(test_data))
sum(colSums(is.na(test_data)))

# combine the training and testing dataset into full dataset
full_data <- rbind(train_data, test_data)


##############################################################################
# 2.Extracts only the measurements on the mean and standard deviation for each measurement. 
##############################################################################

# read the features names 
features <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/features.txt',sep=' ', header=F)

# assign the descriptive feature name to the full dataset
colnames(full_data)[-c(1:2)] <- as.character(features$V2)

# extract the measurements name for mean and standard deviation related
mean_col <- colnames(full_data)[grep('.mean.', colnames(full_data))]
std_col <- colnames(full_data)[grep('.std.', colnames(full_data))]

# get sub-dataset that only have mean and standard deviation measurements
sub_data <- full_data[,c('subject','activity_type',mean_col,std_col)]


##############################################################################
#3. Uses descriptive activity names to name the activities in the data set
##############################################################################

# read activity label file
activity_labels <- read.table('./getdata-projectfiles-UCI HAR Dataset/UCI HAR Dataset/activity_labels.txt', sep=' ', header=F)

# Join the subdataset with activity label by activity label value
sub_data2 <- merge(sub_data, activity_labels, by.x='activity_type', by.y='V1')
# Replace the activity label value to actual activity label names and drop the duplicated column
sub_data2$activity_type <- sub_data2$V2
sub_data2$V2 <- NULL

##############################################################################
# 4. Appropriately labels the data set with descriptive variable names. 
##############################################################################

colnames(sub_data2)
print('The dataset variable names have all already been descriptive names')

##############################################################################
# 5. From the data set in step 4, creates a second, independent tidy data set
#    with the average of each variable for each activity and each subject.
##############################################################################
library(reshape2)

# melt the data by subject and activity_type
sub_data2_melt <- melt(sub_data2, id = c("subject", "activity_type"))

# cast dataset to be mean of value for each variable by subject and activity_type
avg_measurement_sub_data2 <- dcast(sub_data2_melt, subject + activity_type ~ variable, mean)

##############################################################################
# Write the tidy dataset
##############################################################################
write.table(avg_measurement_sub_data2,'./tidy_data.txt',sep = '\t', row.names=F)