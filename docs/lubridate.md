# Dates and times: lubridate {#lubridate}

Resources:

- [Lubridate homepage](https://lubridate.tidyverse.org/)
- [Cheatsheet](https://rawgit.com/rstudio/cheatsheets/master/lubridate.pdf)
- [Book Chapter in R4DS](https://r4ds.had.co.nz/dates-and-times.html)
- [Vignette](https://cran.r-project.org/web/packages/lubridate/vignettes/lubridate.html)

Suggested data set: weather-kiel-holtenau

## What is Lubridate?
Lubridate is an R-Package designed to ease working with date/time variables. These can be challenging in baseR and lubridate allows for frictionless, lubricious working with dates and times, hence the name.

Lubridate is part of the tidyverse package, but can be installed separately as well. It probably reveals most of its usefullness in collaboration with other tidyverse packages. A useful extension, depending on your data, might be the time-series package which is not part of tidyverse.

All mentioned packages can be optained with the following commands in the RStudio consol: 

* `install.packages("lubridate")`
* `install.packages("tidyverse")`
* `install.packages("testit")`

If they have already been installed previously in your environment, they might have to be called using `library(tidyverse)` and so forth - see code chunks below.

## Basics

Some examples of real world date-time formats found in datasets:

How people talk about dates and times often differs from the notation. Depending on the specific use of the data, the given information might be more or less granular. 
When people in the USA talk distance between two places, they often give an approximation of how long it will take for a person to drive from A to B and round up or down to the hour. 

Flight schedules will most likely be exact to the minute, while some sensor data will probably need to be exact to the second. So there will be differing granularity in date and time data. 

Even if this would not be challenging, we still would have to deal with different notations of date and time. People in Germany will write a day-date like: *dd.mm.yyyy* or short dd.mm.yy, while the anglo-saxon realm will use *mm.dd.yyyy* frequently and the most chronologically sound way would be to use *yyyy.mm.dd*, but this doesn't seem to stick with humans. 

On top of these issues there's the fact that time itself does not make the impression of being the most exact parameter out there. Physical time might appear linear, but the way our planet revolves within our galaxy has made it neccessary to adjust date and times every now and then, so our calendar stays in tune with defined periods and seasons. This creates leap years, skipped seconds, daylight-savings time and last, but not least time-zones, which can mess things up even further. After defining date-time data as distinct *"places in time"* often the need for calculation with them arises.

## Import data, clean date-time
We will apply the lubridate package to weather data from on stationary sensor in northern Germany, the weather station in Kiel-Holtenau to be more exact.

Before we introduce the library, the technical prerequisites must be created.

```r
library(readr)          # part of the tidyverse
library(lubridate)      # the mentioned lubrication for date-time wrangling
library(tidyverse)      # the tidyverse with its dplyr functions for data wrangling
library(ggplot2)        # data visualisation package (is actually part of tidyverse, but still)
library(testit)         # testing data selections
```

With the lubridate package activated, we can now call some simple functions to check where we are in time:


```r
# Lubridate
today()                 # date like YYYY-mm-dd
```

```
## [1] "2020-05-14"
```

```r
# Lubridate
now()                   # timestamp like YYYY-mm-dd HH:MM:SS TZ
```

```
## [1] "2020-05-14 09:15:59 CEST"
```

```r
# Base R
Sys.Date()              # date like YYYY-mm-dd
```

```
## [1] "2020-05-14"
```

```r
# Base R
Sys.time()              # timestamp like YYYY-mm-dd HH:MM:SS TZ
```

```
## [1] "2020-05-14 09:15:59 CEST"
```

```r
# Base R
Sys.timezone()          # actual timezone "Europe/Berlin"
```

```
## [1] "Europe/Berlin"
```
The function `today()` delivers a date, while `now()` delivers a date-time (a date + a time).
The third is a baseR function that produces the date, the operating system sees as current, while the fourth displays the systems full current timestamp. 
The first two functions are lubridate functions, which behave just nicer and allow for fluent calculations, while the subseqeunt functions are  baseR and therefore case sensitive.

Now let's have a look at some real world data and the challenges of date-time formatting.

A first step is to import the data, which is given in a csv-format. 
We will use the tidyverse version of `read_csv()` to accomplish this step.

The data will be called `df` (data.frame) in order to allow for briefer referencing in the code further down.

This is the simple approach to read a file with suffix csv.

```r
df <- read_csv("data/weather_kiel_holtenau.csv")
```

```
## Parsed with column specification:
## cols(
##   STATIONS_ID = col_double(),
##   MESS_DATUM = col_double(),
##   TEMPERATUR = col_double(),
##   RELATIVE_FEUCHTE = col_double(),
##   NIEDERSCHLAGSDAUER = col_double(),
##   NIEDERSCHLAGSHOEHE = col_double(),
##   NIEDERSCHLAGSINDIKATOR = col_double()
## )
```

```r
head(df, 5)
```

```
## # A tibble: 5 x 7
##   STATIONS_ID MESS_DATUM TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl>      <dbl>      <dbl>            <dbl>            <dbl>
## 1        2564    2.02e11        3.6             75.6                0
## 2        2564    2.02e11        3.6             75.6                6
## 3        2564    2.02e11        3.6             76.5                0
## 4        2564    2.02e11        3.5             75.6                0
## 5        2564    2.02e11        3.6             75.3                0
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```
All columns are recognized as double values.

By looking at the `MESS_DATUM` column it becomes apparent that this represents a timestamp that was created every 10 minutes. The standard column type definition of the import tool was not able to format this appropriately, which is why the column will be defined as a double for now.

Using the code generator of the data import tool it is possible to format the variables in the appropriate classes. The code snippet is automatically generated by the import tool of RStudio. For our concern of the timestamp variable please note the part `MESS_DATUM = col_datetime(format = "%Y%m%d%H%M"), ...` which will reach our present goal to format the timestamp and class as a POSIXct.


```r
df <- read_csv("data/weather_kiel_holtenau.csv",
               col_types = cols(MESS_DATUM = col_datetime(format = "%Y%m%d%H%M"),
                                NIEDERSCHLAGSDAUER = col_integer(),
                                NIEDERSCHLAGSINDIKATOR = col_integer(),
                                STATIONS_ID = col_integer()))

head(df, 5)
```

```
## # A tibble: 5 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <int> <dttm>                   <dbl>            <dbl>            <int>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <int>
```

We will examine the data further down below. At this point it should be mentioned again that we generated the code to import the data by using the import readr tool of RStudio. 
This allows a first look at the raw csv data before importing and some tweaking of the variables and their classes. We left the locale portion of the import tool untouched.

The following lines of code import the same data in the `MESS_DATUM` column as a string of characters again and then call a specific lubridate function (`ymd()`[<sup>1</sup>](#lubridate-footnote)<a name="lubridate-foot-target"></a>) which basically recognizes the time format in the column given only very little information. 
In our case we only specify the order in which the date-time information is given, which is Year-Month-Day-Hour-Minute. The `ymd_hm()` function delivers the desired result.
This function however only works if it is applied to a character string, which is why we chose to overwrite the dataset `df` with a new import procedure in which we explicitly assign the column `MESS_DATUM` to be of type character after import to than be able to transform it into a date-time.

<a name="lubridate-footnote"></a>TEST<sub>[1](#lubridate-foot-target). `ymd` stands for year-month-day</sub>

```r
df1 <- read_csv("data/weather_kiel_holtenau.csv",
                col_types = cols(MESS_DATUM = col_character(),
                                 NIEDERSCHLAGSINDIKATOR = col_integer(),
                                 STATIONS_ID = col_integer()))

df1$MESS_DATUM <- ymd_hm(df1$MESS_DATUM)
head(df1, 5)
```

```
## # A tibble: 5 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <int> <dttm>                   <dbl>            <dbl>            <dbl>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <int>
```
As can be seen above, the `MESS_DATUM` is now in a POSIXct format again, which is what will be needed for further calculations and analysis of the data set.

Alternatively you could use the following parsing function to achieve the same result. Note that the timestamp needs to be of type character for the `parse_date_time()` function of lubridate, too.

```r
df2 <- read_csv("data/weather_kiel_holtenau.csv",
                col_types = cols(MESS_DATUM = col_character()))

df2$MESS_DATUM <- parse_date_time(df2$MESS_DATUM, orders ="Ymd HM")
head(df2, 5)
```

```
## # A tibble: 5 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl> <dttm>                   <dbl>            <dbl>            <dbl>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```

OK, we have successfully formated the time-stamp data into a productive date-time (dttm) format using three different aproaches.

## Check and modify data
The next steps are a check and elimination procedure to eliminate missing values (NAs) from the dataset,
if some observations might have failed to generate a timestamp.

```r
df %>% 
  filter(is.na(MESS_DATUM)) %>%                   # Checking if there are observations with a missing MESS_DATUM
  head()
```

```
## # A tibble: 0 x 7
## # ... with 7 variables: STATIONS_ID <int>, MESS_DATUM <dttm>, TEMPERATUR <dbl>,
## #   RELATIVE_FEUCHTE <dbl>, NIEDERSCHLAGSDAUER <int>, NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <int>
```

We don't encounter any NAs in the `MESS_DATUM` column. Eliminating NAs from the `MESS_DATUM` column including the other variables of that specific observations from the data frame would have been possible with: 

```r
df <- df %>% 
  filter(!is.na(MESS_DATUM))                      # missing MESS_DATUM observations (NAs) would/will be excluded from the data frame
```

Now it is time to have a first explorative look at the data set.

```r
max(df$MESS_DATUM)                                # The latest observation is dated 10 minutes before midnight on the 13th of April 2020
```

```
## [1] "2020-04-13 23:50:00 UTC"
```

```r
min(df$MESS_DATUM)                                # The earliest observation is dated 14th of April 2019
```

```
## [1] "2019-04-14 UTC"
```

```r
range(df$MESS_DATUM)                              # Another way to get the same information
```

```
## [1] "2019-04-14 00:00:00 UTC" "2020-04-13 23:50:00 UTC"
```

We are dealing with data over the course of one year starting on midnight 14th April 2019 until 10 minutes to midnight on 13th April 2020.

To gather some information on the other variables we can use the `range()` function again.

```r
range(df$STATIONS_ID)                             # This confirms: We are only dealing with data from one and only one weather sensor in the whole data set.
```

```
## [1] 2564 2564
```

```r
range(df$TEMPERATUR)                              # We have a range of temperatures between -2.5 and +32.1 degrees  celius, which sounds about plausible.
```

```
## [1] -2.5 32.1
```

```r
range(df$RELATIVE_FEUCHTE)                        # Delivers an unexpected range, where the minimum is -999 and the maximum is +100. Both values are either not possible (-999) or only of theoretic value (+100)
```

```
## [1] -999  100
```

```r
range(df$NIEDERSCHLAGSDAUER)                      # This delivers an implausible minimum of -999 and an expected maximum of 10 (the maximum amount of minutes in a 10 minute interval).
```

```
## [1] -999   10
```

```r
range(df$NIEDERSCHLAGSHOEHE)                      # Again the range shows a minimum at -999 and a maximum of 7.85 which has to be interpreted as mm which equals litres per square meter (6 x 7.85 = 47.1/h = torrential)
```

```
## [1] -999.00    7.85
```

```r
range(df$NIEDERSCHLAGSINDIKATOR)                  # A look at the data reveals a binary 1 or a 0 for this variable, i.e. it rained or it didn't. -999 must be interpreted as a failed observation
```

```
## [1] -999    1
```

Here we find a so called *fill value* of `-999` or `-999.00` for some observations in a number of variables. These observations are spurious and would have to be deleted from the table to obtain operable data.
Theoretically, the following columns are only relevant if the `NIEDERSCHLAGSINDIKATOR` is set to 1, indicating precipitation.
This column can be considered like a switch for the other two precipitation-related columns:
* `NIEDERSCHLAGSDAUER`
* `NIEDERSCHLAGSHOEHE`

Getting an idea about the extent of weird/failed observations can be accomplished with the following code snippet:

```r
values_of_ind <- df %>% 
  group_by(NIEDERSCHLAGSINDIKATOR) %>%
  tally()    # count the occurences of the distinct values of NIEDERSCHLAGSINDIKATOR
        
head(values_of_ind)
```

```
## # A tibble: 3 x 2
##   NIEDERSCHLAGSINDIKATOR     n
##                    <int> <int>
## 1                   -999   127
## 2                      0 43290
## 3                      1  9287
```

So 127 observations of precipitation are faulty. To modify the `NIEDERSCHLAGSINDIKATOR` for upcoming analysis we do the following in order to set the -999 Values to a more neutral Zero:

```r
df$NIEDERSCHLAGSINDIKATOR <- ifelse(df$NIEDERSCHLAGSINDIKATOR == -999, 0, df$NIEDERSCHLAGSINDIKATOR)

value_of_ind_n <- df %>% 
  group_by(NIEDERSCHLAGSINDIKATOR) %>%
  tally()

head(value_of_ind_n)
```

```
## # A tibble: 2 x 2
##   NIEDERSCHLAGSINDIKATOR     n
##                    <dbl> <int>
## 1                      0 43417
## 2                      1  9287
```

We now have a binary indicator of precipitation. Over the course of the observed year precipiation was observed for $9287/43417 = 21.4\%$ of the time frames.

The column `RELATIVE_FEUCHTE` must also be adjusted in the same way.

```r
values_of_rel <- df %>% 
  group_by(RELATIVE_FEUCHTE) %>%
  filter(RELATIVE_FEUCHTE < 0) %>%
  tally()

head(values_of_rel)
```

```
## # A tibble: 1 x 2
##   RELATIVE_FEUCHTE     n
##              <dbl> <int>
## 1             -999   128
```

The negative values are set to 0 again.

```r
df$RELATIVE_FEUCHTE <- ifelse(df$RELATIVE_FEUCHTE == -999, 0, df$RELATIVE_FEUCHTE)

values_of_rel_n <- df %>% 
  group_by(RELATIVE_FEUCHTE) %>%
  tally()

head(values_of_rel_n)
```

```
## # A tibble: 6 x 2
##   RELATIVE_FEUCHTE     n
##              <dbl> <int>
## 1              0     128
## 2             25       1
## 3             25.4     1
## 4             25.5     1
## 5             25.8     1
## 6             26.3     1
```

The column `NIEDERSCHLAGSHOEHE` can also be tested and adjusted similarly.
This is not strictly necessary since the negative values could be hidden from further analysis using the `NIEDERSCHLAGSINDIKATOR`.

```r
values_of_ARN <- df %>% 
  group_by(NIEDERSCHLAGSHOEHE) %>%
  filter(NIEDERSCHLAGSHOEHE < 0) %>%
  tally()

head(values_of_ARN)
```

```
## # A tibble: 1 x 2
##   NIEDERSCHLAGSHOEHE     n
##                <dbl> <int>
## 1               -999   129
```

The negative values are set to 0 again.

```r
df$NIEDERSCHLAGSHOEHE <- ifelse(df$NIEDERSCHLAGSHOEHE == -999, 0, df$NIEDERSCHLAGSHOEHE)

values_of_RFH <- df %>% 
  group_by(NIEDERSCHLAGSHOEHE) %>%
  tally()

head(values_of_RFH)
```

```
## # A tibble: 6 x 2
##   NIEDERSCHLAGSHOEHE     n
##                <dbl> <int>
## 1               0    47753
## 2               0.01   712
## 3               0.02   343
## 4               0.03   377
## 5               0.04   397
## 6               0.05   234
```

We have set the variable `NIEDERSCHLAGSHOEHE` to 0 in case of these 129 oberservations. The bias that is introduced by this modification can be called negligible in the light of a total 47,753 observations.

## Before exploration
We now have clean and tidy data and can begin with general exploration, analysis and interpretation.

The `TEMPERATUR` is given in degrees Celsius, the `RELATIVE_FEUCHTE` is a percentage Value for humidity which refers to the amount of water vapor present in the air relative to the maximum amount of water vapor the air could hold before at a given temperature. 
As the temperature increases, the air can hold larger amounts of water vapor, hence the relativity of this variable. This relation is referred to as the [Clausius-Clapyron relation](https://en.wikipedia.org/wiki/Clausius%E2%80%93Clapeyron_relation#Meteorology_and_climatology)

The next variable is `NIEDERSCHLAGSDAUER` which is given as an integer smaller or equal to 10. Therefore it gives the time it has rained during the timestamp interval of 10 Minutes.

The variable `NIEDERSCHLAGSHOEHE` is a measure of precipitytion intensity. Its maximum value can also be checked by the following command:

```r
max(df$NIEDERSCHLAGSHOEHE, na.rm = FALSE)
```

```
## [1] 7.85
```

We can assume that this number gives us the amount of precipitation in milimeters, which is a common definiton and is equivalent as liters of rainfall per squaremeter in a given time interval. A strong event in central Europe can be expected to generate around $30\ \frac{mm}{h}$ of precipitation, i.e. $30\ \frac{l}{m^2 h}$r. The maximum value is therefore an indicator of a very heavy downpour as a continuation for a whole hour would have yielded $6 \times 7.85\ \frac{l}{m^2} = 47.1\ \frac{l}{m^2}$.

The next variable is simply a binary expression of precipitation (1) or no precipitation (0) in the given interval. This is a relevant measure as some types of precipitation do not seem to generate enough water to messure an actual amount of water. The dreaded Northgerman "drizzle" comes to mind.

Again: Since the sensor has taken a snapshot every 10 minutes, we have six observations per hour.

### Intervals
Lubridate allows for the definition of an interval object with the `interval` class. An interval is simply defined as the time elapsed between two points on the date-time line.

e.g. the interval of a single day - from the first of March to the second of March can be defined as follows:

```r
tag_int <- interval(ymd("2020-03-01"), ymd("2020-03-02"))
class(tag_int)
```

```
## [1] "Interval"
## attr(,"package")
## [1] "lubridate"
```

```r
range(int_start(tag_int), int_end(tag_int))
```

```
## [1] "2020-03-01 UTC" "2020-03-02 UTC"
```
This small interval of one day turned out to be good for further analysis.
It facilitates to check results at a glance without having to query the whole dataset.
Note that `int_start()` and `int_end()` are used to obtain the start date and end date of the interval, respectively.

When the desired results can be achieved for a small interval we could consider the interval next in size, e.g. Week, month, quarter and year.

#### Durations and periods
For any weather data, an analysis of seasonal differences is a natural (excuse the pun) objective.
The beginning of each year's seasons align with the solar incidences, i.e. the spring and autumnal equinoxes and the days of most and least sunlight hours in summer and winter, respectively.

The following code snippets define the starting points of the different seasons as points on the POSIXct timeline and use these points to calculate new points by later adding durations and periods to them.

For a seasons view, we create the appropriate points in time, where each season begins.

```r
season_19_spring <- ymd_hm("2019-03-20 22:58", tz="CET")
season_19_summer <- ymd_hm("2019-06-21 17:54", tz="CET")
season_19_autumn <- ymd_hm("2019-09-23 09:50", tz="CET")
season_19_winter <- ymd_hm("2019-12-22 05:19", tz="CET")
season_20_spring <- ymd_hm("2020-03-20 04:49", tz="CET")
season_20_summer <- ymd_hm("2020-06-20 23:43", tz="CET")
```
Source: Beginnings of seasons from [timeanddate.de](https://www.timeanddate.de/astronomie/jahreszeiten)

Our goal shall be to create an interval for the spring season.
The following information is available to determine the end point of spring:

|            | Spring 2020                | Summer 2020                |
|------------|:---------------------------|:---------------------------|
|**Start**   | March 20, 04:49            | June 20, 23:43             |
|**Duration**| 92 days, 17 hours, 54 min. | 93 days, 15 hours, 46 min. |


##### Durations
First approach utilizing the `Duration` class of lubridate:

```r
print(paste("spring 2020 - start: ", season_20_spring))
```

```
## [1] "spring 2020 - start:  2020-03-20 04:49:00"
```

```r
class(ddays(92)) # The lubridate function ddays() creates a `Duration` object
```

```
## [1] "Duration"
## attr(,"package")
## [1] "lubridate"
```

```r
season_20_spring_end <- season_20_spring + 
                          ddays(92) + 
                          dhours(17) + 
                          dminutes(54)

cat(paste("Resulting end of spring: ", season_20_spring_end,
            "\nExpected end of spring:  ", season_20_summer))
```

```
## Resulting end of spring:  2020-06-20 23:43:00 
## Expected end of spring:   2020-06-20 23:43:00
```

##### Periods
Second approach utilizing the `Period` class of lubridate:

```r
class(days(92)) # The lubridate function days() creates a `Period` object
```

```
## [1] "Period"
## attr(,"package")
## [1] "lubridate"
```

```r
season_20_spring_per <- season_20_spring + 
                        days(92) + 
                        hours(17) + 
                        minutes(54)

cat(paste("Resulting end of spring: ", season_20_spring_per,
            "\nExpected end of spring:  ", season_20_summer))
```

```
## Resulting end of spring:  2020-06-20 22:43:00 
## Expected end of spring:   2020-06-20 23:43:00
```
This example shows that the use of `Durations` and `Periods` must be weighed up and balanced depending on what are you looking for.

In the second approach using the `Periods` object, there is a one hour difference relative to the first approach. This is because the daylight savings time shift that occured on March 29, 2020 has been implicitly taken into account in the second example.
In other words: Objects of class `Durations` refer to physical time as if you were using a stop watch, while objects of class `Periods` refer to the time and date displayed on a usual watch. That is, the length of an object of class `Period` is not fixed until it is added to a date-time.

#### Math with periods
Using starting points of seasons summer, autumn and winter to create intervals for our data frame.

```r
int_season_summer <- interval(season_19_summer, season_19_autumn - minutes(1))
int_season_autumn <- interval(season_19_autumn, season_19_winter - minutes(1))
int_season_winter <- season_19_winter %--% (season_20_spring - minutes(1))
```

Note that the `%--%` operator in the last line is similar to the calls of the `interval()` function in the lines above.

### Groupings
What follows is an intermediate step working with `group_by()`s and `rename()`s. Subseqeuntly the findings towards seasons will be applied again. 

The following `group_by()` command creates a unique grouping per hour and then calculates the average temperature for every hour, then pastes this value into each observation per hour.
Recall that `tag_int` ranges from `2020-03-01` to `2020-03-02`.

```r
df_sel <- df %>%
  filter(MESS_DATUM >= int_start(tag_int) & 
           MESS_DATUM <= int_end(tag_int))

df_group <- df_sel %>% 
  group_by(STATIONS_ID, hour(MESS_DATUM), day(MESS_DATUM))

head(df_group, 10)
```

```
## # A tibble: 10 x 9
## # Groups:   STATIONS_ID, hour(MESS_DATUM), day(MESS_DATUM) [2]
##    STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##          <int> <dttm>                   <dbl>            <dbl>            <int>
##  1        2564 2020-03-01 00:00:00        5.2             71.1                0
##  2        2564 2020-03-01 00:10:00        5.4             69.5                0
##  3        2564 2020-03-01 00:20:00        5.5             68.2                0
##  4        2564 2020-03-01 00:30:00        5.6             68.3                0
##  5        2564 2020-03-01 00:40:00        5.5             69                  0
##  6        2564 2020-03-01 00:50:00        5.5             69.1                0
##  7        2564 2020-03-01 01:00:00        5.4             69.7                0
##  8        2564 2020-03-01 01:10:00        5.3             70                  0
##  9        2564 2020-03-01 01:20:00        5.3             69.9                0
## 10        2564 2020-03-01 01:30:00        5.2             71                  0
## # ... with 4 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>, `hour(MESS_DATUM)` <int>,
## #   `day(MESS_DATUM)` <int>
```

For better readability, the columns can be renamed after grouping unsing the `rename()` function.

```r
df_group <- df_sel %>% 
  group_by(STATIONS_ID , hour(MESS_DATUM) , day(MESS_DATUM)) %>% 
  rename("STUNDE" = 'hour(MESS_DATUM)', "TAG" = 'day(MESS_DATUM)')

head(df_group, 10)
```

```
## # A tibble: 10 x 9
## # Groups:   STATIONS_ID, STUNDE, TAG [2]
##    STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##          <int> <dttm>                   <dbl>            <dbl>            <int>
##  1        2564 2020-03-01 00:00:00        5.2             71.1                0
##  2        2564 2020-03-01 00:10:00        5.4             69.5                0
##  3        2564 2020-03-01 00:20:00        5.5             68.2                0
##  4        2564 2020-03-01 00:30:00        5.6             68.3                0
##  5        2564 2020-03-01 00:40:00        5.5             69                  0
##  6        2564 2020-03-01 00:50:00        5.5             69.1                0
##  7        2564 2020-03-01 01:00:00        5.4             69.7                0
##  8        2564 2020-03-01 01:10:00        5.3             70                  0
##  9        2564 2020-03-01 01:20:00        5.3             69.9                0
## 10        2564 2020-03-01 01:30:00        5.2             71                  0
## # ... with 4 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>, STUNDE <int>, TAG <int>
```

Or the columns can be named directly in the `group_by()` command:

```r
df_group <- df_sel %>% 
  group_by(STATIONS_ID , STUNDE = hour(MESS_DATUM), TAG = day(MESS_DATUM))

head(df_group, 5)
```

```
## # A tibble: 5 x 9
## # Groups:   STATIONS_ID, STUNDE, TAG [1]
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <int> <dttm>                   <dbl>            <dbl>            <int>
## 1        2564 2020-03-01 00:00:00        5.2             71.1                0
## 2        2564 2020-03-01 00:10:00        5.4             69.5                0
## 3        2564 2020-03-01 00:20:00        5.5             68.2                0
## 4        2564 2020-03-01 00:30:00        5.6             68.3                0
## 5        2564 2020-03-01 00:40:00        5.5             69                  0
## # ... with 4 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>, STUNDE <int>, TAG <int>
```

Now we can filter the data by year, month, week, day and hour of the day. This should give us possibilities to aggregate freely over any time specifications.

#### Seasons
At this point we create data frames for analysing seasons.

##### Spring
We do not consider the spring season as our data frame does not cover this completely.

##### Summer
The challenge when selecting summer dates is the time zone.
The default time zone in lubridate is UTC, as shown in the example below.

```r
df_summer_attempt_1 <- df %>% filter(MESS_DATUM >= int_start(int_season_summer) &
                                     MESS_DATUM <= int_end(int_season_summer))

range(df_summer_attempt_1$MESS_DATUM)
```

```
## [1] "2019-06-21 16:00:00 UTC" "2019-09-23 07:40:00 UTC"
```

If the expected number of data records does not match the currently determined number, the further analysis should not be continued or repeated.
With the help of assertions (`assert()`) one can identify logical errors in programs or analyses and end them in a controlled manner, if necessary.

```r
rows_sum <- count(df_summer_attempt_1)        # or count(df_select_summer, n())
expected_rows_sum <- 13487                    # determined with e.g. excel or SQL

# make sure, that the condition 'expected_rows_sum == rows_sum' holds
assert("expected rows sum" , expected_rows_sum == rows_sum) 

expected_summer_start <- ymd_hm("201906211800", tz="UTC")
expected_summer_end <- ymd_hm("201909230940", tz="UTC")
# TODO TZ Umstellung!!!
cat(paste("Obtained initial value:", min(df_summer_attempt_1$MESS_DATUM), 
            "\nobtained final value:  ", max(df_summer_attempt_1$MESS_DATUM)))
```

```
## Obtained initial value: 2019-06-21 16:00:00 
## obtained final value:   2019-09-23 07:40:00
```

```r
#assert("expected summer-start" , expected_summer_start == min(df_summer_attempt_1$MESS_DATUM))
#assert("expected summer-end" , expected_summer_end == max(df_summer_attempt_1$MESS_DATUM))
```
While our first approach gets the correct number of records (13,487), the initial value of the interval appear to not be correct since the expected initial value would be `2019-06-21 18:00`. This expected initial value however is refers to the CET (or more specifically CEST) time zone's time, which is basically obtained by adding two hours to UTC time, i.e CEST leads UTC be two hours.

In order to print the start and end time points correctly we use the `stamp()` function of lubridate. 
The `stamp()` function creates more human-readable time formats and internally also reads the system's time locale by calling `Sys.getlocale("LC_TIME")`.

```r
sf <- stamp("set to 24 Jun 2019 3:34") # create a template format as a function
```

```
## Multiple formats matched: "set to %d %Om %Y %H:%M"(1), "set to %d %b %Y %H:%M"(1)
```

```
## Using: "set to %d %b %Y %H:%M"
```

```r
cat(paste0(sf(int_start(int_season_summer)), "\n",
           sf(int_end(int_season_summer))))
```

```
## set to 21 Jun 2019 17:54
## set to 23 Sep 2019 09:49
```

```r
sf <- stamp("set to Monday, 24.06.2019 3:34")
```

```
## Multiple formats matched: "set to Monday, %d.%Om.%Y %H:%M"(1), "set to Monday, %d.%m.%Y %H:%M"(1)
```

```
## Using: "set to Monday, %d.%Om.%Y %H:%M"
```

```r
cat(paste0(sf(int_start(int_season_summer)), "\n",
           sf(int_end(int_season_summer))))
```

```
## set to Monday, 21.06.2019 17:54
## set to Monday, 23.09.2019 09:49
```

```r
sf <- stamp("Created Sunday, Jan 17, 1999 3:34")
```

```
## Multiple formats matched: "Created Sunday, %Om %d, %Y %H:%M"(1), "Created Sunday, %b %d, %Y %H:%M"(1)
```

```
## Using: "Created Sunday, %b %d, %Y %H:%M"
```

```r
print(sf(ymd("2010-04-05")))
```

```
## [1] "Created Sunday, Apr 05, 2010 00:00"
```

```r
# print(sf(int_start(int_season_summer)))
# print(sf(int_end(int_season_summer)))

sf <- stamp("Created 01.01.1999 03:34")
```

```
## Multiple formats matched: "Created %Om.%d.%Y %H:%M"(1), "Created %d.%Om.%Y %H:%M"(1), "Created %m.%d.%Y %H:%M"(1), "Created %d.%m.%Y %H:%M"(1)
```

```
## Using: "Created %Om.%d.%Y %H:%M"
```

```r
print(sf(ymd("2010-04-05")))
```

```
## [1] "Created 04.05.2010 00:00"
```

```r
# print(sf(int_start(int_season_summer)))
# print(sf(int_end(int_season_summer)))
```
The function `stamp()` could be a useful function of lubridate, but is in our opinion a little bit hard to configure.

```r
sft <- "%d.%m.%Y-%H:%M"
print(format(int_start(int_season_summer), sft))
```

```
## [1] "21.06.2019-17:54"
```

```r
print(format(int_end(int_season_summer), sft))
```

```
## [1] "23.09.2019-09:49"
```

```r
print(paste("Summer start/End format"
            , format(int_start(int_season_summer), sft) 
            , format(int_end(int_season_summer), sft)
          ))
```

```
## [1] "Summer start/End format 21.06.2019-17:54 23.09.2019-09:49"
```

```r
print(paste("Summer start/End"
            , int_start(int_season_summer) 
            , int_end(int_season_summer)
          ))
```

```
## [1] "Summer start/End 2019-06-21 17:54:00 2019-09-23 09:49:00"
```

If we do not set the time zone correctly, we get the expected number of data records with our current data frame, but with a time delay.
Select data with timezone format of the data frame's first row.

```r
range(df$MESS_DATUM)    # check the range of the whole data frame
```

```
## [1] "2019-04-14 00:00:00 UTC" "2020-04-13 23:50:00 UTC"
```

```r
df_tz <- tz(df$MESS_DATUM[[1]])                                    # timezone of the first element
# print(paste("TimeZone:", df_tz))
df_summer_attempt_2 <- 
  df %>%
    filter(MESS_DATUM >= force_tz(int_start(int_season_summer))          # use the lubridate function force_tz
           & MESS_DATUM <= force_tz(int_end(int_season_summer))) 
range(df_summer_attempt_2$MESS_DATUM)
```

```
## [1] "2019-06-21 16:00:00 UTC" "2019-09-23 07:40:00 UTC"
```

```r
df_summer_attempt_3 <- 
  df %>%
    filter(MESS_DATUM >= with_tz(int_start(int_season_summer),df_tz)     # use the lubridate function with_tz
           & MESS_DATUM <= with_tz(int_end(int_season_summer),df_tz)) 
range(df_summer_attempt_3$MESS_DATUM)
```

```
## [1] "2019-06-21 16:00:00 UTC" "2019-09-23 07:40:00 UTC"
```

```r
n_start_w <- with_tz(int_start(int_season_summer),df_tz)
n_end_w <- with_tz(int_end(int_season_summer),df_tz)
print(paste("with TZ start/end", n_start_w, "/", n_end_w))
```

```
## [1] "with TZ start/end 2019-06-21 15:54:00 / 2019-09-23 07:49:00"
```

```r
n_start_f <- force_tz(int_start(int_season_summer),df_tz)                # the optimal way
n_end_f <- force_tz(int_end(int_season_summer),df_tz)        
print(paste("Force TZ start/end", n_start_f, "/", n_end_f))
```

```
## [1] "Force TZ start/end 2019-06-21 17:54:00 / 2019-09-23 09:49:00"
```

```r
df_select_summer <-                                                    
  df %>%
    filter(MESS_DATUM >= n_start_f
           & MESS_DATUM <= n_end_f) 
range(df_select_summer$MESS_DATUM)
```

```
## [1] "2019-06-21 18:00:00 UTC" "2019-09-23 09:40:00 UTC"
```

We check our data frame again. 

```r
rows_sum <- count(df_summer_attempt_1)           # or count(df_select_summer, n())
expected_rows_sum <- 13487                    # determined with e.g. excel or SQL

# make sure, that the condition 'expected_rows_sum == rows_sum' holds
assert("expected rows sum" , expected_rows_sum == rows_sum)

expected_summer_start <- ymd_hm("201906211800", tz="UTC")
expected_summer_end <- ymd_hm("201909230940", tz="UTC")
# TODO TZ Umstellung!!!
print(paste("summer min and max", min(df_select_summer$MESS_DATUM), max(df_select_summer$MESS_DATUM)))
```

```
## [1] "summer min and max 2019-06-21 18:00:00 2019-09-23 09:40:00"
```

```r
assert("expected summer-start" , expected_summer_start == min(df_select_summer$MESS_DATUM))
assert("expected summer-end" , expected_summer_end == max(df_select_summer$MESS_DATUM))
```


##### Autumn
New attempt to set the start and end points of an interval with the lubridate functions `floor_date()` and `round_date()`.

```r
print(paste("autumn start/end", season_19_autumn, "/", season_19_winter))
```

```
## [1] "autumn start/end 2019-09-23 09:50:00 / 2019-12-22 05:19:00"
```

```r
show_round_up_start <- round_date(season_19_autumn, unit = "hour")
print(paste("Round hour",show_round_up_start))
```

```
## [1] "Round hour 2019-09-23 10:00:00"
```

```r
#TODO - Warum nicht auf 5:10
show_round_down_end <- floor_date(season_19_winter, unit = "hour")
print(paste("Round minute",show_round_down_end))
```

```
## [1] "Round minute 2019-12-22 05:00:00"
```

```r
print(paste("autumn start/end", show_round_up_start, "/", show_round_down_end))
```

```
## [1] "autumn start/end 2019-09-23 10:00:00 / 2019-12-22 05:00:00"
```

These results are not completly satisfying.
So we use our approach from summer selection of data.

```r
a_start_f <- force_tz(int_start(int_season_autumn), df_tz)        
a_end_f <- force_tz(int_end(int_season_autumn), df_tz)        
print(paste("Force TZ start/end", a_start_f, "/", a_end_f))
```

```
## [1] "Force TZ start/end 2019-09-23 09:50:00 / 2019-12-22 05:18:00"
```

```r
df_select_autumn <- 
  df %>%
    filter(MESS_DATUM >= a_start_f 
           & MESS_DATUM <= a_end_f) 
print(count(df_select_autumn))
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1 12933
```

```r
df_group_autumn <- df_select_autumn %>% group_by(STATIONS_ID , TAG=day(MESS_DATUM) , MONAT=month(MESS_DATUM) , JAHR=year(MESS_DATUM)) 
head(df_group_autumn, 10)
```

```
## # A tibble: 10 x 10
## # Groups:   STATIONS_ID, TAG, MONAT, JAHR [1]
##    STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##          <int> <dttm>                   <dbl>            <dbl>            <int>
##  1        2564 2019-09-23 09:50:00       18.8             66.7                0
##  2        2564 2019-09-23 10:00:00       18.9             64.5                0
##  3        2564 2019-09-23 10:10:00       19.5             61.3                0
##  4        2564 2019-09-23 10:20:00       18.9             59.7                0
##  5        2564 2019-09-23 10:30:00       19.4             58.2                0
##  6        2564 2019-09-23 10:40:00       19.6             55.9                0
##  7        2564 2019-09-23 10:50:00       19.8             55.8                0
##  8        2564 2019-09-23 11:00:00       20.2             56.1                0
##  9        2564 2019-09-23 11:10:00       20               57                  0
## 10        2564 2019-09-23 11:20:00       19.3             68.9                0
## # ... with 5 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>, TAG <int>, MONAT <dbl>, JAHR <dbl>
```

```r
range(df_select_autumn$MESS_DATUM)
```

```
## [1] "2019-09-23 09:50:00 UTC" "2019-12-22 05:10:00 UTC"
```

##### Winter

```r
w_start_f <- force_tz(int_start(int_season_winter),df_tz)        
w_end_f <- force_tz(int_end(int_season_winter),df_tz)        
print(paste("Force TZ start/end", w_start_f, "/", w_end_f))
```

```
## [1] "Force TZ start/end 2019-12-22 05:19:00 / 2020-03-20 04:48:00"
```

```r
df_select_winter <- 
  df %>%
    filter(MESS_DATUM >= w_start_f & MESS_DATUM <= w_end_f) 

#df_group <- df_sel %>% group_by(STATIONS_ID , hour(MESS_DATUM) , day(MESS_DATUM) ) head(df_group, 10)
range(df_select_winter$MESS_DATUM)
```

```
## [1] "2019-12-22 05:20:00 UTC" "2020-03-20 04:40:00 UTC"
```

## Exploration - Analysis

First we are going to calculate average temperatures for different standard intervals like hours, days, weeks, months. We will visualise some data and eventually calculate new variables or aggregates for humdity, precipitation etc.y as well, plus more insights, that might not be apparent at this stage.

### Temperatur
The first variable of interest should be `TEMPERATUR`. A quick visualisation delivers this picture:

```r
ggplot(df) +
  geom_point(aes(MESS_DATUM, TEMPERATUR), colour = "red", size = 0.1)
```

<img src="lubridate_files/figure-html/ggplot MESS_DATUM TEMPERATUR for JAHR-1.png" width="672" />

This is a representation of all 52,704 observations of temperature and naturally appears quite crowded. 
However, a typical course of the seasons during a year can already be interpreted from this plot.
Please note that the chart starts at the start of the observations (April 2019) and stretches over a year from there.

Let's try to get a clearer picture of the temperature variable during the course of the observed year. 
We need to form averages and aggregates to make visualisation more to the point and get less crowded pictures.

New variables should represent other dimensions of date-time data. We can use lubridate functions to produce variables for hours, 24h-days, weeks, months and even possibly seasons.

Assigning a year to every observation by creating a new column with lubridate function `year()`:

```r
df <- df %>% 
  mutate(JAHR = year(df$MESS_DATUM))
```

Assigning the specific month (number) to every observation by creating a new column with lubridate function `month()`:

```r
df <- df %>% 
  mutate(MONAT = month(df$MESS_DATUM))
```

Assigning an epiweek number (`EKW` = Epi Kalender Woche) to every observation through a column with the function `epiweek()`:

```r
df <- df %>% 
  mutate(EKW = epiweek(df$MESS_DATUM))
```

Assigning a day number to every observation by creating a new column `JTAG` (Year day):

```r
df <- df %>%
  mutate(JTAG = yday(df$MESS_DATUM))
```

Assigning an hour to every observation by creating a new column `STUNDE`:

```r
df <- df %>%
  mutate(STUNDE = hour(df$MESS_DATUM))
```

Our data frame (`df`) has 12 variables now and thus we can filter the data by year, month, week, day and hour of the day. This gives us new possibilities to aggregate.

The following `group_by()` command creates a unique grouping per hour and then calculates the average temperature for every hour, then pastes this value into each observation per hour.

```r
df_group_Stunde <- df %>%
  group_by(STATIONS_ID, JAHR, MONAT, EKW, JTAG, STUNDE)

df_av_temp_Stunde  <- df_group_Stunde %>%
  summarise(AVGTEMPH = mean(TEMPERATUR)) %>%
  arrange(STATIONS_ID, JAHR, MONAT, EKW, JTAG, STUNDE)
```

This newly created table has reduced the number of observations by the factor 6 to 8,784 and reveals the average temperature per hour.

Let's plot the data again as hourly temperature averages per day:

```r
ggplot(df_av_temp_Stunde)+
  geom_point(aes(JTAG, AVGTEMPH), colour = "red", size = 0.1)
```

<img src="lubridate_files/figure-html/ggplot avg TEMPERATUR STUNDE-1.png" width="672" />

We can see that the data has become aggregated and that the temperature picture becomes somewhat more clear.
Also the visualisation now stretches over the course of a whole calendar year from left to right.
Let's go one step further and shorten the data down to daily averages by:

```r
df_group_JTAG <- df %>%
  group_by(STATIONS_ID, JAHR, MONAT, EKW, JTAG)

df_av_temp_JTAG  <- df_group_JTAG %>%
  summarise(AVGTEMPD = mean(TEMPERATUR)) %>%
  arrange(STATIONS_ID, JAHR, MONAT, EKW, JTAG) 

head(df_av_temp_JTAG, 10)
```

```
## # A tibble: 10 x 6
## # Groups:   STATIONS_ID, JAHR, MONAT, EKW [2]
##    STATIONS_ID  JAHR MONAT   EKW  JTAG AVGTEMPD
##          <int> <dbl> <dbl> <dbl> <dbl>    <dbl>
##  1        2564  2019     4    16   104     4.96
##  2        2564  2019     4    16   105     5.58
##  3        2564  2019     4    16   106     6.43
##  4        2564  2019     4    16   107     8.17
##  5        2564  2019     4    16   108     9.25
##  6        2564  2019     4    16   109     9.63
##  7        2564  2019     4    16   110     8.91
##  8        2564  2019     4    17   111    11.4 
##  9        2564  2019     4    17   112    12.2 
## 10        2564  2019     4    17   113    12.6
```

This newly created table has reduced the number of observations by the factor 24 to 366 (the number of days in a leap year) and reveals the average temperature per day.
Let's plot the data again as daily temperature averages over the whole period as a line:

```r
ggplot(df_av_temp_JTAG)+
  geom_line(aes(JTAG, AVGTEMPD), colour = "red", size = 0.5)
```

<img src="lubridate_files/figure-html/ggplot avg TEMPERATUR JTAG-1.png" width="672" />

The data starts to make even more visual sense. The key take-aways from this plot, next to the rather trivial finding of higher summer temperatures, is the rather high volatility that appears to be attached to daily average temperature throughout the year. This could be interpreted as the changes between high and low pressure weather systems that cross northern Germany.

Let's boil the data down to weekly temperature averages...

```r
df_group_EKW <- df %>%
  group_by(STATIONS_ID, JAHR, MONAT, EKW)

df_av_temp_EKW  <- df_group_EKW %>%
  summarise(AVGTEMPW = mean(TEMPERATUR)) %>%
  arrange(STATIONS_ID, JAHR, MONAT, EKW)

head(df_av_temp_EKW, 10)
```

```
## # A tibble: 10 x 5
## # Groups:   STATIONS_ID, JAHR, MONAT [3]
##    STATIONS_ID  JAHR MONAT   EKW AVGTEMPW
##          <int> <dbl> <dbl> <dbl>    <dbl>
##  1        2564  2019     4    16     7.56
##  2        2564  2019     4    17    13.2 
##  3        2564  2019     4    18    10.4 
##  4        2564  2019     5    18     6.66
##  5        2564  2019     5    19     8.43
##  6        2564  2019     5    20    10.2 
##  7        2564  2019     5    21    13.4 
##  8        2564  2019     5    22    12.9 
##  9        2564  2019     6    22    16.3 
## 10        2564  2019     6    23    17.7
```

... and plot the results again:

```r
ggplot(df_av_temp_EKW)+
  geom_line(aes(EKW, AVGTEMPW), colour = "red", size = 0.5)
```

<img src="lubridate_files/figure-html/ggplot avg TEMPERATUR WOCHE-1.png" width="672" />

Even in weekly aggregates of temperature averages the resulting visualisation still tells a story of volatility.

Let's look at monthly averages...

```r
df_group_MONAT <- df %>%
  group_by(STATIONS_ID, JAHR, MONAT)

df_av_temp_MONAT  <- df_group_MONAT %>%
  summarise(AVGTEMPM = mean(TEMPERATUR)) %>%
  arrange(STATIONS_ID, JAHR, MONAT) 
head(df_av_temp_MONAT, 12)
```

```
## # A tibble: 12 x 4
## # Groups:   STATIONS_ID, JAHR [2]
##    STATIONS_ID  JAHR MONAT AVGTEMPM
##          <int> <dbl> <dbl>    <dbl>
##  1        2564  2019     4    10.4 
##  2        2564  2019     5    10.6 
##  3        2564  2019     6    17.9 
##  4        2564  2019     7    17.3 
##  5        2564  2019     8    18.4 
##  6        2564  2019     9    14.0 
##  7        2564  2019    10    10.8 
##  8        2564  2019    11     6.16
##  9        2564  2019    12     5.20
## 10        2564  2020     1     5.74
## 11        2564  2020     2     5.74
## 12        2564  2020     3     5.58
```
... and plot again:

```r
ggplot(df_av_temp_MONAT)+
  geom_line(aes(MONAT, AVGTEMPM), colour = "red", size = 1)
```

<img src="lubridate_files/figure-html/ggplot avg TEMPERATUR MONAT-1.png" width="672" />

Which is the kind of temperature curve we would expect to see in a north German location like Kiel with a clear pattern of 3 months of summer with higher average temperatures just below 20 degrees.

It is interesting to note that at least the first half of April 2020 is apparently much warmer on average than the second half of April in 2019. The dimension indicates a difference of about 2 degrees celsius, a significant difference.

#### Average temperature for a day
Generally, no separate variables would have to be created to determine the average temperatures. The values can be gleaned differently from the data frame.

```r
df_avg1 <- df_group %>% 
  summarise(avg = mean(TEMPERATUR)) %>% 
  arrange(STATIONS_ID , TAG , STUNDE) 

head(df_avg1, 10) 
```

```
## # A tibble: 10 x 4
## # Groups:   STATIONS_ID, STUNDE [10]
##    STATIONS_ID STUNDE   TAG   avg
##          <int>  <int> <int> <dbl>
##  1        2564      0     1  5.45
##  2        2564      1     1  5.32
##  3        2564      2     1  5.22
##  4        2564      3     1  5.3 
##  5        2564      4     1  5.53
##  6        2564      5     1  5.58
##  7        2564      6     1  5.35
##  8        2564      7     1  5.9 
##  9        2564      8     1  6.18
## 10        2564      9     1  6.5
```

What we can say at this point is that the temperatures in the city of Kiel have a clear seasonal pattern, but remain highly volatile intra-day and inter-day.

Weather however is not just defined as temperature, but humidity and precipitation play a role regarding our sensation and definition of weather as well.

### Precipitation
#### Sum with condition
The question here is, how long did it rain during an hour/day?
There are two different possible approaches. One is without the binary indicator and the other uses it as a switch.

Without indicator:

```r
df_sum1 <- df_group %>% 
  summarise(sum(NIEDERSCHLAGSDAUER)) %>% 
  arrange(STATIONS_ID, TAG, STUNDE)

head(df_sum1, 24) 
```

```
## # A tibble: 24 x 4
## # Groups:   STATIONS_ID, STUNDE [24]
##    STATIONS_ID STUNDE   TAG `sum(NIEDERSCHLAGSDAUER)`
##          <int>  <int> <int>                     <int>
##  1        2564      0     1                         0
##  2        2564      1     1                         0
##  3        2564      2     1                         0
##  4        2564      3     1                         0
##  5        2564      4     1                         0
##  6        2564      5     1                         0
##  7        2564      6     1                         0
##  8        2564      7     1                         0
##  9        2564      8     1                         0
## 10        2564      9     1                         0
## # ... with 14 more rows
```

With indicator and renamed column:

```r
df_sum2 <- df_group %>% 
  summarise(MENGE = sum(NIEDERSCHLAGSDAUER[NIEDERSCHLAGSINDIKATOR == 1])) %>% 
  filter(MENGE > 0) %>%
  arrange(STATIONS_ID, TAG, STUNDE)

head(df_sum2, 24) 
```

```
## # A tibble: 7 x 4
## # Groups:   STATIONS_ID, STUNDE [7]
##   STATIONS_ID STUNDE   TAG MENGE
##         <int>  <int> <int> <int>
## 1        2564     11     1     2
## 2        2564     12     1     3
## 3        2564     13     1    23
## 4        2564     14     1    14
## 5        2564     15     1    10
## 6        2564     18     1    16
## 7        2564     23     1     1
```

Now we apply the knowledge with the indicator to the selection of autumn.
First we examine the duration of rainfall in autumn.

```r
df_sum_autumn_group <- df_group_autumn 
df_sum_autumn <- df_sum_autumn_group %>% 
              summarise(DAUER = sum(NIEDERSCHLAGSDAUER[NIEDERSCHLAGSINDIKATOR == 1])) %>% 
              filter(DAUER > 0) %>%
              arrange(STATIONS_ID, JAHR, MONAT, TAG)

head(df_sum_autumn, 10) 
```

```
## # A tibble: 10 x 5
## # Groups:   STATIONS_ID, TAG, MONAT [10]
##    STATIONS_ID   TAG MONAT  JAHR DAUER
##          <int> <int> <dbl> <dbl> <int>
##  1        2564    24     9  2019    11
##  2        2564    25     9  2019   593
##  3        2564    26     9  2019   181
##  4        2564    27     9  2019   392
##  5        2564    28     9  2019   228
##  6        2564    29     9  2019  1008
##  7        2564    30     9  2019   354
##  8        2564     1    10  2019   752
##  9        2564     2    10  2019   146
## 10        2564     3    10  2019     2
```

Now we examine the milliliter of precipitation.

```r
df_sum_autumn_liter_group <- df_group_autumn 
df_sum_autumn_liter <- df_sum_autumn_liter_group %>%
            summarise(MENGE = sum(NIEDERSCHLAGSHOEHE[NIEDERSCHLAGSINDIKATOR==1])) %>% 
            filter(MENGE > 0) %>%
            arrange(STATIONS_ID, JAHR, MONAT, TAG)

head(df_sum_autumn_liter, 10) 
```

```
## # A tibble: 10 x 5
## # Groups:   STATIONS_ID, TAG, MONAT [10]
##    STATIONS_ID   TAG MONAT  JAHR MENGE
##          <int> <int> <dbl> <dbl> <dbl>
##  1        2564    25     9  2019  0.71
##  2        2564    26     9  2019  1.18
##  3        2564    27     9  2019 13.3 
##  4        2564    28     9  2019  7.57
##  5        2564    29     9  2019 14.6 
##  6        2564    30     9  2019 11.8 
##  7        2564     1    10  2019 15.4 
##  8        2564     2    10  2019  0.36
##  9        2564     4    10  2019  0.1 
## 10        2564     5    10  2019  0.94
```

### Humidity
Let's have one look at humidity the same way we looked at temperatures as hourly averages:

```r
df_av_humi_Stunde  <- df_group_Stunde %>%
  summarise(AVGHUMI = mean(RELATIVE_FEUCHTE)) %>%
  arrange(STATIONS_ID, JAHR, MONAT, EKW, JTAG, STUNDE)
```

Let's plot the data again as hourly humidity averages per day:

```r
ggplot(df_av_humi_Stunde)+
  geom_point(aes(JTAG, AVGHUMI), colour = "red", size = 0.1)
```

<img src="lubridate_files/figure-html/ggplot avg RELATIVE_FEUCHTE-1.png" width="672" />

This plot shows the drier months during the summer as having a lot more hours with lower humidity values than the months that fall into autumn and winter. 

## Wrap up
It was our objective to show and explain some of the more important features of the lubridate package. We can definitely say that handling this dataset in Base R would have been a lot more labourious. Working with time series in lubridate is not easy, but feasible.
This statement also goes for the other used packages in this exploration (dplyr, ggplot, etc.) without which the whole task would have been quite tedious.
