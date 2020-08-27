---
title: "Introduction: Stata to R"
author: "Lucy Hackett"
date: "7/8/2020"
output: 
  html_document:
    theme: yeti
    highlight: haddock 
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
---

<style type="text/css">
body, td {
   font-size: 14px;
}
code.r{
  font-size: 16px;
}
pre {
  font-size: 16px
}
</style>



## Introduction

Wecome to the introductory R Markdown document! In this short tutorial, we will take a shallow dive into the use of R (with a focus on the tidyverse) for reading, inspecting, cleaning and manipulating data. There are MANY excellent intros to R and data wrangling out there. To name a few:

* [Modern Dive](https://moderndive.com/index.html)
* [Tiago Ventura's Intro](https://tiagoventura.rbind.io/files/gsa_tidyverse_workshop)
* [D-Lab's workshops](https://dlab.berkeley.edu/training)

As well as hundreds of tutorials on DataCamp. I will not attempt to recreate these resources. Rather, view this guide as a "translation" of stata-ish syntax and logic to the R system. Therefore I will not go into detail about the different objects and syntax that I use here, but rather will work through practical examples of how things would be done in Stata vs. R

Also note that the flexibility of R allows for many potential ways to do things, making it difficult and probably ill-advised for me to create a one-to-one dictionary of Stata commands to R commands. While this can be overwhelming and confusing at first, with time I think you'll find that flexibility is actually a great feature that allows you to choose the right/most efficient/easiest method depending on your particular data structure or needs.

## Importing packages

In R we have what's called "base R" and there are packages. Base R has tons of capabilities for statistical and probability work, but packages expand this capability by providing us with more specialized commands for things like data visualization, data cleaning and Econometric models. In Stata, use of external packages is less prevalent, but you still may use external packages such as ```ivreg2``` or ```binscatter```, which you gain access to by a one-time installation via ```ssc install```. In R, we can do this either through the package manager window on the lower right side of RStudio, or through the ```install.packages()``` command. Once we have installed the package, in order to use it we call the following: 


```r
library(dplyr)
library(ggplot2)
library(haven)
```

## Directories, importing data 

When working in Stata, frequently the first line in any dofile is ``cd``. In R, we set our current directory with:


```r
setwd('/users/hackettl/drive_berkeley/GSI_GSR/2020_fall/tutorials')
```



A great advantage of R over Stata is you can import many different kinds of files with realtive ease, and hold several datasets in memory at a time. Let's import a few files I have on hand.

**sinac_sample.dta** is a 1% random sample of birth registries from 2008 in Mexico. **cat_local_inegi.csv** is a dictionary of Mexican localities; it gives their name, state, municipality and population.

Here I use the most simple import, but like in insheet, you can choose to start/stop at certain csv rows, change the column names to lowercase, etc; for these options and many more, see the documentation.

Note that to read the birth data I use ``read_dta`` which is a command provided by the **haven** package.


```r
sinac <- read_dta('data/sinac_sample.dta')
local <- read.csv('data/cat_local_inegi.csv')
```

## Inspecting data

A good first step is always to take a look at the data; see how many (non-missing) observations we have, how the data behaves, etc. Here I show some examples of how to do this. In this section we will be mimicking Stata commands such as

* br
* sum
* bys: sum
* hist
* tab
* duplicates drop
* sort

I'll put the Stata command in a comment above the python command where possible, so if you're looking for a specific command, you can search this notebook for that command (i.e., search the document for "tab" for example.





```r
# br
head(sinac, row = 10)
```

```
## # A tibble: 6 x 18
##   edad_madre numero_embarazos talla_nac_vivo peso_nac_vivo mun_res_cve ent_res_cve mun_nac_cve
##        <dbl>            <dbl>          <dbl>         <dbl>       <dbl>       <dbl>       <dbl>
## 1         36                2             49          3310          46          19          39
## 2         28                3             48            NA          33          15           7
## 3         20                2             48          3000         106          16         106
## 4         20                1             52          3400          25           5          25
## 5         27                1             50          3490          51          32          17
## 6         30                6             51          3070          57          24          37
## # … with 11 more variables: ent_nac_cve <dbl>, tot_consult <dbl>, schooling_mother <dbl>,
## #   dia_nac_hijo <dbl>, mes_nac_hijo <dbl>, year_nac_hijo <dbl>, sexo_nac <dbl+lbl>,
## #   semanas_gest <dbl>, apgar <dbl>, proced <dbl+lbl>, atendio <dbl+lbl>
```

I don't always love the information I can see from ``head()``, so if you want a more detailed look you can click on the dataset name in the Environment window (upper right in RStudio) to see a larger format of the data, though this isn't always bery efficient. You can also use this window to see a complete list of all the variables!

To get a quick look at some summary statistics of the data, you can use ``dfapply()`` which is an iterative function with the option "favstats":


```r
# sum sexo_nac
summary(sinac$sexo_nac)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##  0.0000  0.0000  0.0000  0.4884  1.0000  1.0000      35
```

Note that in R, we have to associate variables we are interested in with the dataset they belong to, because we may and in fact do in this case have several datasets in memory. To do this, we use the syntax ``dataset_name$variable_name``. In the tidyverse and in other packages, we can use the pipe ``%>%`` to pass identifying information or transformations onto the next operation:


```r
# tab atendio
sinac %>% count(atendio)
```

```
## # A tibble: 6 x 2
##                        atendio     n
##                      <dbl+lbl> <int>
## 1  1 [Nurse]                      66
## 2  2 [Midwife]                   454
## 3  3 [Authorized person by SS]     8
## 4  4 [Doctor]                  19060
## 5  7 [Other]                     174
## 6 NA                              22
```



```r
# duplicates drop, save to different data
sinac_unique <- sinac %>% distinct()
```



```r
# to do operations by groups, like bys: sum 
# (you can add more variables to the list at will)
# note that we need to ignore missing values with na.rm = TRUE
sinac %>% group_by(ent_res_cve) %>% 
  summarise(avg_weight = mean(peso_nac_vivo, na.rm = TRUE),
            min_weight = min(peso_nac_vivo, na.rm = TRUE))
```

```
## # A tibble: 33 x 3
##    ent_res_cve avg_weight min_weight
##          <dbl>      <dbl>      <dbl>
##  1           1      3115.        500
##  2           2      3310.       1800
##  3           3      3301.        950
##  4           4      3092        1000
##  5           5      3278.       1000
##  6           6      3345.       1875
##  7           7      3185.       1115
##  8           8      3247.       1550
##  9           9      3051.        792
## 10          10      3225.       1500
## # … with 23 more rows
```

Be careful saving data as grouped, as this may affect future modifications you make to the data in unexpected ways!


```r
# view the data sorted: sort
sinac %>% arrange(edad_madre,apgar)
```

```
## # A tibble: 19,784 x 18
##    edad_madre numero_embarazos talla_nac_vivo peso_nac_vivo mun_res_cve ent_res_cve mun_nac_cve
##         <dbl>            <dbl>          <dbl>         <dbl>       <dbl>       <dbl>       <dbl>
##  1          9                1             51          3100           4           2           4
##  2          9                1             54          3100           9           1           7
##  3         10                1             NA            NA          12          25          12
##  4         10                1             52          3305          17          11          17
##  5         10                1             53          3210          92          31          96
##  6         10                1             50          2620           2           9           2
##  7         10                2             48          3100          17          11          17
##  8         10                1             50          3400          17          18          17
##  9         10                1             NA            NA         122          15         122
## 10         10                2             51          3400           3          11           3
## # … with 19,774 more rows, and 11 more variables: ent_nac_cve <dbl>, tot_consult <dbl>,
## #   schooling_mother <dbl>, dia_nac_hijo <dbl>, mes_nac_hijo <dbl>, year_nac_hijo <dbl>,
## #   sexo_nac <dbl+lbl>, semanas_gest <dbl>, apgar <dbl>, proced <dbl+lbl>, atendio <dbl+lbl>
```

```r
# again, here I haven't saved this, I'm just looking at it. 
# To save this, set it equal to a name
```

Looks like the youngest mothers in our data are 9 years old. Let's plot mother's age to get a sense of the distribution:


```r
hist(sinac$edad_madre)
```

![](intro_stata_to_r_files/figure-html/hist-1.png)<!-- -->

Here's a nicer version using ``ggplot``:


```r
ggplot(data = sinac, aes(x = edad_madre)) +
  geom_histogram(color="darkblue", fill="lightblue") + 
  xlab("Mother's age") + 
  theme_bw()
```

```
## Warning: Removed 173 rows containing non-finite values (stat_bin).
```

![](intro_stata_to_r_files/figure-html/gghist-1.png)<!-- -->

##  A note about object assignment
A theme you may have noted above is that we can do things to the data without changing the underlying data or even saving what we're doing. This may remind you of what you might do in Stata with ``preserve ... restore``, which is the only way in Stata (besides tempfiles I guess) to explore manipulations of the data without altering the underlying data.

This is one of the main advantages that R has over Stata; we can hold multiple data, or multiples versions/ manipulations of the data in memory, leaving our original dataset intact! 

## Data cleaning 

Now we've looked around a bit at the data, we're going to clean it. These type of commands in Stata are:

* rename
* replace
* gen
* collapse
* merge

First let's get these column names in English...


```r
# rename
# note that here I AM reassigning the data to save changes
sinac <- sinac %>% rename(mother_age = edad_madre,
                           no_preg = numero_embarazos,
                           length = talla_nac_vivo,
                           weight = peso_nac_vivo,
                           no_appts = tot_consult,
                           day_born = dia_nac_hijo,
                           mo_born = mes_nac_hijo,
                           yr_born = year_nac_hijo, 
                           sex = sexo_nac, 
                           gest_age = semanas_gest, 
                           type_doc = atendio)

sinac %>% head()
```

```
## # A tibble: 6 x 18
##   mother_age no_preg length weight mun_res_cve ent_res_cve mun_nac_cve ent_nac_cve no_appts
##        <dbl>   <dbl>  <dbl>  <dbl>       <dbl>       <dbl>       <dbl>       <dbl>    <dbl>
## 1         36       2     49   3310          46          19          39          19       10
## 2         28       3     48     NA          33          15           7           9       12
## 3         20       2     48   3000         106          16         106          16        2
## 4         20       1     52   3400          25           5          25           5        9
## 5         27       1     50   3490          51          32          17          32        7
## 6         30       6     51   3070          57          24          37          24        1
## # … with 9 more variables: schooling_mother <dbl>, day_born <dbl>, mo_born <dbl>, yr_born <dbl>,
## #   sex <dbl+lbl>, gest_age <dbl>, apgar <dbl>, proced <dbl+lbl>, type_doc <dbl+lbl>
```


```r
# Let's change the state code from 99 to missing
# replace
# ifelse has arguments condition, what to do if true, then what to do if false
sinac <- sinac %>% mutate(ent_res_cve = 
                            ifelse(
                              ent_res_cve == 99,
                              NaN,
                              ent_res_cve
))

summary(sinac$ent_res_cve)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##    1.00   11.00   15.00   16.67   22.00   32.00     149
```

```r
# good! looks like the max code is now 32 (the number of states in Mexico)
```



```r
# gen
# Now let's make a categorical variable for mother's schooling 
sinac <- sinac %>% mutate(schooling_cat = 
                           ifelse(
                             schooling_mother < 4,
                             1,
                             ifelse(
                               schooling_mother >= 4 & schooling_mother <= 6,
                               2,
                               3)
                           ))

summary(sinac$schooling_cat)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##   1.000   1.000   2.000   1.986   3.000   3.000     354
```

Now let's look at the distribution of these categories, which if you're curious correspond to completed secondary, completed high school, and more than high school:


```r
# first label my variable by making it a factor var

sinac <- sinac %>% mutate(school = factor(schooling_cat,
                                          levels = c(1,2,3),
                                          labels = c("Secondary", "HS", "HS+")
                                          )
                          )

ggplot(sinac, aes(school)) + 
  geom_bar(color="darkblue", fill="lightblue") + 
  labs(x = 'Schooling', y = '# observations') +
  theme_bw()
```

![](intro_stata_to_r_files/figure-html/plotschool-1.png)<!-- -->

### Aggregating data (collapse)

Let's get the mean by state; to do this, we want to collapse by the variable 'ent_res_cve'. If you want to group by more variables, just add them to the list for the "by" option. You can also write your own funtions for the "FUN" option, which makes this method about as flexible as you could want!


```r
## stata: collapse (mean) **, by(mun_res_cve ent_res_cve)
sinac_means <- sinac %>% 
  select(mother_age, weight, length, schooling_mother) %>% 
  aggregate(by=list(sinac$ent_res_cve),
            FUN = mean,
            na.rm = TRUE) %>%
  rename(ent_res_cve = Group.1)
```


 ent_res_cve   mother_age     weight     length   schooling_mother
------------  -----------  ---------  ---------  -----------------
           1     25.40152   3114.582   49.57364           5.307087
           2     24.16774   3309.502   50.61504           4.878788
           3     24.33645   3301.020   50.90000           5.175926
           4     25.17213   3092.000   49.11570           5.033058
           5     24.60542   3277.745   50.81377           5.572835
           6     25.14000   3344.837   50.52688           5.424242
           7     25.36635   3184.692   49.72425           4.148265
           8     24.34216   3247.320   50.84615           4.994318
           9     25.57806   3051.453   49.66126           5.709814
          10     25.18333   3224.649   51.04545           5.197987

### Merging 

In Stata, if you're like me you love the table Stata prints for you after a merge reporting how many observations were merged, vs. right- and left-only. This simple report gives us lots of hints about things that may be going wrong, differences in reporting between datasets, and many other sneaky data details. 

In R this is not done exactly the same, but we can still check for these issues using more flexible merge options. Here I will focus on the ``dplyr`` way of merging because it is intuitive, especially for Stata fans, though base R also has a merge function. 

In dplyr there are three separate commands for merging:

* ``inner_join()`` merges the tables and keeps only those observations found in both tables, like running ``keep if _m == 3`` after a Stata merge. 

* ``left_join()`` merges the tables and keeps only observations matched or unmatched but found in the "master" data, like running ``keep if _m == 3 | _m == 1`` in Stata. 

* ``right_join()`` merges the tables and keeps only observations matched or unmatched but found in the "merging" data, like running ``keep if _m == 3 | _m == 2`` in Stata. 

Thre are more fun commands available like ``semi_join()`` and ``anti_join()`` which can be very useful for several tricks we use merging for, like using merging to identify observations we want to keep or discard, but I won't go into those here.

Another nice feature of these join functions is that instead of renaming in order to get matching columns, dplyr will let us tell it that differently named columns across datasets are really the same. Let's see an example. 

Say I want to add to this dataset the population of each municipality. My dictionary is at the locality level, but I need the municipal level. So I'm going to follow these steps:

1. Collapse dictionary to municipal level
2. Merge



```r
# we want total population, so our function for aggregation will be sum:
local_mun <- local %>% 
  select(CVE_ENT, CVE_MUN, POB_TOTAL) %>% 
  aggregate(by=list(local$CVE_ENT, local$CVE_MUN),
            FUN = sum,
            na.rm = TRUE) %>%
  rename(ent_res_cve = Group.1,
         mun_res_cve = Group.2) %>%
  select(ent_res_cve, mun_res_cve, POB_TOTAL)
```


 ent_res_cve   mun_res_cve   POB_TOTAL
------------  ------------  ----------
           1             1      796667
           2             1      466811
           3             1       70656
           4             1       52869
           5             1        1070
           6             1       28683
           7             1       16814
           8             1       11456
          10             1       31307
          11             1       84290

