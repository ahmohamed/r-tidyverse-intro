---
title: "Loading data into R"
teaching: 20
exercises: 10
questions:
- "How do I read data into R?"
- "How do I assign variables?"
- "What is a data frame?"
- "How do I access subsets of a data frame?"
- "How do I calculate simple statistics like mean and median?"
objectives:
- "Read tabular data from a file into a program."
- "Assign values to variables."
- "Select individual values and subsections from data."
- "Perform operations on a data frame of data."
keypoints:
- "The function `dim` gives the dimensions of a data frame."
- "Use `object[x, y]` to select a single element from a data frame."
- "Use `from:to` to specify a sequence that includes the indices from `from` to `to`."
- "All the indexing and subsetting that works on data frames also works on vectors."
- "Use `mean`, `max`, `min` and `sd` to calculate simple statistics."
source: Rmd
---
One of R's most powerful features is its ability to deal with tabular data —
such as you may already have in a spreadsheet or a CSV file. 

The [course data]({{ page.root }}/data/r-novice-cancer.zip) contains a `.csv` file
called `brca-clinical-01.csv`, which you copied to the `data/` folder when we discussed managing projects in RStudio.


```r
PATIENT_ID,SEX,AGE,METASTASIS,OS_STATUS,OS_MONTHS
TCGA-A2-A0T2-01,FEMALE,66,M1,DECEASED,7.89
TCGA-A2-A04P-01,FEMALE,36,M0,DECEASED,17.97
TCGA-A1-A0SK-01,FEMALE,54,M0,DECEASED,31.77
TCGA-A2-A0CM-01,FEMALE,40,M0,DECEASED,24.77
```

We can view the contents of the file by selecting it from the "Files" window in RStudio, and selecting "View File".  This will display the contents of the file in a new window in RStudio.  We can see that the variables names are given in the first line of the file, and that the remaining lines contain the data itself.  Each observation is on a separate line, and variables are separated by commas. Note that viewing the file  _doesn't_ make its contents available to R; to do this we need to _import_ the data.

Let's make a new script for this episode, by choosing the menu options _File_, _New File_, _R Script_.

### Loading Data

Let's import the file called `brca-clinical-01.csv` into our R environment. To import the file, first we need to tell our computer where the file is. We do that by choosing a working directory, that is, a local directory on our computer containing the files we need. This is very important in R. If we forget this step we'll get an error message saying that the file does not exist. We can set the working directory using the function `setwd`. For this example, we change the path to our new directory at the desktop:


```r
setwd("~/Desktop/r-novice-cancer/")
```

Just like in the Unix Shell, we type the command and then press <kbd>Return</kbd> (or <kbd>Enter</kbd>).
Alternatively you can change the working directory using the RStudio GUI using the menu option `Session` -> `Set Working Directory` -> `Choose Directory...`

The data file is located in the directory `data` inside the working directory. Now we can load the data into R using `read.csv`:


```r
read.csv(file = "data/brca-clinical-01.csv")
```

The expression `read.csv(...)` is a [function call]({{ page.root }}/reference.html#function-call) that asks R to run the function `read.csv`.

`read.csv` has two [arguments]({{ page.root }}/reference.html#argument): the name of the file we want to read, and whether the first line of the file contains names for the columns of data.
The filename needs to be a character string (or [string]({{ page.root }}/reference.html#string) for short), so we put it in quotes. Assigning the second argument, `header`, to be `FALSE` indicates that the data file does not have column headers. We'll talk more about the value `FALSE`, and its converse `TRUE`, in lesson 04. In case of our `inflammation-01.csv` example, R auto-generates column names in the sequence `V1` (for "variable 1"), `V2`, and so on, until `V30`.


`read.csv` reads the file, but we can't use the data unless we assign it to a variable. Let's re-run `read.csv` and save its result into a variable called 'dat':


```r
dat <- read.csv(file = "data/brca-clinical-01.csv")
```

This statement doesn't produce any output because the assignment doesn't display anything.
If we want to check if our data has been loaded, we can print the variable's value by typing the name of the variable `dat`. However, for large data sets it is convenient to use the function `head` to display only the first few rows of data.


```r
head(dat)
```

```
##        PATIENT_ID    SEX AGE METASTASIS OS_STATUS OS_MONTHS
## 1 TCGA-A2-A0T2-01 FEMALE  66         M1  DECEASED      7.89
## 2 TCGA-A2-A04P-01 FEMALE  36         M0  DECEASED     17.97
## 3 TCGA-A1-A0SK-01 FEMALE  54         M0  DECEASED     31.77
## 4 TCGA-A2-A0CM-01 FEMALE  40         M0  DECEASED     24.77
## 5 TCGA-AR-A1AR-01 FEMALE  50         M0  DECEASED     17.18
## 6 TCGA-B6-A0WX-01 FEMALE  40         M0  DECEASED     21.45
```

### Manipulating Data

Now that our data are loaded into R, we can start doing things with them.
First, let's ask what type of thing `dat` is:


```r
class(dat)
```

```
## [1] "data.frame"
```

The output tells us that it's a data frame. Think of this structure as a spreadsheet in MS Excel that many of us are familiar with.
Data frames are very useful for storing data and you will use them frequently when programming in R.
A typical data frame of experimental data contains individual observations in rows and variables in columns.

We can see the shape, or [dimensions]({{ page.root }}/reference.html#dimensions-of-an-array), of the data frame with the function `dim`:


```r
dim(dat)
```

```
## [1] 20  6
```

This tells us that our data frame, `dat`, has 20 rows and 6 columns.

If we want to get a single value from the data frame, we can provide an [index]({{ page.root }}/reference.html#index) in square brackets. The first number specifies the row and the second the column:


```r
# first value in dat, row 1, column 1
dat[1, 1]
```

```
## [1] TCGA-A2-A0T2-01
## 20 Levels: TCGA-A1-A0SK-01 TCGA-A2-A04P-01 ... TCGA-BH-A1F0-01
```

```r
# middle value in dat, row 20, column 5
dat[20, 5]
```

```
## [1] LIVING
## Levels: DECEASED LIVING
```

The first value in a data frame index is the row, the second value is the column.
If we want to select more than one row or column, we can use the function `c`, which stands for **c**ombine.
For example, to pick columns 1, 3 and 5 from rows 10 and 20, we can do this:

```r
dat[c(10, 20), c(1, 3, 5)]
```

```
##         PATIENT_ID AGE OS_STATUS
## 10 TCGA-BH-A18Q-01  56  DECEASED
## 20 TCGA-A7-A0DA-01  62    LIVING
```
We frequently want to select contiguous rows or columns, such as the first ten rows, or columns 3 through 7. You can use `c` for this, but it's more convenient to use the `:` operator. This special function generates sequences of numbers:

```r
1:5
```

```
## [1] 1 2 3 4 5
```

```r
3:12
```

```
##  [1]  3  4  5  6  7  8  9 10 11 12
```
For example, we can select the first five columns of values for the first four rows like this:

```r
dat[1:4, 1:5]
```

```
##        PATIENT_ID    SEX AGE METASTASIS OS_STATUS
## 1 TCGA-A2-A0T2-01 FEMALE  66         M1  DECEASED
## 2 TCGA-A2-A04P-01 FEMALE  36         M0  DECEASED
## 3 TCGA-A1-A0SK-01 FEMALE  54         M0  DECEASED
## 4 TCGA-A2-A0CM-01 FEMALE  40         M0  DECEASED
```
or the first five columns of rows 5 to 10 like this:

```r
dat[5:10, 1:5]
```

```
##         PATIENT_ID    SEX AGE METASTASIS OS_STATUS
## 5  TCGA-AR-A1AR-01 FEMALE  50         M0  DECEASED
## 6  TCGA-B6-A0WX-01 FEMALE  40         M0  DECEASED
## 7  TCGA-BH-A1F0-01 FEMALE  80         M0  DECEASED
## 8  TCGA-B6-A0I6-01 FEMALE  49         M0  DECEASED
## 9  TCGA-BH-A18V-01 FEMALE  48         M0  DECEASED
## 10 TCGA-BH-A18Q-01 FEMALE  56         M0  DECEASED
```
If you want to select all rows or all columns, leave that index value empty. 

```r
# All columns from row 5
dat[5, ]
```

```
##        PATIENT_ID    SEX AGE METASTASIS OS_STATUS OS_MONTHS
## 5 TCGA-AR-A1AR-01 FEMALE  50         M0  DECEASED     17.18
```

```r
# All rows from column 4-6
dat[, 4:6]
```

```
##    METASTASIS OS_STATUS OS_MONTHS
## 1          M1  DECEASED      7.89
## 2          M0  DECEASED     17.97
## 3          M0  DECEASED     31.77
## 4          M0  DECEASED     24.77
## 5          M0  DECEASED     17.18
## 6          M0  DECEASED     21.45
## 7          M0  DECEASED     25.79
## 8          M0  DECEASED     32.76
## 9          M0  DECEASED     51.09
## 10         M0  DECEASED     55.59
## 11         M0  DECEASED     90.75
## 12         M0    LIVING      2.37
## 13         M0    LIVING      4.37
## 14         M0    LIVING      5.59
## 15         M0    LIVING      8.77
## 16         M0    LIVING      9.59
## 17         M0    LIVING      9.66
## 18         M0    LIVING     10.15
## 19         M0    LIVING     10.71
## 20         M0    LIVING     12.25
```
If you leave both index values empty (i.e., `dat[,]`), you get the entire data frame. 
> ## Addressing Columns by Name
>
> Columns can also be addressed by name, with either the `$` operator (ie. `dat$AGE`) or square brackets (ie. `dat[, 'AGE']`).
{: .callout}
Now let's perform some common mathematical operations to learn more about our patients data.
When analyzing data we often want to look at partial statistics, such as the average or maximum overall survival across patients.
One way to do this is to select the data we want to create a new temporary data frame, and then perform the calculation on this subset:

```r
# OS colum, all of the patients (rows)
os_data <- dat[, 6]
# max OS
max(os_data)
```

```
## [1] 90.75
```

R also has functions for other common calculations, e.g. finding the minimum, mean, median, and standard deviation of the data:

```r
# minimum OS
min(dat[, 6])
```

```
## [1] 2.37
```

```r
# mean OS
mean(dat[, 6])
```

```
## [1] 22.5235
```

```r
# median OS
median(dat[, 6])
```

```
## [1] 14.715
```

```r
# standard deviation of OS
sd(dat[, 6])
```

```
## [1] 21.73699
```

> ## Forcing Conversion
>
> Note that R may return an error when you attempt to perform similar calculations on 
> sliced *rows* of data frames. This is because some functions in R automatically convert 
> the object type to a numeric vector, while others do not (e.g. `max(dat[1, ])` works as 
> expected, while `mean(dat[1, ])` returns an error). You can fix this by including an 
> explicit call to `as.numeric()`, e.g. `mean(as.numeric(dat[1, ]))`. By contrast, 
> calculations on sliced *columns* always work as expected, since columns of data frames 
> are already defined as vectors.
{: .callout}


R also has a function that summaries the previous common calculations:

```r
# Summary function
summary(dat)
```

```
##            PATIENT_ID     SEX          AGE        METASTASIS    OS_STATUS 
##  TCGA-A1-A0SK-01: 1   FEMALE:20   Min.   :36.00   M0:19      DECEASED:11  
##  TCGA-A2-A04P-01: 1               1st Qu.:44.75   M1: 1      LIVING  : 9  
##  TCGA-A2-A0CM-01: 1               Median :52.00                           
##  TCGA-A2-A0T2-01: 1               Mean   :52.75                           
##  TCGA-A7-A0CE-01: 1               3rd Qu.:59.75                           
##  TCGA-A7-A0DA-01: 1               Max.   :80.00                           
##  (Other)        :14                                                       
##    OS_MONTHS     
##  Min.   : 2.370  
##  1st Qu.: 9.385  
##  Median :14.715  
##  Mean   :22.523  
##  3rd Qu.:27.285  
##  Max.   :90.750  
## 
```


