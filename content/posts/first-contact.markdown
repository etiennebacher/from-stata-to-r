---
date: "2020-01-20"
title: "First contact with the data on R"
author: "Etienne Bacher"
draft: true
tags: ["R", "Stata"]
output: 
  blogdown::html_page:
    toc: true 
---

<p style="color:rgb(127, 165, 179)"> <b> Note: </b>  </p> <p style="color:rgb(127, 165, 179)">In this and future articles, you will see some arrows below R code. If you click on it, it will display the Stata code equivalent to the R code displayed. However, since those are two different softwares, they are not completely equivalent and some of the Stata code may not fully correspond to the R code. Consider it more like a reference point not to be lost rather than like an exact equivalent. </p>

<br>

In this post, you will see how to import and treat data, make descriptive statistics and a few plots. I will also show you a personal method to organize one's work. 


## Files used and organization of the project

First of all, you need to create a project. In RStudio, you can do "File", "New Project" and then choose the location of the project and its name. In the folder that contains the project, I have several sub-folders: Figures, Bases_Used, Bases_Created. To be able to save or use files in these particular sub-folders, I use the package **`here`**. The command **`here()`** shows the path to your project and you just need to complete the path to access to your datasets or other files.


```r
# if you've never installed this package before, do:
# install.packages("here")
library(here)
```

Why is this package important? Your code must be reproducible, either for your current collaborators to work efficiently with you or for other people to check your code and to use it in the future. Using paths that work only for your computer (like "/home/Mr X/somefolder/somesubfolder/Project") makes it longer and more annoying to use your code since it requires to manually change paths in order to import data or other files. The package **`here`** makes it much easier to reproduce your code since it automatically detects the path to access to your data. You only need to keep the same structure between R files and datasets. You will see in the next part how to use it.

## Import data

We will use data contained in Excel (**`.xlsx`**) and text (**`.txt`**) files. You can find these files (and the full R script corresponding to this post) [here](https://github.com/etiennebacher/from-stata-to-r/tree/master/public/Data/first-contact). To import Excel data, we wil need the **`readxl`** package. 


```r
library(readxl)
```

We use the **`read_excel`** function of this package to import excel files and the function **`read.table`** (in base R) to import the data: 

<!-- La partie qui suit doit être visible pour correspondre à ce que je dis mais ne doit pas être exécutée car pas les bons chemins d'accès -->

```r
base1 <- read_excel(here("Bases_Used/Base_Excel.xlsx"), sheet = "Base1")
base2 <- read_excel(here("Bases_Used/Base_Excel.xlsx"), sheet = "Base2")
base3 <- read_excel(here("Bases_Used/Base_Excel.xlsx"), sheet = "Base3")
base4 <- read.table(here("Bases_Used/Base_Text.txt"), header = TRUE)
```
<!-- La partie qui suit ne doit pas être visible mais doit être exécutée -->


<details>
<summary> Stata
</summary>
<p>
```stata
cd "/path/to/Bases_Used"
import excel using Base_Excel, sheet("Base1") firstrow
``` 
</details>

As you can see, if your project is in a folder and if you stored you datasets in the Bases_Used subfolder, this code will work automatically since **`here`** detects the path. Now, we have stored the four datasets in four objects called **`data.frames`**. To me, this simple thing is an advantage on Stata where storing multiple datasets in the same time is not intuitive at all.


## Merge dataframes

We want to have a unique dataset to make descriptive statistics and econometrics (we will just do descriptive statistics in this post). Therefore, we will merge these datasets together, first by using the **`dplyr`** package. This package is one of the references for data manipulation. It is extremely useful and much more easy to use than base R. You may find a cheatsheet (i.e. a recap of the functions) for this package [here](https://rstudio.com/resources/cheatsheets/), along with cheatsheets of many other great packages.

First, we want to regroup **`base1`** and **`base2`**. To do so, we just need to put one under the other and to "stick" them together with **`bind_rows`** and we observe the result:


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

<details>
<summary> Stata
</summary>
<p>

```stata
preserve

*** Open base #2 and bind the rows
clear all 
import excel using Base_Excel, sheet("Base2") firstrow
tempfile base2
save  `base2' 
restore
append using `base2'
``` 
</details>

As you can see, we obtain a dataframe with 6 columns (like each table separately) and 23 rows: 18 in the first table, 5 in the second table. Now, we merge this dataframe with **`base3`**. **`base_created`** and **`base3`** only have one column in common (**`hhid`**) so we will need to specify that we want to merge these two bases by this column:


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

<details>
<summary> Stata
</summary>
<p>
```stata
preserve 

*** Open base #3 and merge
clear all
cd ..\Bases_Used 
import excel using Base_Excel, sheet("Base3") firstrow
tempfile base3
save `base3'
restore 
merge m:1 hhid using `base3' 
drop _merge 
```
</details>

**`left_join`** is a **`dplyr`** function saying that the first dataframe mentioned (here **`base_created`**) is the "most important" and that we will stick the second one (here **`base3`**) to it. If there are more rows in the first one than in the second one, then there will be some missing values but the number of rows will stay the same. If we knew that **`base3`** had more rows than **`base_created`**, we would have used **`right_join`**.

We now want to merge **`base_created`** with **`base4`**. The problem is that there are no common columns so we will need to create one in each. Moreover, **`base_created`** contains data for the year 2019 and **`base4`** for the year 2020. We will need to create columns to specify that too:


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

<details>
<summary> Stata
</summary>
<p>
```stata

rename indidy1 indid 
gen year=2019 
preserve 

* Open base #4 and merge
clear all
import delimited Base_Text.txt 
rename indidy2 indid 
gen year=2020
tempfile base4
save `base4'
restore 

merge 1:1 hhid indid year using `base4'
drop _merge
``` 
</details>

But we have many missing values for the new rows because **`base4`** only contained three columns. We want to have a data frame arranged by household then by individual and finally by year. Using only **`dplyr`** functions, we can do:


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

Notice that there are some **`%>%`** between the lines: it is a pipe and its function is to connect lines of code between them so that we don't have to write **`base_created2`** every time. Now that our dataframe is arranged, we need to fill the missing values. Fortunately, these missing values do not change for an individual since they concern the gender, the location, the name and the surname. So basically, we can just take the value of the cell above (corresponding to year 2019) and replicate it in each cell (corresponding to year 2020):


```r
library(tidyr)
base_created2 <- base_created2 %>%
  fill(select_if(., ~ any(is.na(.))) %>% 
         names(),
       .direction = 'down')
```

<details>
<summary> Stata
</summary>
<p>
```stata
foreach x of varlist surname name gender location {
  bysort hhid indid: replace `x'=`x'[_n-1] if year==2020
}
```
</details>

Let me explain the code above:

* **`fill`** aims to fill cells
* **`select_if`** selects columns according to the condition defined
* **`any(is.na(.))`** is a logical question asking if there are missing values (NA)
* **`.`** indicates that we want to apply the function to the whole dataframe
* **`names`** tells us what the names of the columns selected are
* **`.direction`** tells the direction in which the filling goes

So **`fill(select_if(., ~ any(is.na(.))) %>% names(), .direction = 'down')`** means that for the dataframe, we select each column which has some NA in it and we obtain their names. In these columns, the empty cells are filled by the value of the cell above (since the direction is "down").

Finally, we want the first three columns to be **`hhid`**, **`indid`** and **`year`**, and we create a ID column named **`hhind`** which is just the union of **`hhid`** and **`indid`**.


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

<details>
<summary> Stata
</summary>
<p>
```stata
egen hhind=group(hhid indid) 
order hhind hhid indid year * 
sort hhid indid year 
```
</details>

That's it, we now have the complete dataframe. 


## Clean the data

There are still some things to do. First, we remark that there are some errors in the column **`location`** (**`England_error`** and **`Spain_error`**) so we correct it:


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

<details>
<summary> Stata
</summary>
<p>
```stata
replace localisation="England" if localisation=="England_error"
replace localisation="Spain" if localisation=="Spain_error"
```
</details>

Basically, what we've done here is that we have selected every cell in the whole dataframe that had the value **`England_error`** (respectively **`Spain_error`**) and we replaced these cells by **`England`** (**`Spain`**). We also need to recode the column **`gender`** because binary variables have to take values of 0 or 1, not 1 or 2.


```r
base_created2$gender <- recode(base_created2$gender, `2` = 0)
```

<details>
<summary> Stata
</summary>
<p>
```stata
label define genderlab 1 "M" 2 "F"
label values gender genderlab
recode gender (2=0 "Female") (1=1 "Male"), gen(gender2)
drop gender
rename gender2 gender
```
</details>

To have more details on the dataframe, we need to create some labels. To do so, we need the **`upData`** function in the **`Hmisc`** package.


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
                location = "household's location")
base_created2 <- upData(base_created2, labels = var.labels)
```

<details>
<summary> Stata
</summary>
<p>
```stata
label variable hhind "individual's ID"
label variable indid "household's ID" 
label variable year "year"
label variable hhid "individual's ID in the household"
label variable surname "Surname"
label variable name "Name"
label variable gender "1 if male, 0 if female"
label variable wage "wage"
label variable location "household's location"
```
</details>

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
## location             household's location character character
```

Now that our dataframe is clean and detailed, we can compute some descriptive statistics. But before doing it, we might want to save it:


```r
write.xlsx(base_created2, file = here("Bases_Created/modified_base.xlsx")
```

<details>
<summary> Stata
</summary>
<p>
```stata
cd ..\Bases_Created  
export excel using "modified_base.xls", replace
```
</details>


## Descriptive Statistics

First of all, if we want to check the number of people per location or gender and per year, we use the **`table`** function:


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

<details>
<summary> Stata
</summary>
<p>
```stata
tab gender if year==2019  
tab location if year==2019 
```
</details>

To have more detailed statistics, you can use many functions. Here, we use the function **`describe`** from the **`Hmisc`** package


```r
describe(base_created2)
```

```
## base_created2 
## 
##  9  Variables      46  Observations
## --------------------------------------------------------------------------------
## hhind : individual's ID 
##        n  missing distinct 
##       46        0       23 
## 
## lowest : 11 12 13 14 21, highest: 71 72 81 82 83
## --------------------------------------------------------------------------------
## hhid : household's ID 
##        n  missing distinct     Info     Mean      Gmd 
##       46        0        8    0.975    4.217    2.783 
## 
## lowest : 1 2 3 4 5, highest: 4 5 6 7 8
##                                                           
## Value          1     2     3     4     5     6     7     8
## Frequency      8     8     4     2    10     4     4     6
## Proportion 0.174 0.174 0.087 0.043 0.217 0.087 0.087 0.130
## --------------------------------------------------------------------------------
## indid : individual's ID in the household 
##        n  missing distinct     Info     Mean      Gmd 
##       46        0        5    0.923    2.217    1.306 
## 
## lowest : 1 2 3 4 5, highest: 1 2 3 4 5
##                                         
## Value          1     2     3     4     5
## Frequency     16    14     8     6     2
## Proportion 0.348 0.304 0.174 0.130 0.043
## --------------------------------------------------------------------------------
## year 
##        n  missing distinct     Info     Mean      Gmd 
##       46        0        2     0.75     2020   0.5111 
##                     
## Value      2019 2020
## Frequency    23   23
## Proportion  0.5  0.5
## --------------------------------------------------------------------------------
## surname 
##        n  missing distinct 
##       46        0       23 
## 
## lowest : ANDERSON BROWN    DAVIS    DOE      JACKSON 
## highest: THOMAS   THOMPSON WHITE    WILLIAMS WILSON  
## --------------------------------------------------------------------------------
## name 
##        n  missing distinct 
##       46        0       23 
## 
## lowest : Barbara Charles Daniel  David   Donald 
## highest: Richard Robert  Susan   Thomas  William
## --------------------------------------------------------------------------------
## gender : 1 if male, 0 if female 
##        n  missing distinct     Info      Sum     Mean      Gmd 
##       46        0        2    0.715       28   0.6087    0.487 
## 
## --------------------------------------------------------------------------------
## wage 
##        n  missing distinct     Info     Mean      Gmd      .05      .10 
##       46        0       37    0.998     2059    477.4     1627     1692 
##      .25      .50      .75      .90      .95 
##     1800     1901     2098     2373     3575 
## 
## lowest : 1397 1600 1608 1683 1690, highest: 2384 3500 3600 3782 3784
## --------------------------------------------------------------------------------
## location : household's location 
##        n  missing distinct 
##       46        0        4 
##                                           
## Value      England  France   Italy   Spain
## Frequency       12      24       2       8
## Proportion   0.261   0.522   0.043   0.174
## --------------------------------------------------------------------------------
```

<details>
<summary> Stata
</summary>
<p>
```stata
sum *, detail
```
</details>

but you can also try the function **`summary`** (automatically available in base R), **`stat.desc`** in **`pastecs`**, **`skim`** in **`skimr`** or even **`makeDataReport`** in **`dataMaid`** to have a complete PDF report summarizing your data. To summarize data under certain conditions (e.g. to have the average wage for each location), you can use **`dplyr`**:


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

<details>
<summary> Stata
</summary>
<p>
```stata
tabstat wage if year==2019, stats(N mean sd min max p25 p50 p75) by(location)
tabstat wage if year==2020, stats(N mean sd min max p25 p50 p75) by(location)
```
</details>

## Plots

Finally, we want to plot some data to include in our report or article (or anything else). **`ggplot2`** is THE reference to make plots with R. The **`ggplot`** function does not create a graph but tells what is the data you are going to use and the aesthetics (**`aes`**). Here, we want to display the wages in a histogram and to distinguish them per year. Therefore, we want to fill the bars according to the year. To precise the type of graph we want, we add **`+ geom_histogram()`** after **`ggplot`**. You may change the number of **`bins`** to have a more precise histogram. 


```r
library(ggplot2)
hist1 <- ggplot(data = base_created2, 
                mapping = aes(wage, fill = factor(year))) +
  geom_histogram(bins = 10)
hist1
```

<img src="/posts/first-contact_files/figure-html/unnamed-chunk-20-1.png" width="672" />

<details>
<summary> Stata
</summary>
<p>
```stata
histogram wage if year==2019, saving(Hist1, replace) bin(10) freq title("Year 2019") ytitle("Frequency") 
histogram wage if year==2020, saving(Hist2, replace) bin(10) freq title("Year 2020") ytitle("Frequency")
```
</details>

If you prefer one histogram per year, you can use the **`facet_wrap()`** argument, as below.


```r
hist2 <- ggplot(data = base_created2, 
                mapping = aes(wage, fill = factor(year))) +
  geom_histogram(bins = 10) +
  facet_wrap(vars(year))
hist2
```

<img src="/posts/first-contact_files/figure-html/unnamed-chunk-21-1.png" width="672" />

<details>
<summary> Stata
</summary>
<p>
```stata
graph combine Hist1.gph Hist2.gph, col(2) xsize(10) ysize(5) iscale(1.5) title("{bf:Wage distribution per year}")
```
</details>

Finally, you may want to export these graphs. To do so, we use **`ggsave`** (you can replace .pdf by .eps or .png if you want): 


```r
ggsave(here("Figures/plot1.pdf"), plot = hist1)
```

<details>
<summary> Stata
</summary>
<p>
```stata
graph export Histogram1.pdf,  replace
```
</details>

That's it! In this first post, you have seen how to import, clean and tidy datasets, and how to make some descriptive statistics and some plots. I hope this was helpful to you!
