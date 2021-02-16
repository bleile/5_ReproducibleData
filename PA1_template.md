\#\#Reproducible Research Project 1 Assignment

### Instructions

*Introduction*

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
“quantified self” movement – a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

Dataset: [Activity monitoring
data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)
\[52K\]

The variables included in this dataset are:

-   steps: Number of steps taking in a 5-minute interval (missing values
    are coded as NA)
-   date: The date on which the measurement was taken in YYYY-MM-DD
    format
-   interval: Identifier for the 5-minute interval in which measurement
    was taken

The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this dataset.

***Review Criteria***

**Repo**

1.  Valid GitHub URL
2.  At least one commit beyond the original fork
3.  Valid SHA-1
4.  SHA-1 corresponds to a specific commit

**Commit containing full submission**

1.  Code for reading in the dataset and/or processing the data
2.  Histogram of the total number of steps taken each day
3.  Mean and median number of steps taken each day
4.  Time series plot of the average number of steps taken
5.  The 5-minute interval that, on average, contains the maximum number
    of steps
6.  Code to describe and show a strategy for imputing missing data
7.  Histogram of the total number of steps taken each day after missing
    values are imputed
8.  Panel plot comparing the average number of steps taken per 5-minute
    interval across weekdays and weekends
9.  All of the R code needed to reproduce the results (numbers, plots,
    etc.) in the report

***Assignment***

This assignment will be described in multiple parts. You will need to
write a report that answers the questions detailed below. Ultimately,
you will need to complete the entire assignment in a single R markdown
document that can be processed by knitr and be transformed into an HTML
file.

Throughout your report make sure you always include the code that you
used to generate the output you present. When writing code chunks in the
R markdown document, always use echo = TRUE so that someone else will be
able to read the code. This assignment will be evaluated via peer
assessment so it is essential that your peer evaluators be able to
review the code for your analysis.

For the plotting aspects of this assignment, feel free to use any
plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the GitHub repository created for this assignment. You will
submit this assignment by pushing your completed files into your forked
repository on GitHub. The assignment submission will consist of the URL
to your GitHub repository and the SHA-1 commit ID for your repository
state.

NOTE: The GitHub repository also contains the dataset for the assignment
so you do not have to download the data separately.

*Set Environment*

``` r
#setwd("~/Documents/GitHub/5_ReproducibleResearch/ReproducibleResearch_PeerAssessment1")
library(tidyverse, lib.loc = "/Library/Frameworks/R.framework/Versions/4.0/Resources/library")
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.0 ──

    ## ✓ ggplot2 3.3.3     ✓ purrr   0.3.4
    ## ✓ tibble  3.0.6     ✓ dplyr   1.0.4
    ## ✓ tidyr   1.1.2     ✓ stringr 1.4.0
    ## ✓ readr   1.4.0     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

``` r
library("data.table")
```

    ## 
    ## Attaching package: 'data.table'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     between, first, last

    ## The following object is masked from 'package:purrr':
    ## 
    ##     transpose

``` r
library("knitr")
knitr::opts_chunk$set(echo = TRUE)
library(data.table)
setwd("~")
```

*Loading and preprocessing the data*  
Show any code that is needed to

1.  Load the data (i.e. read.csv)

``` r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile = 'activity.zip', method = "curl")
unzip("activity.zip")
Activity <- data.table::fread(input = "activity.csv")
```

1.  Process/transform the data (if necessary) into a format suitable for
    your analysis

``` r
Activity[, date := as.POSIXct(date, format = "%Y-%m-%d")]
```

*What is mean total number of steps taken per day?*  
For this part of the assignment, you can ignore the missing values in
the dataset.

1.  Calculate the total number of steps taken per day

``` r
Steps <- Activity[, c(lapply(.SD, sum, na.rm = FALSE)), .SDcols = c("steps"), by = .(date)] 
```

1.  If you do not understand the difference between a histogram and a
    barplot, research the difference between them. Make a histogram of
    the total number of steps taken each day

``` r
ggplot(Steps, aes(x = steps)) + geom_histogram(fill = "black", binwidth = 1000) + labs(title = "Daily", x = "Steps", y = "Freq")
```

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](PA1_template_files/figure-markdown_github/unnamed-chunk-5-1.png) 1.
Calculate and report the mean and median of the total number of steps
taken per day

``` r
Steps[, .(Avg = mean(steps, na.rm = TRUE), Med = median(steps, na.rm = TRUE))]
```

    ##         Avg   Med
    ## 1: 10766.19 10765

*What is the average daily activity pattern*?

1.  Make a time series plot (i.e. type = “l”) of the 5-minute interval
    (x-axis) and the average number of steps taken, averaged across all
    days (y-axis)

``` r
Interval <- Activity[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)] 
ggplot(Interval, aes(x = interval , y = steps)) + geom_line(color="black", size=1) + labs(title = "Daily Avgerage", x = "Interval", y = "Steps")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-7-1.png) 1.
Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

``` r
Interval[steps == max(steps), .(max_interval = interval)]
```

    ##    max_interval
    ## 1:          835

*Imputing missing values*  
Note that there are a number of days/intervals where there are missing
values (coded as NA). The presence of missing days may introduce bias
into some calculations or summaries of the data.

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with NAs)

``` r
Activity[is.na(steps), .N ]
```

    ## [1] 2304

1.  Devise a strategy for filling in all of the missing values in the
    dataset. The strategy does not need to be sophisticated. For
    example, you could use the mean/median for that day, or the mean for
    that 5-minute interval, etc.

``` r
ActivityImputed <- Activity
ActivityImputed[is.na(steps), "steps"] <- Activity[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps")]
```

    ## Warning in `[<-.data.table`(`*tmp*`, is.na(steps), "steps", value =
    ## structure(list(: 37.382600 (type 'double') at RHS position 1 truncated
    ## (precision lost) when assigning to type 'integer' (column 1 named 'steps')

1.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

``` r
data.table::fwrite(x = ActivityImputed, file = "ActivityImputed.csv", quote = FALSE)
```

1.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day. Do these values differ from the estimates from the first
    part of the assignment? What is the impact of imputing missing data
    on the estimates of the total daily number of steps?

``` r
StepsImputed <- ActivityImputed[, c(lapply(.SD, sum)), .SDcols = c("steps"), by = .(date)] 
ggplot(StepsImputed, aes(x = steps)) + geom_histogram(fill = "black", binwidth = 1000) + labs(title = "Daily", x = "Steps", y = "Freq")
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-12-1.png)
*Data Source Steps \| Mean \| Median* Activity \| 10765 \| 10765
Activity Imputed \| 9354 \| 10395

*Are there differences in activity patterns between weekdays and
weekends?*  
For this part the weekdays() function may be of some help here. Use the
dataset with the filled-in missing values for this part.

1.  Create a new factor variable in the dataset with two levels –
    “weekday” and “weekend” indicating whether a given date is a weekday
    or weekend day.

``` r
Activity[, `DoW`:= weekdays(x = date)]
Activity[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `DoW`), "weekday or weekend"] <- "weekday"
Activity[grepl(pattern = "Saturday|Sunday", x = `DoW`), "weekday or weekend"] <- "weekend"
Activity[, `weekday or weekend` := as.factor(`weekday or weekend`)]
```

1.  Make a panel plot containing a time series plot (i.e. type = “l”) of
    the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).
    See the README file in the GitHub repository to see an example of
    what this plot should look like using simulated data.

``` r
Activity[is.na(steps), "steps"] <- Activity[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps")]
```

    ## Warning in `[<-.data.table`(`*tmp*`, is.na(steps), "steps", value =
    ## structure(list(: 37.382600 (type 'double') at RHS position 1 truncated
    ## (precision lost) when assigning to type 'integer' (column 1 named 'steps')

``` r
Interval <- Activity[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday or weekend`)] 
ggplot(Interval , aes(x = interval , y = steps, color=`weekday or weekend`)) + geom_line() + labs(title = "Avg. Steps by Weekday or Weekend", x = "Interval", y = "Steps") + facet_wrap(~`weekday or weekend` , ncol = 1, nrow=2)
```

![](PA1_template_files/figure-markdown_github/unnamed-chunk-14-1.png)

***Submitting the Assignment***  
To submit the assignment:

1.  Commit your completed PA1_template.Rmd file to the master branch of
    your git repository (you should already be on the master branch
    unless you created new ones)
2.  Commit your PA1_template.md and PA1_template.html files produced by
    processing your R markdown file with knit2html() function in R (from
    the knitr package) by running the function from the console.
3.  If your document has figures included (it should) then they should
    have been placed in the figure/ directory by default (unless you
    overrided the default). Add and commit the figure/ directory to your
    git repository so that the figures appear in the markdown file when
    it displays on github.
4.  Push your master branch to GitHub.
5.  Submit the URL to your GitHub repository for this assignment on the
    course web site.

In addition to submitting the URL for your GitHub repository, you will
need to submit the 40 character SHA-1 hash (as string of numbers from
0-9 and letters from a-f) that identifies the repository commit that
contains the version of the files you want to submit. You can do this in
GitHub by doing the following:

1.  Going to your GitHub repository web page for this assignment
2.  Click on the “?? commits” link where ?? is the number of commits you
    have in the repository. For example, if you made a total of 10
    commits to this repository, the link should say “10 commits”.
3.  You will see a list of commits that you have made to this
    repository. The most recent commit is at the very top. If this
    represents the version of the files you want to submit, then just
    click the “copy to clipboard” button on the right hand side that
    should appear when you hover over the SHA-1 hash. Paste this SHA-1
    hash into the course web site when you submit your assignment. If
    you don’t want to use the most recent commit, then go down and find
    the commit you want and copy the SHA-1 hash.

A valid submission will look something like (this is just an example!)

    https://github.com/rdpeng/RepData_PeerAssessment17c376cc54

    47f11537f8740af8e07d6facc3d9645