run_analysis
============
Last updated 2016-05-16 00:57:19 using R version 3.2.4 (2016-03-10).


Instructions for project
------------------------

> The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.  
>
> One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:
>
> http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones
>
> Here are the data for the project:
>
> https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip
>
> You should create one R script called run_analysis.R that does the following.
>
> 1. **DONE** Merges the training and the test sets to create one data set.
> 2. **DONE** Extracts only the measurements on the mean and standard deviation for each measurement.
> 3. **DONE** Uses descriptive activity names to name the activities in the data set.
> 4. **DONE** Appropriately labels the data set with descriptive activity names.
> 5. **DONE** Creates a second, independent tidy data set with the average of each variable for each activity and each subject.
>
> Good luck!

**The codebook is at the end of this document.**


Preliminaries
-------------

Load packages.


```r
packages <- c("data.table", "reshape2")
sapply(packages, require, character.only=TRUE, quietly=TRUE)
```

```
## Warning in library(package, lib.loc = lib.loc, character.only = TRUE,
## logical.return = TRUE, : aucun package nommé 'data.table' n'est trouvé
```

```
## Warning in library(package, lib.loc = lib.loc, character.only = TRUE,
## logical.return = TRUE, : aucun package nommé 'reshape2' n'est trouvé
```

```
## data.table   reshape2 
##      FALSE      FALSE
```

Set path.


```r
path <- getwd()
path
```

```
## [1] "/Users/joseph/Desktop/DataScience/C4W4/Project"
```


Get the data
------------

Download the file. Put it in the `Data` folder. **This was already done on 2014-04-11; save time and don't evaluate again.**


```r
url <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
f <- "Dataset.zip"
if (!file.exists(path)) {dir.create(path)}
download.file(url, file.path(path, f))
```

Unzip the file. **This was already done on 2014-04-11; save time and don't evaluate again.**


```r
executable <- file.path("C:", "Program Files (x86)", "7-Zip", "7z.exe")
parameters <- "x"
cmd <- paste(paste0("\"", executable, "\""), parameters, paste0("\"", file.path(path, f), "\""))
system(cmd)
```

The archive put the files in a folder named `UCI HAR Dataset`. Set this folder as the input path. List the files here.


```r
pathIn <- file.path(path, "UCI HAR Dataset")
list.files(pathIn, recursive=TRUE)
```

```
## character(0)
```

**See the `README.txt` file in /Users/joseph/Desktop/DataScience/C4W4/Project for detailed information on the dataset.**

For the purposes of this project, the files in the `Inertial Signals` folders are not used.


Read the files
--------------

Read the subject files.


```r
dtSubjectTrain <- fread(file.path(pathIn, "train", "subject_train.txt"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

```r
dtSubjectTest  <- fread(file.path(pathIn, "test" , "subject_test.txt" ))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

Read the activity files. For some reason, these are called *label* files in the `README.txt` documentation.


```r
dtActivityTrain <- fread(file.path(pathIn, "train", "Y_train.txt"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

```r
dtActivityTest  <- fread(file.path(pathIn, "test" , "Y_test.txt" ))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

Read the data files. `fread` seems to be giving me some trouble reading files. Using a helper function, read the file with `read.table` instead, then convert the resulting data frame to a data table. Return the data table.


```r
fileToDataTable <- function (f) {
	df <- read.table(f)
	dt <- data.table(df)
}
dtTrain <- fileToDataTable(file.path(pathIn, "train", "X_train.txt"))
```

```
## Warning in file(file, "rt"): impossible d'ouvrir le fichier '/Users/joseph/
## Desktop/DataScience/C4W4/Project/UCI HAR Dataset/train/X_train.txt' : No
## such file or directory
```

```
## Error in file(file, "rt"): impossible d'ouvrir la connexion
```

```r
dtTest  <- fileToDataTable(file.path(pathIn, "test" , "X_test.txt" ))
```

```
## Warning in file(file, "rt"): impossible d'ouvrir le fichier '/Users/joseph/
## Desktop/DataScience/C4W4/Project/UCI HAR Dataset/test/X_test.txt' : No such
## file or directory
```

```
## Error in file(file, "rt"): impossible d'ouvrir la connexion
```


Merge the training and the test sets
------------------------------------

Concatenate the data tables.


```r
dtSubject <- rbind(dtSubjectTrain, dtSubjectTest)
```

```
## Error in rbind(dtSubjectTrain, dtSubjectTest): objet 'dtSubjectTrain' introuvable
```

```r
setnames(dtSubject, "V1", "subject")
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setnames"
```

```r
dtActivity <- rbind(dtActivityTrain, dtActivityTest)
```

```
## Error in rbind(dtActivityTrain, dtActivityTest): objet 'dtActivityTrain' introuvable
```

```r
setnames(dtActivity, "V1", "activityNum")
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setnames"
```

```r
dt <- rbind(dtTrain, dtTest)
```

```
## Error in rbind(dtTrain, dtTest): objet 'dtTrain' introuvable
```

Merge columns.


```r
dtSubject <- cbind(dtSubject, dtActivity)
```

```
## Error in cbind(dtSubject, dtActivity): objet 'dtSubject' introuvable
```

```r
dt <- cbind(dtSubject, dt)
```

```
## Error in cbind(dtSubject, dt): objet 'dtSubject' introuvable
```

Set key.


```r
setkey(dt, subject, activityNum)
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setkey"
```


Extract only the mean and standard deviation
--------------------------------------------

Read the `features.txt` file. This tells which variables in `dt` are measurements for the mean and standard deviation.


```r
dtFeatures <- fread(file.path(pathIn, "features.txt"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

```r
setnames(dtFeatures, names(dtFeatures), c("featureNum", "featureName"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setnames"
```

Subset only measurements for the mean and standard deviation.


```r
dtFeatures <- dtFeatures[grepl("mean\\(\\)|std\\(\\)", featureName)]
```

```
## Error in eval(expr, envir, enclos): objet 'dtFeatures' introuvable
```

Convert the column numbers to a vector of variable names matching columns in `dt`.


```r
dtFeatures$featureCode <- dtFeatures[, paste0("V", featureNum)]
```

```
## Error in eval(expr, envir, enclos): objet 'dtFeatures' introuvable
```

```r
head(dtFeatures)
```

```
## Error in head(dtFeatures): objet 'dtFeatures' introuvable
```

```r
dtFeatures$featureCode
```

```
## Error in eval(expr, envir, enclos): objet 'dtFeatures' introuvable
```

Subset these variables using variable names.


```r
select <- c(key(dt), dtFeatures$featureCode)
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "key"
```

```r
dt <- dt[, select, with=FALSE]
```

```
## Error in eval(expr, envir, enclos): objet 'select' introuvable
```


Use descriptive activity names
------------------------------

Read `activity_labels.txt` file. This will be used to add descriptive names to the activities.


```r
dtActivityNames <- fread(file.path(pathIn, "activity_labels.txt"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "fread"
```

```r
setnames(dtActivityNames, names(dtActivityNames), c("activityNum", "activityName"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setnames"
```


Label with descriptive activity names
-----------------------------------------------------------------

Merge activity labels.


```r
dt <- merge(dt, dtActivityNames, by="activityNum", all.x=TRUE)
```

```
## Error in as.data.frame.default(x): impossible de convertir automatiquement la classe  ""function"" en un tableau de données (data.frame)
```

Add `activityName` as a key.


```r
setkey(dt, subject, activityNum, activityName)
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setkey"
```

Melt the data table to reshape it from a short and wide format to a tall and narrow format.


```r
dt <- data.table(melt(dt, key(dt), variable.name="featureCode"))
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "data.table"
```

Merge activity name.


```r
dt <- merge(dt, dtFeatures[, list(featureNum, featureCode, featureName)], by="featureCode", all.x=TRUE)
```

```
## Error in as.data.frame.default(x): impossible de convertir automatiquement la classe  ""function"" en un tableau de données (data.frame)
```

Create a new variable, `activity` that is equivalent to `activityName` as a factor class.
Create a new variable, `feature` that is equivalent to `featureName` as a factor class.


```r
dt$activity <- factor(dt$activityName)
```

```
## Error in dt$activityName: objet de type 'closure' non indiçable
```

```r
dt$feature <- factor(dt$featureName)
```

```
## Error in dt$featureName: objet de type 'closure' non indiçable
```

Seperate features from `featureName` using the helper function `grepthis`.


```r
grepthis <- function (regex) {
  grepl(regex, dt$feature)
}
## Features with 2 categories
n <- 2
y <- matrix(seq(1, n), nrow=n)
x <- matrix(c(grepthis("^t"), grepthis("^f")), ncol=nrow(y))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featDomain <- factor(x %*% y, labels=c("Time", "Freq"))
```

```
## Error in factor(x %*% y, labels = c("Time", "Freq")): objet 'x' introuvable
```

```r
x <- matrix(c(grepthis("Acc"), grepthis("Gyro")), ncol=nrow(y))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featInstrument <- factor(x %*% y, labels=c("Accelerometer", "Gyroscope"))
```

```
## Error in factor(x %*% y, labels = c("Accelerometer", "Gyroscope")): objet 'x' introuvable
```

```r
x <- matrix(c(grepthis("BodyAcc"), grepthis("GravityAcc")), ncol=nrow(y))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featAcceleration <- factor(x %*% y, labels=c(NA, "Body", "Gravity"))
```

```
## Error in factor(x %*% y, labels = c(NA, "Body", "Gravity")): objet 'x' introuvable
```

```r
x <- matrix(c(grepthis("mean()"), grepthis("std()")), ncol=nrow(y))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featVariable <- factor(x %*% y, labels=c("Mean", "SD"))
```

```
## Error in factor(x %*% y, labels = c("Mean", "SD")): objet 'x' introuvable
```

```r
## Features with 1 category
dt$featJerk <- factor(grepthis("Jerk"), labels=c(NA, "Jerk"))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featMagnitude <- factor(grepthis("Mag"), labels=c(NA, "Magnitude"))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
## Features with 3 categories
n <- 3
y <- matrix(seq(1, n), nrow=n)
x <- matrix(c(grepthis("-X"), grepthis("-Y"), grepthis("-Z")), ncol=nrow(y))
```

```
## Error in dt$feature: objet de type 'closure' non indiçable
```

```r
dt$featAxis <- factor(x %*% y, labels=c(NA, "X", "Y", "Z"))
```

```
## Error in factor(x %*% y, labels = c(NA, "X", "Y", "Z")): objet 'x' introuvable
```

Check to make sure all possible combinations of `feature` are accounted for by all possible combinations of the factor class variables.


```r
r1 <- nrow(dt[, .N, by=c("feature")])
```

```
## Error in nrow(dt[, .N, by = c("feature")]): objet '.N' introuvable
```

```r
r2 <- nrow(dt[, .N, by=c("featDomain", "featAcceleration", "featInstrument", "featJerk", "featMagnitude", "featVariable", "featAxis")])
```

```
## Error in nrow(dt[, .N, by = c("featDomain", "featAcceleration", "featInstrument", : objet '.N' introuvable
```

```r
r1 == r2
```

```
## Error in eval(expr, envir, enclos): objet 'r1' introuvable
```

Yes, I accounted for all possible combinations. `feature` is now redundant.



Create a tidy data set
----------------------

Create a data set with the average of each variable for each activity and each subject.


```r
setkey(dt, subject, activity, featDomain, featAcceleration, featInstrument, featJerk, featMagnitude, featVariable, featAxis)
```

```
## Error in eval(expr, envir, enclos): impossible de trouver la fonction "setkey"
```

```r
dtTidy <- dt[, list(count = .N, average = mean(value)), by=key(dt)]
```

```
## Error in eval(expr, envir, enclos): objet '.N' introuvable
```

Make codebook.


```r
knit("makeCodebook.Rmd", output="codebook.md", encoding="ISO8859-1", quiet=TRUE)
```

```
## Warning in readLines(if (is.character(input2)) {: impossible d'ouvrir le
## fichier 'makeCodebook.Rmd' : No such file or directory
```

```
## Error in readLines(if (is.character(input2)) {: impossible d'ouvrir la connexion
```

```r
markdownToHTML("codebook.md", "codebook.html")
```

```
## Warning in readLines(con): impossible d'ouvrir le fichier 'codebook.md' :
## No such file or directory
```

```
## Error in readLines(con): impossible d'ouvrir la connexion
```
