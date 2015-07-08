#**Getting and cleaning data**
For creating a tidy data set of wearable computing data originally from http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

##**Files in this repo**

 1. **README.md** -- this file you are reading now
 2. **run_analysis.R** -- actual R code

##**run_analysis.R goals**

1. Merges the training and the test sets to create one data set. 
2. Extracts only the measurements on the mean and standard deviation for each measurement. 
3. Uses descriptive activity names to name the activities in the data set.
4. Appropriately labels the data set with descriptive activity names.
5.  Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

It should run in a folder of the Samsung data (the zip had this folder: UCI HAR Dataset) The script assumes it has in it's working directory the following files and folders:

activity_labels.txt
features.txt
test/
train/


##**run_analysis.R explanation**

It follows the goals step by step.

Step 1:

    # read all the data
    test.labels <- read.table("test/y_test.txt", col.names="label")
    test.subjects <- read.table("test/subject_test.txt", col.names="subject")
    test.data <- read.table("test/X_test.txt")
    train.labels <- read.table("train/y_train.txt", col.names="label")
    train.subjects <- read.table("train/subject_train.txt", col.names="subject")
    train.data <- read.table("train/X_train.txt")
    
    # put it together in the format of: subjects, labels, everything else
    data <- rbind(cbind(test.subjects, test.labels, test.data),
                  cbind(train.subjects, train.labels, train.data))


Read all the test and training files: *X_test.txt*, *y_test.txt* and *subject_test.txt*.

Combine the files to a data frame in the form of subjects, labels, the rest of the data.

Step 2:

    # read the features
    features <- read.table("features.txt", strip.white=TRUE, stringsAsFactors=FALSE)
    # only retain features of mean and standard deviation
    features.mean.std <- features[grep("mean\\(\\)|std\\(\\)", features$V2), ]
    
    # select only the means and standard deviations from data
    # increment by 2 because data has subjects and labels in the beginning
    data.mean.std <- data[, c(1, 2, features.mean.std$V1+2)]

Read the features from *features.txt* and filter it to only leave features that are either means ("mean()") or standard deviations ("std()"). The reason for leaving out meanFreq() is that the goal for this step is to only include means and standard deviations of measurements, of which meanFreq() is neither.
A new data frame is then created that includes subjects, labels and the described features.

Step 3:

    # read the labels (activities)
    labels <- read.table("activity_labels.txt", stringsAsFactors=FALSE)
    # replace labels in data with label names
    data.mean.std$label <- labels[data.mean.std$label, 2]

Read the activity labels from *activity_labels.txt* and replace the numbers with the text.

Step 4:

    # first make a list of the current column names and feature names
    good.colnames <- c("subject", "label", features.mean.std$V2)
    # then tidy that list
    # by removing every non-alphabetic character and converting to lowercase
    good.colnames <- tolower(gsub("[^[:alpha:]]", "", good.colnames))
    # then use the list as column names for data
    colnames(data.mean.std) <- good.colnames

Make a column list (includig "subjects" and "label" at the start)
Tidy the list by removing all non-alphanumeric characters and converting the result to lowercase
Apply the now-good-columnnames to the data frame

Step 5:

    # find the mean for each combination of subject and label
    aggr.data <- aggregate(data.mean.std[, 3:ncol(data.mean.std)], by=list(subject = data.mean.std$subject, label = data.mean.std$label),mean)



Create a new data frame by finding the mean for each combination of subject and label. It's done by aggregate() function.

    # write the data for course upload
    write.table(format(aggr.data, scientific=T), "data_set_with_the_averages.txt", row.names=F, col.names=F, quote=2)
    
Write the new tidy set into a text file called *data_set_with_the_averages.txt*, formatted similarly to the original files.
