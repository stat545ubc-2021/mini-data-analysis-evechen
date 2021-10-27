Mini Data-Analysis Deliverable 3
================
Eve Chen

# Welcome to my last milestone in the mini data analysis project!

In Milestone 1, you explored your data and came up with research
questions. In Milestone 2, you obtained some results by making summary
tables and graphs.

In this (3rd) milestone, you’ll be sharpening some of the results you
obtained from your previous milestone by:

-   Manipulating special data types in R: factors and/or dates and
    times.
-   Fitting a model object to your data, and extract a result.
-   Reading and writing data as separate files.

**NOTE**: The main purpose of the mini data analysis is to integrate
what you learn in class in an analysis. Although each milestone provides
a framework for you to conduct your analysis, it’s possible that you
might find the instructions too rigid for your data set. If this is the
case, you may deviate from the instructions – just make sure you’re
demonstrating a wide range of tools and techniques taught in this class.

## Instructions

**To complete this milestone**, edit [this very `.Rmd`
file](https://raw.githubusercontent.com/UBC-STAT/stat545.stat.ubc.ca/master/content/mini-project/mini-project-3.Rmd)
directly. Fill in the sections that are tagged with
`<!--- start your work here--->`.

**To submit this milestone**, make sure to knit this `.Rmd` file to an
`.md` file by changing the YAML output settings from
`output: html_document` to `output: github_document`. Commit and push
all of your work to your mini-analysis GitHub repository, and tag a
release on GitHub. Then, submit a link to your tagged release on canvas.

**Points**: This milestone is worth 40 points (compared to the usual 30
points): 30 for your analysis, and 10 for your entire mini-analysis
GitHub repository. Details follow.

**Research Questions**: In Milestone 2, you chose two research questions
to focus on. Wherever realistic, your work in this milestone should
relate to these research questions whenever we ask for justification
behind your work. In the case that some tasks in this milestone don’t
align well with one of your research questions, feel free to discuss
your results in the context of a different research question.

# Setup

Begin by loading your data and the tidyverse package below:

``` r
library(datateachr) # <- might contain the data you picked!
library(tidyverse)
```

    ## Warning: package 'tidyverse' was built under R version 4.1.1

    ## Warning: package 'tibble' was built under R version 4.1.1

    ## Warning: package 'readr' was built under R version 4.1.1

``` r
library(broom)
```

    ## Warning: package 'broom' was built under R version 4.1.1

``` r
library(here)
```

    ## Warning: package 'here' was built under R version 4.1.1

From Milestone 2, you chose two research questions. What were they? Put
them here.

<!-------------------------- Start your work below ---------------------------->

1.  Is there a trend (increases or decreases) of the `maximum` and
    `minimum` flow rate over time (in `year` or `season`)?
2.  Is there **seasonality** to when `maximum` and `minimum` flow rates
    are usually recorded and `flow` values differs in range?
    <!----------------------------------------------------------------------------->

# Exercise 1: Special Data Types (10)

For this exercise, you’ll be choosing two of the three tasks below –
both tasks that you choose are worth 5 points each.

But first, tasks 1 and 2 below ask you to modify a plot you made in a
previous milestone. The plot you choose should involve plotting across
at least three groups (whether by facetting, or using an aesthetic like
colour). Place this plot below (you’re allowed to modify the plot if
you’d like). If you don’t have such a plot, you’ll need to make one.
Place the code for your plot below.

<!-------------------------- Start your work below ---------------------------->

-   First, let’s generate a new tibble `flow_sample_season` as last
    deliverable. This new object is from `flow_sample` and has a new
    column of four-leveled factors `season` divided by `month`.

``` r
flow_sample_season <- flow_sample %>%
  drop_na("flow") %>% #remove 2 observations with empty flow record
  mutate_if(sapply(flow_sample, is.character), as.factor) %>%  #all character variables as factors
  mutate(season = factor(case_when(month %in% c(3, 4, 5) ~ "Spring",
                                   month %in% c(6, 7, 8) ~ "Summer",
                                   month %in% c(9, 10, 11) ~ "Autumn",
                                   month %in% c(12, 1, 2) ~ "Winter"),
                         levels = c("Spring", "Summer", "Autumn", "Winter"))) #add season factor and define order of levels
```

-   Now we can generate a plot with facetted by 4 `seasons` and colored
    by different kind of conditions `sym`.

``` r
flow_sample_season  %>% 
  ggplot(aes(flow, year, colour = sym)) +
  geom_point(size = 1.5, alpha = 0.8) +
  scale_x_log10(labels = scales::label_number(suffix = "m^3/s")) +
  scale_colour_discrete(labels = c('A: partial day', 'B: Ice condtions', 'E: Estimated', 'NA: No additional info')) +
  labs(x = "Flow Rate", y = "Year", colour = "Additional info") +
  facet_wrap(~ season, ncol = 1) +
  theme_bw() 
```

![](mini-project-3_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

<!----------------------------------------------------------------------------->

Now, choose two of the following tasks.

1.  Produce a new plot that reorders a factor in your original plot,
    using the `forcats` package (3 points). Then, in a sentence or two,
    briefly explain why you chose this ordering (1 point here for
    demonstrating understanding of the reordering, and 1 point for
    demonstrating some justification for the reordering, which could be
    subtle or speculative.)

2.  Produce a new plot that groups some factor levels together into an
    “other” category (or something similar), using the `forcats` package
    (3 points). Then, in a sentence or two, briefly explain why you
    chose this grouping (1 point here for demonstrating understanding of
    the grouping, and 1 point for demonstrating some justification for
    the grouping, which could be subtle or speculative.)

3.  If your data has some sort of time-based column like a date (but
    something more granular than just a year):

    1.  Make a new column that uses a function from the `lubridate` or
        `tsibble` package to modify your original time-based column. (3
        points)
        -   Note that you might first have to *make* a time-based column
            using a function like `ymd()`, but this doesn’t count.
        -   Examples of something you might do here: extract the day of
            the year from a date, or extract the weekday, or let 24
            hours elapse on your dates.
    2.  Then, in a sentence or two, explain how your new column might be
        useful in exploring a research question. (1 point for
        demonstrating understanding of the function you used, and 1
        point for your justification, which could be subtle or
        speculative).
        -   For example, you could say something like “Investigating the
            day of the week might be insightful because penguins don’t
            work on weekends, and so may respond differently”.

<!-------------------------- Start your work below ---------------------------->

**Task Number**: 1

-   Let’s produce a new plot using `fct_inorder()` from the `forcats`
    packages, to re-order the plot based on the values of the `flow`
    rate, which orders them in the average value of `flow`. This way we
    can observe the four `seasons` from the ones with **lower** flow
    rate recorded to **higher**.

``` r
flow_sample_season  %>% 
  mutate(season = fct_reorder(season, flow))  %>% 
  ggplot(aes(flow, year, colour = sym)) +
  geom_point(size = 1.5, alpha = 0.8) +
  scale_x_log10(labels = scales::label_number(suffix = "m^3/s")) +
  scale_colour_discrete(labels = c('A: partial day', 'B: Ice condtions', 'E: Estimated', 'NA: No additional info')) +
  labs(x = "Flow Rate", y = "Year", colour = "Additional info") +
  facet_wrap(~ season, ncol = 1) +
  theme_bw() 
```

![](mini-project-3_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

<!----------------------------------------------------------------------------->
<!-------------------------- Start your work below ---------------------------->

**Task Number**: 2

-   Produce a new plot that groups all the `seasons` except for `Spring`
    into an `Other`, using the `forcats` package.

-   I used this grouping to be able to see the `flow` rate recorded
    during the `Spring` as well as the `flow` rate during other
    `seasons`, since `Spring` is the only season that contains both
    `minimum` and `maximum` flow rate. I can then be able to extrapolate
    **if there’s a year when both `minimum` and `maximum` rate was
    recorded in `Spring`, and if the records taken in such close date
    would make the value different from the others**.

``` r
flow_sample_season  %>% 
  mutate(season = fct_reorder(season, flow),
         season = fct_other(season, keep = "Spring", other_level = "Other"))  %>% 
  ggplot(aes(flow, year, colour = sym)) +
  geom_point(size = 1.5, alpha = 0.8) +
  scale_x_log10(labels = scales::label_number(suffix = "m^3/s")) +
  scale_colour_discrete(labels = c('A: partial day', 'B: Ice condtions', 'E: Estimated', 'NA: No additional info')) +
  labs(x = "Flow Rate", y = "Year", colour = "Additional info") +
  facet_wrap(~ season, ncol = 2) +
  theme_bw() 
```

![](mini-project-3_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

-   From the plot we see that there are years when both records are
    taken in `Spring`. Values of minimum or maximum `flow` vary less (in
    narrower range) compared to the other seasons. There are no records
    with additional information as `A: partial day` or `E: Estimated` in
    `Spring`.

<!----------------------------------------------------------------------------->

# Exercise 2: Modelling

## 2.0 (no points)

Pick a research question, and pick a variable of interest (we’ll call it
“Y”) that’s relevant to the research question. Indicate these.

<!-------------------------- Start your work below ---------------------------->

**Research Question**: Is there a trend (increases or decreases) of the
`maximum` flow rate over `year`?

**Variable of interest**: Maximum `flow` rate and `year`

<!----------------------------------------------------------------------------->

## 2.1 (5 points)

Fit a model or run a hypothesis test that provides insight on this
variable with respect to the research question. Store the model object
as a variable, and print its output to screen. We’ll omit having to
justify your choice, because we don’t expect you to know about model
specifics in STAT 545.

-   **Note**: It’s OK if you don’t know how these models/tests work.
    Here are some examples of things you can do here, but the sky’s the
    limit.
    -   You could fit a model that makes predictions on Y using another
        variable, by using the `lm()` function.
    -   You could test whether the mean of Y e quals 0 using `t.test()`,
        or maybe the mean across two groups are different using
        `t.test()`, or maybe the mean across multiple groups are
        different using `anova()` (you may have to pivot your data for
        the latter two).
    -   You could use `lm()` to test for significance of regression.

<!-------------------------- Start your work below ---------------------------->

-   I decided use the lm() function to test for significance of
    regression for `flow` based on the `year`. I put this into the model
    `lmfit`.

``` r
(lmfit <- lm(flow ~ year, data = filter(flow_sample, extreme_type == "maximum")))
```

    ## 
    ## Call:
    ## lm(formula = flow ~ year, data = filter(flow_sample, extreme_type == 
    ##     "maximum"))
    ## 
    ## Coefficients:
    ## (Intercept)         year  
    ##     946.202       -0.374

-   We can graph it to see what the significance looks like.

``` r
filter(flow_sample, extreme_type == "maximum") %>% 
  ggplot(aes(year, flow)) +
  geom_point() +
  geom_smooth(method="lm") +
  scale_x_log10()
```

    ## `geom_smooth()` using formula 'y ~ x'

![](mini-project-3_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

<!----------------------------------------------------------------------------->

## 2.2 (5 points)

Produce something relevant from your fitted model: either predictions on
Y, or a single value like a regression coefficient or a p-value.

-   Be sure to indicate in writing what you chose to produce.
-   Your code should either output a tibble (in which case you should
    indicate the column that contains the thing you’re looking for), or
    the thing you’re looking for itself.
-   Obtain your results using the `broom` package if possible. If your
    model is not compatible with the broom function you’re needing, then
    you can obtain your results by some other means, but first indicate
    which broom function is not compatible.

<!-------------------------- Start your work below ---------------------------->

-   I chose to produce the **p value** from the fitted model
    `maximum flow` rate and `year`. This outputted a tibble, and the p
    value is under the 5th column `p.calue`. I obtained my results using
    the `broom` package function `tidy()`.

``` r
tidy(lmfit)
```

    ## # A tibble: 2 x 5
    ##   term        estimate std.error statistic p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)  946.      363.         2.60  0.0105
    ## 2 year          -0.374     0.185     -2.02  0.0458

-   The p value for the `intercept` is `0.010`, and `0.046` for `year`.
    They are **less than 0.05**. This means that this model that can be
    intepreted as `flow = 946.20 - 0.37 year` fits the data well.

-   To answer the research question: Since the estimated slope of `year`
    is `-0.37 < 0`, we can say that the `maximum flow` is **decreasing**
    over `year`.

<!----------------------------------------------------------------------------->

# Exercise 3: Reading and writing data

Get set up for this exercise by making a folder called `output` in the
top level of your project folder / repository. You’ll be saving things
there.

## 3.1 (5 points)

Take a summary table that you made from Milestone 2 (Exercise 1.2), and
write it as a csv file in your `output` folder. Use the `here::here()`
function.

-   **Robustness criteria**: You should be able to move your Mini
    Project repository / project folder to some other location on your
    computer, or move this very Rmd file to another location within your
    project repository / folder, and your code should still work.
-   **Reproducibility criteria**: You should be able to delete the csv
    file, and remake it simply by knitting this Rmd file.

<!-------------------------- Start your work below ---------------------------->

-   I took the summary table where I compute the *range* (*max*, *min*
    and contrast) , *mean* and *median* of `flow` across the groups of
    `extream_type` from the data, to observe the overall range and
    statistical features of the flow rate recorded as different type.
    This was then saved as a csv file onto the `output` folder in the
    main directory. This was done via the `write_csv` and `here::here()`
    function.

``` r
# Create the summary table
(flow_extreme_type <- flow_sample %>%
   drop_na() %>%
   group_by(extreme_type) %>% 
   summarise(flow_maximum = max(flow),
             flow_minimum = min(flow),
             flow_range = max(flow) - min(flow),
             flow_mean = mean(flow),
             flow_median = median(flow),
             n = n()))
```

    ## # A tibble: 2 x 7
    ##   extreme_type flow_maximum flow_minimum flow_range flow_mean flow_median     n
    ##   <chr>               <dbl>        <dbl>      <dbl>     <dbl>       <dbl> <int>
    ## 1 maximum            220          146         74       171         147        3
    ## 2 minimum              8.44         3.62       4.82      6.27        6.17    96

``` r
# Write it into a csv file and save in the "output" folder
write_csv(flow_extreme_type, here::here("output", "flow_extreme_type.csv"))
```

<!----------------------------------------------------------------------------->

## 3.2 (5 points)

Write your model object from Exercise 2 to an R binary file (an RDS),
and load it again. Be sure to save the binary file in your `output`
folder. Use the functions `saveRDS()` and `readRDS()`.

-   The same robustness and reproducibility criteria as in 3.1 apply
    here.

<!-------------------------- Start your work below ---------------------------->

``` r
# Create the summary table
(flow_extreme_type <- flow_sample %>%
   drop_na() %>%
   group_by(extreme_type) %>% 
   summarise(flow_maximum = max(flow),
             flow_minimum = min(flow),
             flow_range = max(flow) - min(flow),
             flow_mean = mean(flow),
             flow_median = median(flow),
             n = n()))
```

    ## # A tibble: 2 x 7
    ##   extreme_type flow_maximum flow_minimum flow_range flow_mean flow_median     n
    ##   <chr>               <dbl>        <dbl>      <dbl>     <dbl>       <dbl> <int>
    ## 1 maximum            220          146         74       171         147        3
    ## 2 minimum              8.44         3.62       4.82      6.27        6.17    96

``` r
# Write and load an RDS file 
saveRDS(flow_extreme_type, here("output", "flow_extreme_type.rds"))
readRDS(here("output", "flow_extreme_type.rds"))
```

    ## # A tibble: 2 x 7
    ##   extreme_type flow_maximum flow_minimum flow_range flow_mean flow_median     n
    ##   <chr>               <dbl>        <dbl>      <dbl>     <dbl>       <dbl> <int>
    ## 1 maximum            220          146         74       171         147        3
    ## 2 minimum              8.44         3.62       4.82      6.27        6.17    96

<!----------------------------------------------------------------------------->

# Tidy Repository

Now that this is your last milestone, your entire project repository
should be organized. Here are the criteria we’re looking for.

## Main README (3 points)

There should be a file named `README.md` at the top level of your
repository. Its contents should automatically appear when you visit the
repository on GitHub.

Minimum contents of the README file:

-   In a sentence or two, explains what this repository is, so that
    future-you or someone else stumbling on your repository can be
    oriented to the repository.
-   In a sentence or two (or more??), briefly explains how to engage
    with the repository. You can assume the person reading knows the
    material from STAT 545A. Basically, if a visitor to your repository
    wants to explore your project, what should they know?

Once you get in the habit of making README files, and seeing more README
files in other projects, you’ll wonder how you ever got by without them!
They are tremendously helpful.

## File and Folder structure (3 points)

You should have at least four folders in the top level of your
repository: one for each milestone, and one output folder. If there are
any other folders, these are explained in the main README.

Each milestone document is contained in its respective folder, and
nowhere else.

Every level-1 folder (that is, the ones stored in the top level, like
“Milestone1” and “output”) has a `README` file, explaining in a sentence
or two what is in the folder, in plain language (it’s enough to say
something like “This folder contains the source for Milestone 1”).

## Output (2 points)

All output is recent and relevant:

-   All Rmd files have been `knit`ted to their output, and all data
    files saved from Exercise 3 above appear in the `output` folder.
-   All of these output files are up-to-date – that is, they haven’t
    fallen behind after the source (Rmd) files have been updated.
-   There should be no relic output files. For example, if you were
    knitting an Rmd to html, but then changed the output to be only a
    markdown file, then the html file is a relic and should be deleted.

Our recommendation: delete all output files, and re-knit each
milestone’s Rmd file, so that everything is up to date and relevant.

PS: there’s a way where you can run all project code using a single
command, instead of clicking “knit” three times. More on this in STAT
545B!

## Error-free code (1 point)

This Milestone 3 document knits error-free. (We’ve already graded this
aspect for Milestone 1 and 2)

## Tagged release (1 point)

You’ve tagged a release for Milestone 3. (We’ve already graded this
aspect for Milestone 1 and 2)
