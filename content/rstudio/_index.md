---
title: "Prikazy v rstudiu"
date: 2023-03-04T17:57:12+01:00
draft: false
---


```{r include=FALSE}
library(tidyverse)
library(magrittr)
```

## Data Wrangling

Data wrangling process consists of reorganizing, transforming and mapping data from one "raw" form into another in order to make it more usable and valuable for a variety of downstream uses including machine learning.

### Setup

Tidyverse is an opinionated collection of R packages designed for data wangling and analytics. All packages share an underlying design philosophy, grammar, and data structures.

Install the complete tidyverse with: `install.packages("tidyverse")`, which should provide you access to function implemented in packages readr, tidyr, dplyr, ggplor2, tibbles, purrr and many others. Among these,

-   **readr** provides a fast and friendly way to read rectangular data (like csv, tsv, and fwf). It is designed to flexibly parse many types of data found in the wild, while still cleanly failing when data unexpectedly changes.
-   **tibble** is a modern re-imagining of the data frame, keeping what time has proven to be effective, and throwing out what it has not. Tibbles are data.frames that are lazy and surly: they do less and complain more forcing you to confront problems earlier, typically leading to cleaner, more expressive code.\
-   **tidyr** provides a set of functions that help you get to tidy data. Tidy data is data with a consistent form: every variable goes in a column, and every column is a variable.
-   **dplyr** provides a grammar of data manipulation, providing a consistent set of verbs that solve the most common data manipulation challenges.

### Reading rectangular data

In this practicum, we shall investigate FIFA22 dataset from Kaggle. These data will serve for in-depth comparison of player stats across different playing positions.

To load data into R, we will simply call `readr::read_csv`with players_22.csv and the location from which that file should be read: `data <- read_csv(file = "/Users/micha/OneDrive/Documents/Data Science Programming/Data/players_22.csv", num_threads = 4)` The data will be then loaded as tibble --- a reworked version of data frame that is accessible under the name of *data*.

```{r include=FALSE}
library(tidyverse)
data <- read_csv(file = "/Users/micha/OneDrive/Documents/Data Science Programming/Data/players_22.csv", num_threads = 4) 
```

### Defining hypotheses

According to the statistics, 80% of professional football players are right footed, which means that left footed players are something of a rarity. However, when we consider some of the greatest names to play the game, we do start to see a trend that some of the best players --- Lionel Messi, Ferenc Puskas and Diego Maradona --- have ever have been lefty's. 

So what makes the left footer different, or more special? Is it just that they do things differently, or is it a case of something else at play? To gain local insights into the topic, we shall explore Slovak and Czech left-footed players by asking questions like:

-   **Task 1:** What is the prevalence of left/right footed Slovak and Czech players in `players_22.csv`,
-   **Task 2:** support your findings with a table showing the number of left and right footed players for each nation. Ideally, variable data (\# or players, etc.) are stored in columns, whilst observational data (nationalities) are stored in rows.
-   **Task 3:** explore how left and right-footed players differ in terms of player skills.

### dplyr package

To perform tasks 1 and 2, **dplyr** provides a set of tools for efficient manipulation of datasets, eventually going beyond simple subsetting operations. Our primary interest are:

```{r echo=FALSE, out.width='70%', fig.retina=T}
knitr::include_graphics('Figures/Dplyr.png')
```

### Pushing data between ops

A common practice in R is not to call **dplyr** functions individually. Instead, we build data processing workflows using pipes. Starting with raw data, the pipe operator **%\>%** pushes left-hand side values forward into expressions that appear on the right-hand side, thus replacing `f(x)` with the `x %>% f()` notation.

### Extracting variables with select

```{r include=FALSE}
library(tidyverse)
library(magrittr)
data <- read_csv(file = "/Users/micha/OneDrive/Documents/Data Science Programming/Data/players_22.csv") 
```

It is not uncommon to get data sets with hundreds or even thousands of variables. Since this is the case for `players_22.csv`, the first challenge is narrowing in on the variables that are we actually interested in. Function `dplyr::select()` allows us to rapidly zoom in on selected slices of data after providing the names of the variables.

To work with less data on next iterations, we select the appropriate columns like this.

```{r}
data %>% select(short_name, 
                nationality_name,
                preferred_foot,
                player_positions,
                overall, potential, wage_eur) %>% 
          print(n=3)
```

There are a number of helper functions you can use within `select()`. For example, you can add additional columns using regular expression matching. Let's do that with all columns that start with *skills*.

```{r echo=FALSE, out.width='70%', fig.retina=T}
knitr::include_graphics('Figures/Dplyr select.png')
```

```{r}
data %>% select(short_name, nationality_name, starts_with("skill")) %>% 
          print(n=3)
```

Conveniently, another interesting option is `select()`-ing columns in conjunction with the `everything()`. This can be useful if we have a handful of variables we would like to move to the start of the data frame:

```{r}
data %>% select(short_name, nationality_name, everything()) %>% 
          print(n=3)
```

The way to think about `select()` is that that expression eventually evaluates to a logical vector. So if you use `starts_with()` it goes through the variables in the tibble and asks whether the variable name starts with the right set of characters.

### Extracting observations with filter

`filter()` allows you to select observations based on their values. The first argument is the name of the data frame, which we normally omit when piping. The second argument of `filter()` is the list of expressions that are normally used for predicate filtering. For example, we can select all Slovak players with:

```{r}
data %>% select(short_name, nationality_name, starts_with("skill")) %>%
         filter(nationality_name == "Slovakia" | nationality_name == "Czech Republic") %>%
         print(n=3)
```

R either prints out the results, or saves them to a variable. If you want to do both, you can wrap the assignment in parentheses: `SK <- filter(data, nationality_name == "Slovakia")`.

Multiple arguments to `filter()` are combined with "& (and)", implying that every expression must be true in order for a row to be included in the output. Note, that you cannot simply write `data %>% filter(month == 11 | 12)`, which literally translates into English: "finds all flights that departed in November or December." Instead, you have to write it always in full: `data %>% filter(month == 11 | month == 12)`.

A useful shorthand for this problem is `x %in% y`. This will select every row where x is one of the values in y. We could use it to rewrite the preceding code:

```{r}
data %>% select(short_name, nationality_name, starts_with("skill")) %>%
         filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
         print(n=3)
```

### Aggregating data summaries

Data aggregation is a process whereby data is gathered and expressed in a summary form. When data is aggregated, atomic data rows -- typically gathered from multiple sources -- are replaced with totals or summary statistics. 

Aggregate data is typically found in a data warehouse, as it can provide answers to analytical questions and also dramatically reduce the time to query large sets of data.

In R, summary statistics are generated using the `summarise()` function. Groups of observed aggregates are replaced with summary statistics based on those observations. Following our case, say, we want to get the counts of Slovak and Czech players.

```{r}
data %>% select(short_name, nationality_name, starts_with("skill")) %>%
         filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
         summarise(Count =n())

```

### Excercises

-   Check the type of the output of `summarise()` from the previous example. Does it have any attributes and can those attributes be changed?
-   Use R help to find the way how to convert a tibble output to a numeric vector. 
-   However, getting raw counts --- aka number of lines --- for soccer players who fulfill the condition is not particularly helpful. It would be better to break stats down by nationality. We can do that by placing `group_by(nationality_name, preferred_foot)` before `summarise()`.  

### tidyr package

```{r echo=FALSE, message=FALSE, warning=FALSE}
data %>% select(short_name, nationality_name, 
                preferred_foot, player_positions) %>%
  filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
  group_by(nationality_name, preferred_foot) %>%
  summarise(count = n())
```

You will have noticed that the output is not right. It does contain the requested information, true, but the information is not arranged the way needed for easy data manipulation and extraction. 

Ideally, we would prefer having separate columns for left and right-footed players, whilst observation counts for different nationalities in rows.

To get your data into shape, you can use one of two *pivoting* functions from **tidyr** package. Tidyr includes a function called `pivot_wider()` that makes our data frame longer by increasing the number of columns and decreasing the number of rows. It’s relatively rare you need `pivot_wider()` to make tidy data, but it’s often useful for creating *summary tables* for presentation. 

```{r message=FALSE, warning=FALSE}
d<- data %>% select(short_name, nationality_name, preferred_foot, player_positions) %>%
  filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
  group_by(nationality_name, preferred_foot) %>%
  summarise(count = n()) %>%
  pivot_wider(names_from = preferred_foot, values_from = count) 
d
```

An inverse transformation to `pivot_wider()` is `pivot_longer()` that "lengthens" data, thus increasing the number of rows and decreasing the number of columns.  

```{r message=FALSE, warning=FALSE}
d %<>% pivot_longer(cols = c("Left", "Right"), names_to = "pref.foot", values_to = "Counts") ; d
```

### Transposing tables with t()

If you still do not like how the table is set... well turn it upside down by transposing. Transposing swaps columns with rows and rows with columns. There are several functions for this job, but normally use `t()` function provided by the basic installation of the R programming language. 

```{r message=FALSE, warning=FALSE}
t(d)
```

### Adding new columns

`mutate()` always adds new columns at the end of your data set. If we only want to keep the new variables, we use `transmute()`. Conversely, `mutate_if()` and `mutate_at()` can be used as **sed()** replacement in R.

There are many functions for creating new variables that we can use with `mutate()`. The **key property is that the function must be vectorized**: it must take a vector of values as input, and return a vector with the same number of values as output.

* Arithmetic operators +, -, *, /, ^ : These are all vectorized, using the so-called “recycling rules.” If one parameter is shorter than the other, it will be automatically extended to be the same length. This is most useful when one of the arguments is a single number.
* Modular arithmetic (%/% and %%): %/% (integer division) and %% (remainder), where x == y * (x %/% y) + (x %% y). Modular arithmetic is a handy tool because it allows you to break integers into pieces.

To demonstrate the how `mutate()` works, let's calculate total number of player for each nation and determine the proportion of left footed.

```{r echo=TRUE, message=FALSE, warning=FALSE}
data %>% select(short_name, nationality_name, preferred_foot, player_positions) %>%
  filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
  group_by(nationality_name, preferred_foot) %>%
  summarise(count = n()) %>%
  pivot_wider(names_from = preferred_foot, values_from = count) %>%
  mutate(Total = Left + Right, Proportion = Left / Total)
```

### Renaming columns

`rename()` function is available in the dplyr package, which is used to change the particular column name present in the data frame. Say, we want to rename *nationality_name* column to *Nationality*, we can do that with `rename()` easily.

```{r echo=TRUE, message=FALSE, warning=FALSE}
data %>% select(short_name, nationality_name, preferred_foot, player_positions) %>%
  filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
  group_by(nationality_name, preferred_foot) %>%
  summarise(count = n()) %>%
  pivot_wider(names_from = preferred_foot, values_from = count) %>%
  mutate(Total = Left + Right, Proportion = Left / Total) %>%
  rename(Nationality = nationality_name)
```

### Excercises

-   **Task 4:** In some cases, you may want to replace `summarise()` by `table()`.

Find the way to add the missing columns (Total and Proportion of left footed players) to the table shown below.

```{r}
data %>% select(nationality_name, preferred_foot) %>%
  filter(nationality_name %in% c("Slovakia", "Czech Republic")) %>%
  table() 
```

-   **Task 5:** Use table that shows the number of Slovak and Czech left/right-footed players. Append column summaries (e.g. sum, median, min, max, etc.) to the table as the bottom row(s). 
-   **Task 6:** Retrieve positions at which Slovak left footed players play. 
-   **Task 7:** Present your result as in a tabular form with counts for each position. Only one playing position per column is allowed. Convert empty cells/NAs to 0s. 
-   **Task 8:** Left footers are thought to be more creative offensive players. But how do they fare as defensive players? Let's find the difference in their defensive rating (column defending) and split the resulting data according to player's height. Compare righties and lefties.  
-   **Task 9:** Find Slovak and Czech left footed players whose data show the biggest difference between their overall and potential scores. These guys will be the prime targets for any scout. Be gentleman and sort the resulting table in a descending order. 
-   **Task 10:** Which Slovak and Czech left footed players have the highest BMI and at which position do they play? 
