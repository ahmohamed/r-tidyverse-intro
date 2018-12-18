---
title: Manipulating tibbles with dplyr
teaching: 40
exercises: 15
questions:
- "How can I manipulate tibbles without repeating myself?"
objectives:
- " To be able to use the six main `dplyr` data manipulation 'verbs' with pipes."
keypoints:
- "Use the `dplyr` package to manipulate tibbles."
- "Use `select()` to choose variables from a tibbles."
- "Use `filter()` to choose data based on values."
- "Use `group_by()` and `summarize()` to work with subsets of data."
- "Use `mutate()` to create new variables."
source: Rmd
---



In the previous episode we used the `readr` package to load tabular data into a tibble within R.  The `readr` package is part of a family of packages known as the   [tidyverse](http://tidyverse.org/).  The tidyverse packages are designed to work well together; they provide a modern and streamlined approach to data-analysis, and deal with some of the idiosyncrasies of base R.


This loads the most commonly used packages in the tidyverse; we used `readr` in the previous episode.  We will cover all of the other main packages, with the exception of `purrr` in this course. There are other [libraries included](https://github.com/tidyverse/tidyverse) but these are less widely used, and must be loaded manually if they are required; these aren't covered in this course. 

As we did with the [previous episode]({{ page.root }}/03-tibbles) we use the `read_csv()` function to load data from a comma separated file. Let's make a new script (using the file menu), and load the tidyverse: (in the previous episode we only loaded `readr`; since we'll be using several packages in the tidyverse, we load them all).


~~~
library("tidyverse")
~~~
{: .language-r}



~~~
── Attaching packages ────────────────────────────────── tidyverse 1.2.1 ──
~~~
{: .output}



~~~
✔ ggplot2 3.1.0     ✔ purrr   0.2.5
✔ tibble  1.4.2     ✔ dplyr   0.7.8
✔ tidyr   0.8.2     ✔ stringr 1.3.1
✔ readr   1.2.1     ✔ forcats 0.3.0
~~~
{: .output}



~~~
── Conflicts ───────────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
~~~
{: .output}



~~~
brca <- read_csv("data/brca-clinical-data.csv")
~~~
{: .language-r}



~~~
Parsed with column specification:
cols(
  .default = col_character(),
  SIGCLUST_UNSUPERVISED_MRNA = col_double(),
  SIGCLUST_INTRINSIC_MRNA = col_double(),
  MIRNA_CLUSTER = col_double(),
  METHYLATION_CLUSTER = col_double(),
  CN_CLUSTER = col_double(),
  INTEGRATED_CLUSTERS_WITH_PAM50 = col_double(),
  INTEGRATED_CLUSTERS_NO_EXP = col_double(),
  INTEGRATED_CLUSTERS_UNSUP_EXP = col_double(),
  AGE = col_double(),
  OS_MONTHS = col_double()
)
~~~
{: .output}



~~~
See spec(...) for full column specifications.
~~~
{: .output}

As we discussed in the [previous episode]({{ page.root }}/03-tibbles), variables in R can be character, integer, double, etc.   A tibble (and R's built in equivalent; the data-frame) require that all the values in a particular column have the same data type.  The `read_csv()` function will attempt to infer the data type of each column, and prints the column types it has guessed to the screen.  If the wrong column types have been generated, you can pass the `col_types=` option to `read_csv()`.  

For example, if we wanted to control column types, we would use:


~~~
brca <- read_csv("data/brca-clinical-data.csv", col_types = cols(
    AGE = col_integer(),
    SEX = col_factor(),
    METASTASIS = col_factor(),
    OS_STATUS = col_factor()
))
~~~
{: .language-r}

> ## Setting column types
> 
> Try reading a file using the `read_csv()` defaults (i.e. guessing column types).
> If this fails you can cut and paste the guessed column specification, and modify
> this with the correct column types.  It is good practice to do this anyway; it makes
> the data types of your columns explicit, and will help protect you if the format 
> of your data changes.
{: .callout}

## Manipulating tibbles 

Manipulation of tibbles means many things to many researchers. We often
select only certain observations (rows) or variables (columns). We often group the
data by a certain variable(s), or calculate summary statistics.

## The `dplyr` package

The  [`dplyr`](https://cran.r-project.org/web/packages/dplyr/dplyr.pdf)
package is part of the tidyverse.  It provides a number of very useful functions for manipulating tibbles (and their base-R cousin, the `data.frame`) 
in a way that will reduce repetition, reduce the probability of making
errors, and probably even save you some typing. 

We will cover:

1. selecting variables with `select()`
2. subsetting observations with `filter()`
3. grouping observations with `group_by()`
4. generating summary statistics using `summarize()`
5. generating new variables using `mutate()`
6. Sorting tibbles using `arrange()`
7. chaining operations together using pipes `%>%` 

## Using `select()`

If, for example, we wanted to move forward with only a few of the variables in
our tibble we use the `select()` function. This will keep only the
variables you select.


~~~
brca_molecular <- select(brca, PATIENT_ID, ER_STATUS, PR_STATUS, HER2_STATUS, AGE, SEX, OS_MONTHS)
print(brca_molecular)
~~~
{: .language-r}



~~~
# A tibble: 894 x 7
   PATIENT_ID    ER_STATUS PR_STATUS    HER2_STATUS    AGE SEX   OS_MONTHS
   <chr>         <chr>     <chr>        <chr>        <int> <fct>     <dbl>
 1 TCGA-A2-A0T2… Negative  Negative     Negative        66 FEMA…      7.89
 2 TCGA-A2-A04P… Negative  Negative     Negative        36 FEMA…     18.0 
 3 TCGA-A1-A0SK… Negative  Negative     Negative        54 FEMA…     31.8 
 4 TCGA-A2-A0CM… Negative  Negative     Negative        40 FEMA…     24.8 
 5 TCGA-AR-A1AR… Negative  Negative     Negative        50 FEMA…     17.2 
 6 TCGA-B6-A0WX… Negative  Negative     Negative        40 FEMA…     21.4 
 7 TCGA-BH-A1F0… Negative  Indetermina… Negative        80 FEMA…     25.8 
 8 TCGA-B6-A0I6… Negative  Negative     Not Availab…    49 FEMA…     32.8 
 9 TCGA-BH-A18V… Negative  Negative     Negative        48 FEMA…     51.1 
10 TCGA-BH-A18Q… Negative  Negative     Negative        56 FEMA…     55.6 
# ... with 884 more rows
~~~
{: .output}

Select will select _columns_ of data.  What if we want to select rows that meet certain criteria?  

## Using `filter()`

The `filter()` function is used to select rows of data.  For example, to select only HER2 positive patients:


~~~
brca_her2 <- filter(brca, HER2_STATUS=="Positive")
print(brca_her2)
~~~
{: .language-r}



~~~
# A tibble: 114 x 30
   SAMPLE_ID PATIENT_ID ER_STATUS PR_STATUS HER2_STATUS TUMOR_STAGE
   <chr>     <chr>      <chr>     <chr>     <chr>       <chr>      
 1 TCGA-A8-… TCGA-A8-A… Positive  Positive  Positive    T2         
 2 TCGA-E2-… TCGA-E2-A… Positive  Positive  Positive    T2         
 3 TCGA-B6-… TCGA-B6-A… Indeterm… Positive  Positive    T3         
 4 TCGA-BH-… TCGA-BH-A… Indeterm… Negative  Positive    T2         
 5 TCGA-AR-… TCGA-AR-A… Positive  Positive  Positive    T2         
 6 TCGA-BH-… TCGA-BH-A… Positive  Positive  Positive    T2         
 7 TCGA-A2-… TCGA-A2-A… Negative  Negative  Positive    T3         
 8 TCGA-BH-… TCGA-BH-A… Positive  Negative  Positive    T1         
 9 TCGA-BH-… TCGA-BH-A… Negative  Negative  Positive    T3         
10 TCGA-A2-… TCGA-A2-A… Negative  Negative  Positive    T2         
# ... with 104 more rows, and 24 more variables: TUMOR_T1_CODED <chr>,
#   NODES <chr>, NODE_CODED <chr>, METASTASIS_CODED <chr>,
#   CONVERTED_STAGE <chr>, SURVIVAL_DATA_FORM <chr>, PAM50_SUBTYPE <chr>,
#   SIGCLUST_UNSUPERVISED_MRNA <dbl>, SIGCLUST_INTRINSIC_MRNA <dbl>,
#   MIRNA_CLUSTER <dbl>, METHYLATION_CLUSTER <dbl>, RPPA_CLUSTER <chr>,
#   CN_CLUSTER <dbl>, INTEGRATED_CLUSTERS_WITH_PAM50 <dbl>,
#   INTEGRATED_CLUSTERS_NO_EXP <dbl>, INTEGRATED_CLUSTERS_UNSUP_EXP <dbl>,
#   CANCER_TYPE_DETAILED <chr>, ONCOTREE_CODE <chr>, CANCER_TYPE <chr>,
#   SEX <fct>, AGE <int>, METASTASIS <fct>, OS_STATUS <fct>,
#   OS_MONTHS <dbl>
~~~
{: .output}

Only rows of the data where the condition (i.e. `HER2_STATUS=="Positive"`) is `TRUE` are kept.

## Using pipes and dplyr

We've now seen how to choose certain columns of data (using `select()`) and certain rows of data (using `filter()`).  In an analysis we often want to do both of these things (and many other things, like calculating summary statistics, which we'll come to shortly).    How do we combine these?

There are several ways of doing this; the method we will learn about today is using _pipes_.  

The pipe operator `%>%` lets us pipe the output of one command into the next.   This allows us to build up a data-processing pipeline.  This approach has several advantages:

* We can build the pipeline piecemeal — building the pipeline step-by-step is easier than trying to 
perform a complex series of operations in one go
* It is easy to modify and reuse the pipeline
* We don't have to make temporary tibbles as the analysis progresses.

> ## Pipelines and the shell
>
> If you're familiar with the Unix shell, you may already have used pipes to
> pass the output from one command to the next.  The concept is the same, except
> the shell uses the `|` character rather than R's pipe operator `%>%`
{: .callout}


> ## Keyboard shortcuts and getting help
> 
> The pipe operator can be tedious to type.  In Rstudio pressing <kbd>Ctrl</kbd> + <kbd>Shift</kbd>+<kbd>M</kbd> under
> Windows / Linux will insert the pipe operator.  On the mac, use <kbd>&#8984;</kbd> + <kbd>Shift</kbd>+<kbd>M</kbd>.
>
> We can use tab completion to complete variable names when entering commands.
> This saves typing and reduces the risk of error.
> 
> RStudio includes a helpful "cheat sheet", which summarises the main functionality
> and syntax of `dplyr`.  This can be accessed via the
> help menu --> cheatsheets --> data transformation with dplyr. 
>
{: .callout}

Let's rewrite the select command example using the pipe operator:


~~~
brca_molecular <- brca %>% select(PATIENT_ID, ER_STATUS, PR_STATUS, HER2_STATUS, AGE, SEX, OS_MONTHS)
print(brca_molecular)
~~~
{: .language-r}



~~~
# A tibble: 894 x 7
   PATIENT_ID    ER_STATUS PR_STATUS    HER2_STATUS    AGE SEX   OS_MONTHS
   <chr>         <chr>     <chr>        <chr>        <int> <fct>     <dbl>
 1 TCGA-A2-A0T2… Negative  Negative     Negative        66 FEMA…      7.89
 2 TCGA-A2-A04P… Negative  Negative     Negative        36 FEMA…     18.0 
 3 TCGA-A1-A0SK… Negative  Negative     Negative        54 FEMA…     31.8 
 4 TCGA-A2-A0CM… Negative  Negative     Negative        40 FEMA…     24.8 
 5 TCGA-AR-A1AR… Negative  Negative     Negative        50 FEMA…     17.2 
 6 TCGA-B6-A0WX… Negative  Negative     Negative        40 FEMA…     21.4 
 7 TCGA-BH-A1F0… Negative  Indetermina… Negative        80 FEMA…     25.8 
 8 TCGA-B6-A0I6… Negative  Negative     Not Availab…    49 FEMA…     32.8 
 9 TCGA-BH-A18V… Negative  Negative     Negative        48 FEMA…     51.1 
10 TCGA-BH-A18Q… Negative  Negative     Negative        56 FEMA…     55.6 
# ... with 884 more rows
~~~
{: .output}

To help you understand why we wrote that in that way, let's walk through it step
by step. First we summon the `brca` tibble and pass it on, using the pipe
symbol `%>%`, to the next step, which is the `select()` function. In this case
we don't specify which data object we use in the `select()` function since in
gets that from the previous pipe. 

What if we wanted to combine this with the filter example? I.e. we want to select year, country and GDP per capita, but only for countries in Europe?  We can join these two operations using a pipe; feeding the output of one command directly into the next:



~~~
brca_molecular_her2 <- brca %>% 
  filter(HER2_STATUS=="Positive") %>% 
  select(PATIENT_ID, ER_STATUS, PR_STATUS, HER2_STATUS, AGE, SEX, OS_MONTHS)
print(brca_molecular_her2)
~~~
{: .language-r}



~~~
# A tibble: 114 x 7
   PATIENT_ID     ER_STATUS    PR_STATUS HER2_STATUS   AGE SEX   OS_MONTHS
   <chr>          <chr>        <chr>     <chr>       <int> <fct>     <dbl>
 1 TCGA-A8-A08H-… Positive     Positive  Positive       66 FEMA…       0  
 2 TCGA-E2-A14Y-… Positive     Positive  Positive       35 FEMA…      22.6
 3 TCGA-B6-A0I9-… Indetermina… Positive  Positive       62 FEMA…      12.1
 4 TCGA-BH-A18R-… Indetermina… Negative  Positive       50 FEMA…      37.5
 5 TCGA-AR-A1AT-… Positive     Positive  Positive       62 FEMA…      41.8
 6 TCGA-BH-A0DZ-… Positive     Positive  Positive       43 FEMA…      16.3
 7 TCGA-A2-A0T1-… Negative     Negative  Positive       55 FEMA…      17.1
 8 TCGA-BH-A0AW-… Positive     Negative  Positive       56 FEMA…      20.4
 9 TCGA-BH-A0EE-… Negative     Negative  Positive       68 FEMA…      31.0
10 TCGA-A2-A0D1-… Negative     Negative  Positive       76 FEMA…      34.5
# ... with 104 more rows
~~~
{: .output}


What about if we wanted to match more than one item?  To do this we use the `%in%` operator:


~~~
brca_molecular_her2 <- brca %>% 
  filter(HER2_STATUS %in% c("Positive", "Negative")) %>% 
  select(PATIENT_ID, ER_STATUS, PR_STATUS, HER2_STATUS, AGE, SEX, OS_MONTHS)
  
print(brca_molecular_her2)
~~~
{: .language-r}



~~~
# A tibble: 766 x 7
   PATIENT_ID     ER_STATUS PR_STATUS    HER2_STATUS   AGE SEX   OS_MONTHS
   <chr>          <chr>     <chr>        <chr>       <int> <fct>     <dbl>
 1 TCGA-A2-A0T2-… Negative  Negative     Negative       66 FEMA…      7.89
 2 TCGA-A2-A04P-… Negative  Negative     Negative       36 FEMA…     18.0 
 3 TCGA-A1-A0SK-… Negative  Negative     Negative       54 FEMA…     31.8 
 4 TCGA-A2-A0CM-… Negative  Negative     Negative       40 FEMA…     24.8 
 5 TCGA-AR-A1AR-… Negative  Negative     Negative       50 FEMA…     17.2 
 6 TCGA-B6-A0WX-… Negative  Negative     Negative       40 FEMA…     21.4 
 7 TCGA-BH-A1F0-… Negative  Indetermina… Negative       80 FEMA…     25.8 
 8 TCGA-BH-A18V-… Negative  Negative     Negative       48 FEMA…     51.1 
 9 TCGA-BH-A18Q-… Negative  Negative     Negative       56 FEMA…     55.6 
10 TCGA-BH-A18K-… Positive  Positive     Negative       46 FEMA…     90.8 
# ... with 756 more rows
~~~
{: .output}


> ## Another way of thinking about pipes
>
> It might be useful to think of the statement
> 
> ~~~
> brca_molecular_her2 <- brca %>% 
>   filter(HER2_STATUS=="Positive") %>% 
>   select(PATIENT_ID, ER_STATUS, PR_STATUS, HER2_STATUS, AGE, SEX, OS_MONTHS)
> ~~~
> {: .language-r}
>  as a sentence, which we can read as
> "take the brca data *and then* `filter` records where HER2_STATUS == Positive
> *and then* `select` the columns.
> 
> We can think of the `filter()` and `select()` functions as verbs in the sentence; 
> they do things to the data flowing through the pipeline.  
>
{: .callout}

> ## Splitting your commands over multiple lines
> 
> It's generally a good idea to put one command per line when
> writing your analyses.  This makes them easier to read.   When
> doing this, it's important that the `%>%` goes at the _end_ of the
> line, as in the example above.  If we put it at the beginning of a line, e.g.:
> 
> 
> ~~~
> brca_wrong <- brca
> %>% filter(HER2_STATUS=="Positive")
> ~~~
> {: .language-r}
> 
> 
> 
> ~~~
> Error: <text>:2:1: unexpected SPECIAL
> 1: brca_wrong <- brca
> 2: %>%
>    ^
> ~~~
> {: .error}
> 
> the first line makes a valid R command.  R will then treat the next line 
> as a new command, which won't work.
{: .callout}


> ## Challenge 1
>
> Write a single command (which can span multiple lines and includes pipes) that
> will produce a tibble that has the values of `OS_STATUS`, `AGE`
> and `SEX`, for the patients that are ER-positive.  How many rows does your tibble  
> have? (You can use the `nrow()` function to find out how many rows are in a tibble.)
>
> > ## Solution to Challenge 1
> >
> >~~~
> >brca_er <- brca %>% 
> >  filter(ER_STATUS=="Positive") %>% 
> >  select(OS_STATUS, AGE, SEX)
> > nrow(brca_er)
> >~~~
> >{: .language-r}
> >
> >
> >
> >~~~
> >[1] 601
> >~~~
> >{: .output}
> > As with last time, first we pass the brca tibble to the `filter()`
> > function, then we pass the filtered version of the brca tibble  to the
> > `select()` function. **Note:** The order of operations is very important in this
> > case. If we used 'select' first, filter would not be able to find the variable
> > continent since we would have removed it in the previous step.
> {: .solution}
{: .challenge}


## Sorting tibbles

The `arrange()` function will sort a tibble by one or more of the variables in it:


~~~
brca_molecular_her2 %>%
  filter(HER2_STATUS=="Positive", ER_STATUS=="Positive") %>% 
  arrange(OS_MONTHS)
~~~
{: .language-r}



~~~
# A tibble: 79 x 7
   PATIENT_ID      ER_STATUS PR_STATUS HER2_STATUS   AGE SEX    OS_MONTHS
   <chr>           <chr>     <chr>     <chr>       <int> <fct>      <dbl>
 1 TCGA-A8-A08H-01 Positive  Positive  Positive       66 FEMALE      0   
 2 TCGA-A8-A09G-01 Positive  Negative  Positive       79 FEMALE      0   
 3 TCGA-C8-A12T-01 Positive  Positive  Positive       43 FEMALE      0   
 4 TCGA-C8-A132-01 Positive  Positive  Positive       56 FEMALE      0   
 5 TCGA-D8-A1JB-01 Positive  Positive  Positive       54 FEMALE      0   
 6 TCGA-E9-A1N6-01 Positive  Positive  Positive       52 FEMALE      0   
 7 TCGA-E9-A22D-01 Positive  Positive  Positive       38 FEMALE      0   
 8 TCGA-C8-A1HL-01 Positive  Negative  Positive       38 FEMALE      0.07
 9 TCGA-C8-A138-01 Positive  Negative  Positive       54 FEMALE      0.23
10 TCGA-C8-A26W-01 Positive  Positive  Positive       58 FEMALE      0.23
# ... with 69 more rows
~~~
{: .output}
We can use the `desc()` function to sort a variable in reverse order:


~~~
brca_molecular_her2 %>%
  filter(HER2_STATUS=="Positive", ER_STATUS=="Positive") %>% 
  arrange( desc(OS_MONTHS) )
~~~
{: .language-r}



~~~
# A tibble: 79 x 7
   PATIENT_ID      ER_STATUS PR_STATUS HER2_STATUS   AGE SEX    OS_MONTHS
   <chr>           <chr>     <chr>     <chr>       <int> <fct>      <dbl>
 1 TCGA-B6-A0RH-01 Positive  Positive  Positive       51 FEMALE     189. 
 2 TCGA-AQ-A04L-01 Positive  Negative  Positive       48 FEMALE     110. 
 3 TCGA-BH-A18M-01 Positive  Positive  Positive       39 FEMALE      72.5
 4 TCGA-AO-A12C-01 Positive  Positive  Positive       42 FEMALE      65.5
 5 TCGA-AO-A0JM-01 Positive  Positive  Positive       40 FEMALE      60.0
 6 TCGA-BH-A0B7-01 Positive  Positive  Positive       42 FEMALE      56.9
 7 TCGA-A2-A0CX-01 Positive  Negative  Positive       52 FEMALE      56.8
 8 TCGA-AR-A250-01 Positive  Negative  Positive       58 FEMALE      56.1
 9 TCGA-A2-A04X-01 Positive  Positive  Positive       34 FEMALE      55.4
10 TCGA-A8-A076-01 Positive  Positive  Positive       66 FEMALE      53.9
# ... with 69 more rows
~~~
{: .output}

## Generating new variables

The `mutate()` function lets us add new variables to our tibble.  It will often be the case that these are variables we _derive_ from existing variables in the data-frame. 

As an example, the brca data contains the overall survival of each patient, and their age.  We can use this to calculate the Age-relative OS:


~~~
brca_os_age <- brca_molecular_her2 %>% 
  mutate(OS_AGE = OS_MONTHS/AGE)
~~~
{: .language-r}

We can also use functions within mutate to generate new variables.  For example, to take the log of `OS_MONTHS` we could use:


~~~
brca_molecular_her2 %>% 
  mutate(logOS = log(OS_MONTHS))
~~~
{: .language-r}



~~~
# A tibble: 766 x 8
   PATIENT_ID  ER_STATUS PR_STATUS HER2_STATUS   AGE SEX   OS_MONTHS logOS
   <chr>       <chr>     <chr>     <chr>       <int> <fct>     <dbl> <dbl>
 1 TCGA-A2-A0… Negative  Negative  Negative       66 FEMA…      7.89  2.07
 2 TCGA-A2-A0… Negative  Negative  Negative       36 FEMA…     18.0   2.89
 3 TCGA-A1-A0… Negative  Negative  Negative       54 FEMA…     31.8   3.46
 4 TCGA-A2-A0… Negative  Negative  Negative       40 FEMA…     24.8   3.21
 5 TCGA-AR-A1… Negative  Negative  Negative       50 FEMA…     17.2   2.84
 6 TCGA-B6-A0… Negative  Negative  Negative       40 FEMA…     21.4   3.07
 7 TCGA-BH-A1… Negative  Indeterm… Negative       80 FEMA…     25.8   3.25
 8 TCGA-BH-A1… Negative  Negative  Negative       48 FEMA…     51.1   3.93
 9 TCGA-BH-A1… Negative  Negative  Negative       56 FEMA…     55.6   4.02
10 TCGA-BH-A1… Positive  Positive  Negative       46 FEMA…     90.8   4.51
# ... with 756 more rows
~~~
{: .output}

The dplyr cheat sheet contains many useful functions which can be used with dplyr.  This can be found in the help menu of RStudio. You will use one of these functions in the next challenge.

> ## Challenge 2
> 
> Create a tibble containing all ER- and her2-positive patients, OS status and OS in months
> and the rank of patients according to their OS. (note that ranking the patients _will not_ sort the table; the row order will be unchanged.  You can use the `arrange()` function to sort the table).
>
> Hint: First `filter()` to get the rows you want, and then use `mutate()` to create a new variable with the rank in it.  The cheat-sheet contains useful
> functions you can use when you make new variables (the cheat-sheets can be found in the help menu in RStudio).  
> You can use `min_rank` function to rank in this example.
>
> Can you reverse the ranking order
> so that the country with the longest life expectancy gets the lowest rank?
> Hint: This is similar to sorting in reverse order.
>
> > ## Solution to challenge 2
> > 
> > ~~~
> > brca_os_rank <- brca %>% 
> >   filter(HER2_STATUS=="Positive", ER_STATUS=="Positive") %>% 
> >   select(PATIENT_ID, HER2_STATUS, OS_MONTHS)
> >   mutate(os_rank = min_rank(OS_MONTHS))
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > Error in rank(x, ties.method = "min", na.last = "keep"): object 'OS_MONTHS' not found
> > ~~~
> > {: .error}
> > 
> > 
> > 
> > ~~~
> > print(brca_os_rank, n=100)
> > ~~~
> > {: .language-r}
> > 
> > 
> > 
> > ~~~
> > # A tibble: 79 x 3
> >    PATIENT_ID      HER2_STATUS OS_MONTHS
> >    <chr>           <chr>           <dbl>
> >  1 TCGA-A8-A08H-01 Positive         0   
> >  2 TCGA-E2-A14Y-01 Positive        22.6 
> >  3 TCGA-AR-A1AT-01 Positive        41.8 
> >  4 TCGA-BH-A0DZ-01 Positive        16.3 
> >  5 TCGA-BH-A0AW-01 Positive        20.4 
> >  6 TCGA-A2-A04X-01 Positive        55.4 
> >  7 TCGA-A2-A0CX-01 Positive        56.8 
> >  8 TCGA-B6-A0RH-01 Positive       189.  
> >  9 TCGA-A8-A076-01 Positive        53.9 
> > 10 TCGA-A8-A07B-01 Positive        33.0 
> > 11 TCGA-A8-A07I-01 Positive        14   
> > 12 TCGA-A8-A08B-01 Positive        23.1 
> > 13 TCGA-A8-A09G-01 Positive         0   
> > 14 TCGA-AR-A0TX-01 Positive         0.49
> > 15 TCGA-BH-A0B7-01 Positive        56.9 
> > 16 TCGA-C8-A12T-01 Positive         0   
> > 17 TCGA-C8-A138-01 Positive         0.23
> > 18 TCGA-E2-A14V-01 Positive        24.6 
> > 19 TCGA-E2-A152-01 Positive        19.3 
> > 20 TCGA-BH-A18M-01 Positive        72.5 
> > 21 TCGA-BH-A0B4-01 Positive        39.1 
> > 22 TCGA-A2-A0SY-01 Positive        44.3 
> > 23 TCGA-AO-A12C-01 Positive        65.5 
> > 24 TCGA-AQ-A04L-01 Positive       110.  
> > 25 TCGA-A8-A07P-01 Positive        11.0 
> > 26 TCGA-A8-A099-01 Positive         9.99
> > 27 TCGA-AN-A0FD-01 Positive         6.41
> > 28 TCGA-AN-A0FT-01 Positive         6.01
> > 29 TCGA-AR-A1AX-01 Positive        36.2 
> > 30 TCGA-C8-A132-01 Positive         0   
> > 31 TCGA-E2-A15E-01 Positive        17.0 
> > 32 TCGA-E2-A15H-01 Positive         1.38
> > 33 TCGA-E2-A1B1-01 Positive        44.7 
> > 34 TCGA-A8-A06X-01 Positive        31.0 
> > 35 TCGA-BH-A18U-01 Positive        51.4 
> > 36 TCGA-D8-A140-01 Positive        13.2 
> > 37 TCGA-AQ-A04H-01 Positive        14.4 
> > 38 TCGA-A2-A0YG-01 Positive        21.8 
> > 39 TCGA-A2-A0EY-01 Positive        24.2 
> > 40 TCGA-BH-A0C0-01 Positive        41.7 
> > 41 TCGA-AO-A0JM-01 Positive        60.0 
> > 42 TCGA-A1-A0SM-01 Positive         7.95
> > 43 TCGA-A8-A08G-01 Positive        19.9 
> > 44 TCGA-A8-A08P-01 Positive        31.0 
> > 45 TCGA-A8-A097-01 Positive        12.0 
> > 46 TCGA-A8-A09I-01 Positive        33.0 
> > 47 TCGA-A8-A09N-01 Positive         1.02
> > 48 TCGA-AN-A0AJ-01 Positive         7.98
> > 49 TCGA-AR-A0TQ-01 Positive        49.3 
> > 50 TCGA-BH-A0C7-01 Positive        42.9 
> > 51 TCGA-BH-A0DD-01 Positive        45.8 
> > 52 TCGA-C8-A1HL-01 Positive         0.07
> > 53 TCGA-E2-A14W-01 Positive        27.4 
> > 54 TCGA-AQ-A0Y5-01 Positive         5.29
> > 55 TCGA-BH-A1F2-01 Positive        31.5 
> > 56 TCGA-D8-A1XS-01 Positive         0.59
> > 57 TCGA-A1-A0SN-01 Positive        39.3 
> > 58 TCGA-A7-A26H-01 Positive         2.1 
> > 59 TCGA-A8-A08S-01 Positive        19.1 
> > 60 TCGA-AC-A23C-01 Positive         0.95
> > 61 TCGA-AC-A23H-01 Positive         2.66
> > 62 TCGA-AQ-A1H2-01 Positive         6.47
> > 63 TCGA-AR-A250-01 Positive        56.1 
> > 64 TCGA-AR-A254-01 Positive        39.8 
> > 65 TCGA-AR-A255-01 Positive        34.8 
> > 66 TCGA-C8-A26W-01 Positive         0.23
> > 67 TCGA-D8-A1J9-01 Positive         8.15
> > 68 TCGA-D8-A1JB-01 Positive         0   
> > 69 TCGA-D8-A1X5-01 Positive         5.62
> > 70 TCGA-D8-A1XJ-01 Positive        11.6 
> > 71 TCGA-D8-A1XY-01 Positive         2.66
> > 72 TCGA-D8-A27N-01 Positive         4.76
> > 73 TCGA-D8-A27W-01 Positive         0.49
> > 74 TCGA-E9-A1N6-01 Positive         0   
> > 75 TCGA-E9-A22D-01 Positive         0   
> > 76 TCGA-EW-A1IW-01 Positive         8.28
> > 77 TCGA-EW-A1J3-01 Positive         8.28
> > 78 TCGA-EW-A1OZ-01 Positive        30.9 
> > 79 TCGA-EW-A1PD-01 Positive         5.72
> > ~~~
> > {: .output}
> > 
> > To reverse the order of the ranking, use the `desc` function, i.e.
> > `mutate(rank = min_rank(desc(OS_MONTHS)))`.
> > 
> {: .solution}
{: .challenge}

## Calculating summary statistics

We often wish to calculate a summary statistic (the mean, standard deviation, etc.)
for a variable.  We frequently want to calculate a separate summary statistic for several
groups of data (e.g. the experiment and control group).    We can calculate a summary statistic
for the whole data-set using the dplyr's `summarise()` function:


~~~
brca %>% 
  filter(HER2_STATUS=="Positive") %>% 
  summarise(mean_os = mean(OS_MONTHS))
~~~
{: .language-r}



~~~
# A tibble: 1 x 1
  mean_os
    <dbl>
1    25.8
~~~
{: .output}

To generate summary statistics for each value of another variable we use the 
`group_by()` function:


~~~
brca %>% 
  filter(HER2_STATUS %in% c("Positive", "Negative")) %>% 
  group_by(HER2_STATUS) %>% 
  summarise(mean_os = mean(OS_MONTHS))
~~~
{: .language-r}



~~~
# A tibble: 2 x 2
  HER2_STATUS mean_os
  <chr>         <dbl>
1 Negative       29.2
2 Positive       25.8
~~~
{: .output}



> ## Statistics revision
> 
> If you need to revise or learn about statistical concepts, the University Library's "My Learning Essentials" team have produced a site [Start to Finish:Statistics](https://www.escholar.manchester.ac.uk/learning-objects/mle/packages/statistics/) which covers important statistical concepts.
> 
{: .callout}


> ## Challenge 3
>
> For each combination of HER2 and ER status, calculate the average OS.
>
> > ## Solution to Challenge 3
> >
> >
> >~~~
> > brca_mean_os = brca %>% 
> >   group_by(HER2_STATUS, ER_STATUS) %>% 
> >   summarise(mean_os = mean(OS_MONTHS))
> > print(brca_mean_os)
> >~~~
> >{: .language-r}
> >
> >
> >
> >~~~
> ># A tibble: 17 x 3
> ># Groups:   HER2_STATUS [?]
> >   HER2_STATUS   ER_STATUS                   mean_os
> >   <chr>         <chr>                         <dbl>
> > 1 Equivocal     Negative                      8.97 
> > 2 Equivocal     Positive                     22.1  
> > 3 Negative      Negative                     28.5  
> > 4 Negative      Not Performed                 2.26 
> > 5 Negative      Performed but Not Available  75.4  
> > 6 Negative      Positive                     30.7  
> > 7 Not Available Negative                     74.6  
> > 8 Not Available Not Performed                13.1  
> > 9 Not Available Positive                     54.3  
> >10 Positive      Indeterminate                24.8  
> >11 Positive      Negative                     30.1  
> >12 Positive      Not Performed                 0.015
> >13 Positive      Positive                     24.7  
> >14 <NA>          Negative                     58.7  
> >15 <NA>          Performed but Not Available  59.9  
> >16 <NA>          Positive                     56.8  
> >17 <NA>          <NA>                         NA    
> >~~~
> >{: .output}
> {: .solution}
{: .challenge}

## `count()` and `n()`
A very common operation is to count the number of observations for each
group. The `dplyr` package comes with two related functions that help with this.


If we need to use the number of observations in calculations, the `n()` function
is useful. For instance, if we wanted to get the standard error of the life
expectancy per continent:


~~~
brca %>% 
  group_by(HER2_STATUS) %>% 
  summarise(n_pantients = n())
~~~
{: .language-r}



~~~
# A tibble: 5 x 2
  HER2_STATUS   n_pantients
  <chr>               <int>
1 Equivocal              10
2 Negative              652
3 Not Available          15
4 Positive              114
5 <NA>                  103
~~~
{: .output}

Although we could use the `group_by()`, `n()` and `summarize()` functions to calculate the number of observations in each group, `dplyr` provides the `count()` function which automatically groups the data, calculates the totals and then un-groups it. 


~~~
brca %>%
    filter(HER2_STATUS %in% c("Positive", "Negative")) %>%
    count(HER2_STATUS, sort = TRUE)
~~~
{: .language-r}



~~~
# A tibble: 2 x 2
  HER2_STATUS     n
  <chr>       <int>
1 Negative      652
2 Positive      114
~~~
{: .output}

We can optionally sort the results in descending order by adding `sort=TRUE`.

## Connect mutate with logical filtering: `ifelse()`

When creating new variables, we can hook this with a logical condition. A simple combination of 
`mutate()` and `ifelse()` facilitates filtering right where it is needed: in the moment of creating something new.
This easy-to-read statement is a fast and powerful way of discarding certain data (even though the overall dimension
of the tibble will not change) or for updating values depending on this given condition.

The `ifelse()` function takes three parameters.  The first it the logical test.  The second is the value to use if the test is TRUE for that observation, and the third is the value to use if the test is FALSE.


~~~
# Categorize patients into "young" and "old" based on age.
brca %>%
  mutate(age_group = ifelse(AGE > 50, "old", "young")) %>% 
  select(PATIENT_ID, AGE, age_group)
~~~
{: .language-r}



~~~
# A tibble: 894 x 3
   PATIENT_ID        AGE age_group
   <chr>           <int> <chr>    
 1 TCGA-A2-A0T2-01    66 old      
 2 TCGA-A2-A04P-01    36 young    
 3 TCGA-A1-A0SK-01    54 old      
 4 TCGA-A2-A0CM-01    40 young    
 5 TCGA-AR-A1AR-01    50 young    
 6 TCGA-B6-A0WX-01    40 young    
 7 TCGA-BH-A1F0-01    80 old      
 8 TCGA-B6-A0I6-01    49 young    
 9 TCGA-BH-A18V-01    48 young    
10 TCGA-BH-A18Q-01    56 old      
# ... with 884 more rows
~~~
{: .output}


> ## Equivalent functions in base R
>
> In this course we've taught the tidyverse.  You are likely come across
> code written others in base R.  You can find a guide to some base R functions
> and their tidyverse equivalents [here](http://www.significantdigits.org/2017/10/switching-from-base-r-to-tidyverse/),
> which may be useful when reading their code.
>
{: .callout}
## Other great resources

* [Data Wrangling tutorial](https://suzan.rbind.io/categories/tutorial/) — an excellent four part tutorial covering selecting data, filtering data, summarising and transforming your data.
* [R for Data Science](http://r4ds.had.co.nz/)
* [Data Wrangling Cheat sheet](https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)
* [Introduction to dplyr](https://cran.r-project.org/web/packages/dplyr/vignettes/dplyr.html) — this is the package vignette.  It can be viewed within R using `vignette(package="dplyr", "dplyr")`
* [Data wrangling with R and RStudio](https://www.rstudio.com/resources/webinars/data-wrangling-with-r-and-rstudio/)
