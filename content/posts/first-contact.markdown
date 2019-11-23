---
date: "2019-11-23"
title: "First contact with the data on R"
author: "Etienne Bacher"
draft: true
tags: ["R", "Stata"]
output: 
  blogdown::html_page:
    toc: true 
---

In this post, you will see how to import and treat data, make descriptive statistics and a few plots. However, I am still a recent user of R and I don't know much about the optimal organization of files in a project (and many people explain it better than me online). Therefore, I will show you a personal method that is perhaps not the most effective but that works.

## Files used and organization of the project


In the folder that contains the project, I have several sub-folders: Figures, Bases_Used, Bases_Created. To be able to save or use files in these particular sub-folders, I put their path as vectors.


```r
figdir <- c("/your/path/to/your/project/Figures/")
base_used_dir <- c("/your/path/to/your/project/Bases_Used/")
base_created_dir <- c("/your/path/to/your/project/Bases_Created/")
```


## Import data

We will use data contained in Excel (`.xlsx`) and text (`.txt`) files. You can find these files (and the full R script corresponding to this post) [here](https://github.com/etiennebacher/from-stata-to-r/tree/master/public/Data/first-contact). To import Excel data, we wil need the `readxl` package. 


```r
# if you've never installed this package before, do:
# install.packages("readxl")
library(readxl)
```

We use the `read_excel` function of this package to import excel files and the function `read.table` (in base R) to import the data: 

<!-- La partie qui suit doit être visible pour correspondre à ce que je dis mais ne doit pas être exécutée car pas les bons chemins d'accès -->

```r
base1 <- read_excel(paste(base_used_dir, "Base_Excel.xlsx", sep = ""), sheet = "Base1")
base2 <- read_excel(paste(base_used_dir, "Base_Excel.xlsx", sep = ""), sheet = "Base2")
base3 <- read_excel(paste(base_used_dir, "Base_Excel.xlsx", sep = ""), sheet = "Base3")
base4 <- read.table(paste(base_used_dir, "Base_Text.txt", sep = ""), header = TRUE)
```
<!-- La partie qui suit ne doit pas être visible mais doit être exécutée -->


Now, we have stored the four datasets in four objects called `data.frames`. To me, this simple thing is an advantage on Stata where storing multiple datasets in the same time is not intuitive at all.


## Merge dataframes

We want to have a unique dataset to make descriptive statistics and econometrics (we will just do descriptive statistics in this post). Therefore, we will merge these datasets together, first by using the `dplyr` package. This package is one of the references for data manipulation. It is extremely useful and much more easy to use than base R. You may find a cheatsheet (i.e. a recap of the functions) for this package [here](https://rstudio.com/resources/cheatsheets/), along with cheatsheets of many other great packages.

First, we want to regroup `base1` and `base2`. To do so, we just need to put one under the other and to "stick" them together with `bind_rows` and we observe the result:


```r
library(dplyr)
base_created <- bind_rows(base1, base2)
base_created
```

```
## # A tibble: 23 x 6
##     hhid indidy1 surname   name     gender  wage
##    <dbl>   <dbl> <chr>     <chr>     <dbl> <dbl>
##  1     1       1 BROWN     Robert        1  2000
##  2     1       2 JONES     Michael       1  2100
##  3     1       3 MILLER    William       1  2300
##  4     1       4 DAVIS     David         1  1800
##  5     2       1 RODRIGUEZ Mary          2  3600
##  6     2       2 MARTINEZ  Patricia      2  3500
##  7     2       3 WILSON    Linda         2  1900
##  8     2       4 ANDERSON  Richard       1  1900
##  9     3       1 THOMAS    Charles       1  1800
## 10     3       2 TAYLOR    Barbara       2  1890
## # … with 13 more rows
```

As you can see, we obtain a dataframe with 6 columns (like each table separately) and 23 rows: 18 in the first table, 5 in the second table. Now, we merge this dataframe with `base3`. `base_created` and `base3` only have one column in common (`hhid`) so we will need to specify that we want to merge these two bases by this column:


```r
base_created <- left_join(base_created, base3, by = "hhid")
base_created
```

```
## # A tibble: 23 x 7
##     hhid indidy1 surname   name     gender  wage location
##    <dbl>   <dbl> <chr>     <chr>     <dbl> <dbl> <chr>   
##  1     1       1 BROWN     Robert        1  2000 France  
##  2     1       2 JONES     Michael       1  2100 France  
##  3     1       3 MILLER    William       1  2300 France  
##  4     1       4 DAVIS     David         1  1800 France  
##  5     2       1 RODRIGUEZ Mary          2  3600 England 
##  6     2       2 MARTINEZ  Patricia      2  3500 England 
##  7     2       3 WILSON    Linda         2  1900 England 
##  8     2       4 ANDERSON  Richard       1  1900 England 
##  9     3       1 THOMAS    Charles       1  1800 Spain   
## 10     3       2 TAYLOR    Barbara       2  1890 Spain   
## # … with 13 more rows
```

`left_join` is a `dplyr` function saying that the first dataframe mentioned (here `base_created`) is the "most important" and that we will stick the second one (here `base3`) to it. If there are more rows in the first one than in the second one, then there will be some missing values but the number of rows will stay the same. If we knew that `base3` had more rows than `base_created`, we would have used `right_join`.

We now want to merge `base_created` with `base4`. The problem is that there are no common columns so we will need to create one in each. Moreover, `base_created` contains data for the year 2019 and `base4` for the year 2020. We will need to create columns to specify that too:


```r
# rename the second column of base_created and of base4
colnames(base_created)[2] <- "indid"
colnames(base4)[2] <- "indid"

# create the column "year", that will take the value 2019 
# for base_created and 2020 for base4
base_created$year <- 2019
base4$year <- 2020
```

From this point, we can merge these two dataframes:


```r
base_created2 <- bind_rows(base_created, base4)
base_created2
```

```
## # A tibble: 46 x 8
##     hhid indid surname   name     gender  wage location  year
##    <dbl> <dbl> <chr>     <chr>     <dbl> <dbl> <chr>    <dbl>
##  1     1     1 BROWN     Robert        1  2000 France    2019
##  2     1     2 JONES     Michael       1  2100 France    2019
##  3     1     3 MILLER    William       1  2300 France    2019
##  4     1     4 DAVIS     David         1  1800 France    2019
##  5     2     1 RODRIGUEZ Mary          2  3600 England   2019
##  6     2     2 MARTINEZ  Patricia      2  3500 England   2019
##  7     2     3 WILSON    Linda         2  1900 England   2019
##  8     2     4 ANDERSON  Richard       1  1900 England   2019
##  9     3     1 THOMAS    Charles       1  1800 Spain     2019
## 10     3     2 TAYLOR    Barbara       2  1890 Spain     2019
## # … with 36 more rows
```

But we have many missing values for the new rows because `base4` only contained three columns. We want to have a data frame arranged by household then by individual and finally by year. Using only `dplyr` functions, we can do:


```r
base_created2 <- base_created2 %>% 
  group_by(hhid, indid) %>% 
  arrange(hhid, indid, year) %>%
  ungroup()
base_created2
```

```
## # A tibble: 46 x 8
##     hhid indid surname   name    gender  wage location  year
##    <dbl> <dbl> <chr>     <chr>    <dbl> <dbl> <chr>    <dbl>
##  1     1     1 BROWN     Robert       1  2000 France    2019
##  2     1     1 <NA>      <NA>        NA  2136 <NA>      2020
##  3     1     2 JONES     Michael      1  2100 France    2019
##  4     1     2 <NA>      <NA>        NA  2362 <NA>      2020
##  5     1     3 MILLER    William      1  2300 France    2019
##  6     1     3 <NA>      <NA>        NA  2384 <NA>      2020
##  7     1     4 DAVIS     David        1  1800 France    2019
##  8     1     4 <NA>      <NA>        NA  2090 <NA>      2020
##  9     2     1 RODRIGUEZ Mary         2  3600 England   2019
## 10     2     1 <NA>      <NA>        NA  3784 <NA>      2020
## # … with 36 more rows
```

Notice that there are `%>%` between the lines: it is a pipe and its function is to connect lines of code between them so that we don't have to write `base_created2` every time. Now that our dataframe is arranged, we need to fill the missing values. Fortunately, these missing values do not change for an individual since they concern the gender, the location, the name and the surname. So basically, we can just take the value of the cell above (corresponding to year 2019) and replicate it in each cell (corresponding to year 2020):


```r
library(tidyr)
base_created2 <- base_created2 %>%
  fill(select_if(., ~ any(is.na(.))) %>% 
         names(),
       .direction = 'down')
```

Let me explain the code above:

* `fill` aims at fill cells
* `select_if` selects columns according to the condition defined
* `any(is.na(.))` is a logical question asking if there are missing values (NA)
* `.` indicates that we want to apply the function to the whole dataframe
* `names` tells us what the names of the columns selected are
* `.direction` tells the direction in which the filling goes

So `fill(select_if(., ~ any(is.na(.))) %>% names(), .direction = 'down')` means that for the dataframe, we select each column which has some NA in it and we obtain their names. In these columns, the empty cells are filled by the value of the cell above (since the direction is "down").

Finally, we want the first three columns to be `hhid`, `indid` and `year`, and we create a ID column named `hhind` which is just the union of `hhid` and `indid`.


```r
base_created2 <- base_created2 %>%
  select(hhid, indid, year, everything()) %>%
  unite(hhind, c(hhid, indid), sep = "", remove = FALSE) 
base_created2
```

```
## # A tibble: 46 x 9
##    hhind  hhid indid  year surname   name    gender  wage location
##    <chr> <dbl> <dbl> <dbl> <chr>     <chr>    <dbl> <dbl> <chr>   
##  1 11        1     1  2019 BROWN     Robert       1  2000 France  
##  2 11        1     1  2020 BROWN     Robert       1  2136 France  
##  3 12        1     2  2019 JONES     Michael      1  2100 France  
##  4 12        1     2  2020 JONES     Michael      1  2362 France  
##  5 13        1     3  2019 MILLER    William      1  2300 France  
##  6 13        1     3  2020 MILLER    William      1  2384 France  
##  7 14        1     4  2019 DAVIS     David        1  1800 France  
##  8 14        1     4  2020 DAVIS     David        1  2090 France  
##  9 21        2     1  2019 RODRIGUEZ Mary         2  3600 England 
## 10 21        2     1  2020 RODRIGUEZ Mary         2  3784 England 
## # … with 36 more rows
```

That's it, we now have the complete dataframe. 


## Clean the data

There are still some things to do. First, we remark that there are some errors in the column `location` (`England_error` and `Spain_error`) so we correct it:


```r
# display the unique values of the column "location"
unique(base_created2$location)
```

```
## [1] "France"        "England"       "Spain"         "Italy"        
## [5] "England_error" "Spain_error"
```

```r
# correct the errors
base_created2[base_created2 == "England_error"] <- "England"
base_created2[base_created2 == "Spain_error"] <- "Spain"
unique(base_created2$location)
```

```
## [1] "France"  "England" "Spain"   "Italy"
```

Basically, what we've done here is that we have selected every cell in the whole dataframe that had the value `England_error` (respectively `Spain_error`) and we replaced these cells by `England` (`Spain`). We also need to recode the column `gender` because binary variables have to take values of 0 or 1, not 1 or 2.


```r
base_created2$gender <- recode(base_created2$gender, `2` = 0)
```

To have more details on the dataframe, we need to create some labels. To do so, we need the `upData` function in the `Hmisc` package.


```r
library(Hmisc)
var.labels <- c(hhind = "individual's ID",
                hhid = "household's ID",
                indid = "individual's ID in the household",
                year = "year",
                surname = "surname",
                name = "name",
                gender = "1 if male, 0 if female",
                wage = "wage",
                location = "location")
base_created2 <- upData(base_created2, labels = var.labels)
```

We can see the result with: 


```r
contents(base_created2)
```

```
## 
## Data frame:base_created2	46 observations and 9 variables    Maximum # NAs:0
## 
## 
##                                    Labels     Class   Storage
## hhind                     individual's ID character character
## hhid                       household's ID   integer   integer
## indid    individual's ID in the household   integer   integer
## year                                 year   integer   integer
## surname                           surname character character
## name                                 name character character
## gender             1 if male, 0 if female   integer   integer
## wage                                 wage   integer   integer
## location                         location character character
```

Now that our dataframe is clean and detailed, we can compute some descriptive statistics. But before doing it, we might want to save it:


```r
write.xlsx(base_created2, file = paste(base_created_dir, "modified_base.xlsx", sep = ""))
```


## Descriptive Statistics

First of all, if we want to check the number of people per location or gender and per year, we use the `table` function:


```r
table(base_created2$gender, base_created2$year)
```

```
##    
##     2019 2020
##   0    9    9
##   1   14   14
```

```r
table(base_created2$location, base_created2$year)
```

```
##          
##           2019 2020
##   England    6    6
##   France    12   12
##   Italy      1    1
##   Spain      4    4
```

To have more detailed statistics, you can use many functions. Here, we use


```r
summary(base_created2)
```

```
##     hhind                hhid           indid            year     
##  Length:46          Min.   :1.000   Min.   :1.000   Min.   :2019  
##  Class1:labelled    1st Qu.:2.000   1st Qu.:1.000   1st Qu.:2019  
##  Class2:character   Median :5.000   Median :2.000   Median :2020  
##  Mode  :character   Mean   :4.217   Mean   :2.217   Mean   :2020  
##                     3rd Qu.:6.000   3rd Qu.:3.000   3rd Qu.:2020  
##                     Max.   :8.000   Max.   :5.000   Max.   :2020  
##    surname              name               gender            wage     
##  Length:46          Length:46          Min.   :0.0000   Min.   :1397  
##  Class1:labelled    Class1:labelled    1st Qu.:0.0000   1st Qu.:1800  
##  Class2:character   Class2:character   Median :1.0000   Median :1901  
##  Mode  :character   Mode  :character   Mean   :0.6087   Mean   :2059  
##                                        3rd Qu.:1.0000   3rd Qu.:2098  
##                                        Max.   :1.0000   Max.   :3784  
##    location        
##  Length:46         
##  Class1:labelled   
##  Class2:character  
##  Mode  :character  
##                    
## 
```

but you can try the function `stat.desc` in `pastecs`, `skim` in `skimr` or even `makeDataReport` in `dataMaid` to have a complete PDF report summarizing your data. To summarize data under certain conditions (e.g. to have the average wage for each location), you can use `dplyr`:


```r
# you can change the argument in group_by() by gender for example
base_created2 %>%
  group_by(location) %>%
  summarize_at(.vars = "wage", .funs = "mean")
```

```
## # A tibble: 4 x 2
##   location    wage
##   <labelled> <dbl>
## 1 England    2452.
## 2 France     1935.
## 3 Italy      1801 
## 4 Spain      1905.
```


## Plots

Finally, we want to plot some data to include in our report or article (or anything else). `ggplot2` is THE reference to make plots with R. The `ggplot` function does not create a graph but tells what is the data you are going to use and the aesthetics (`aes`). Here, we want to display the wages in a histogram and to distinguish them per year. Therefore, we want to fill the bars according to the year. To precise the type of graph we want, we add `+ geom_histogram()` after `ggplot`. You may change the number of `bins` to have a more precise histogram. 


```r
library(ggplot2)
hist1 <- ggplot(data = base_created2, 
                mapping = aes(wage, fill = factor(year))) +
  geom_histogram(bins = 10)
hist1
```

<img src="/posts/first-contact_files/figure-html/unnamed-chunk-20-1.png" width="672" />
If you prefer one histogram per year, you can use the `facet_grid()` argument, as below.


```r
hist2 <- ggplot(data = base_created2, 
                mapping = aes(wage)) +
  geom_histogram(bins = 10) +
  facet_grid(cols = vars(year), scales = "fixed")
hist2
```

<img src="/posts/first-contact_files/figure-html/unnamed-chunk-21-1.png" width="672" />

Finally, you may want to export these graphs. To do so, we use `ggsave` (you can replace .pdf by .eps or .png if you want): 


```r
ggsave(paste(figdir, "plot1.pdf", sep = ""), plot = hist1)
```

That's it! In this first post, you have seen how to import, clean and tidy datasets, and how to make some descriptive statistics and some plots. I hope this was helpful to you!
