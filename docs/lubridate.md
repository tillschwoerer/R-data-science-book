# Dates and times: lubridate {#lubridate}

Resources:

- [Lubridate homepage](https://lubridate.tidyverse.org/)
- [Cheatsheet](https://rawgit.com/rstudio/cheatsheets/master/lubridate.pdf)
- [Book Chapter in R4DS](https://r4ds.had.co.nz/dates-and-times.html)
- [Vignette](https://cran.r-project.org/web/packages/lubridate/vignettes/lubridate.html)

Suggested data set: weather-kiel-holtenau

## Background

### What is Lubridate?
Lubridate is an R-Package designed to ease working with date/time variables. These can be quite challenging in baseR and lubridate makes working with dates and times more frictionless, hence the name.

Lubridate ist part of the tidyverse package, but can be installed seperately as well. It probably reveals most of its usefullness in collaboration with other tidyverse packages. A useful extension depending on your data might be the time-series package, which is not part of tidyverse.

All mentioned packages can be optained with the following commands. 

package.install("lubridate")
package.install("tidyverse")
package.install("time-series")

If the have been installed previously in your environment, they might have to be called upon by using 
library(tidyverse) and so forth.

## Basics

Some examples of real world date - time formats found in datasets:

How people talk about dates and times often differs from the notation of the given information. Depending on the specific use of the data, the given information might be more or less granular. When people in the USA talk distance between two places, they often give an approximation of how long it will take a person to drive from A to B and round-up or down to the hour. 

Flight schedules will most likely be exact to the minute, while some sensordata will probably need to be exact to the milisecond. So there will be differing granularity in date and time data. 

Even if this would not be a challenge, we still would have to deal with different notations of date and time. People in Germany will write a day-date like: dd.mm.yyyy or short dd.mm.yy, while the anglo-saxon realm will use mm.dd.yyyy frequently and the most chronologically sound way would be to use yyyy.mm.dd, but this doesn't seem to stick with humans. 

On top of these issues there's the fact that time itself does not make the impression of being the most exact parameter out there. Universal time might appear linear, but the way our planet revolves our galaxy has made it neccessary to adjust date and times every now and then, so our Kalender stays in tune with our defined seasons. This creates leap years, skipped seconds, daylight-savings time and last, but not least time-zones, which can mess things up even further.

Three types of date/time data
Sys.time() functions
Time Formats

## Application - Import, Clean Data / date-time
We will apply the lubridate package to Weather data from on stationary sensor in northern Germany, the weather station in Kiel-Holtenau to be more exact.

Before we introduce the library, the prerequisites must be created.

```r
# zum Lesen von Dateien
library(readr)
library(lubridate)
```

```
## 
## Attache Paket: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

A first step is to import the data, which is given in a csv-format. We will use the tidyverse version of read_csv to accomplish this step.

The data will be called df in order to make reference in code an writing more efficient further down.

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
head(df)
```

```
## # A tibble: 6 x 7
##   STATIONS_ID MESS_DATUM TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl>      <dbl>      <dbl>            <dbl>            <dbl>
## 1        2564    2.02e11        3.6             75.6                0
## 2        2564    2.02e11        3.6             75.6                6
## 3        2564    2.02e11        3.6             76.5                0
## 4        2564    2.02e11        3.5             75.6                0
## 5        2564    2.02e11        3.6             75.3                0
## 6        2564    2.02e11        3.7             75.1                2
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```
All columns are recognized as double values.


We will examine the data further in a few seconds. At this point it should be mentioned that we generated the code to import the data by using the data import readr tool of RStudio. This tool allows a first look at the raw csv data before import and some tweaking. 

By looking at the "MESS_DATUM" column it became apparent that this is a timestamp that was created every 10 minutes. The "normal" column type definitions of the import tool would not have sufficed to format this appropriately, which is why we chose to keep the column defined as a double until after import.

```r
df$MESS_DATUM <- ymd_hm(df$MESS_DATUM)
head(df)
```

```
## # A tibble: 6 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl> <dttm>                   <dbl>            <dbl>            <dbl>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## 6        2564 2019-04-14 00:50:00        3.7             75.1                2
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```

When you importing the column "MESS_DATUM" as character, you can use the following function:


```r
df_a2 <- read_csv("data/weather_kiel_holtenau.csv", 
    col_types = cols(MESS_DATUM = col_character()))

df_a2$MESS_DATUM <- parse_date_time(df_a2$MESS_DATUM, orders ="Ymd HM")
head(df_a2)
```

```
## # A tibble: 6 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl> <dttm>                   <dbl>            <dbl>            <dbl>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## 6        2564 2019-04-14 00:50:00        3.7             75.1                2
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```


If you no longer want to process or analyze the data, you can also display a date-time object in a specific format.

```r
df$MESS_DATUM <- format(df$MESS_DATUM, "%d.%m.%Y-%H:%M")
#TODO
# statt format( df$MESS_DATUM, "%Y%m%d%H%M") geht auch
# strftime(Sys.time(), "%Y%m%d%H%M")
# return character
# der nächste Ansatz müsste wieder von einer list zu einem character-array gewandelt werden, sonst Anzeigen von NA
#  strptime(df$MESS_DATUM, "%Y%m%d%H%M")
# strptime(Sys.time(), "%Y%m%d%H%M")
# return list

head(df)
```

```
## # A tibble: 6 x 7
##   STATIONS_ID MESS_DATUM TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <dbl> <chr>           <dbl>            <dbl>            <dbl>
## 1        2564 14.04.201~        3.6             75.6                0
## 2        2564 14.04.201~        3.6             75.6                6
## 3        2564 14.04.201~        3.6             76.5                0
## 4        2564 14.04.201~        3.5             75.6                0
## 5        2564 14.04.201~        3.6             75.3                0
## 6        2564 14.04.201~        3.7             75.1                2
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <dbl>
```
This approach is ok, but not yet ideal for further processing.

The second approach is a little better.
You can already specify data types for each column when reading in.
For this Step you can use the RStudio "Import Dataset"-Wizard.

Is DF immutable?

```r
df_a3 <- read_csv("data/weather_kiel_holtenau.csv", 
    col_types = cols(MESS_DATUM = col_datetime(format = "%Y%m%d%H%M"), 
        NIEDERSCHLAGSDAUER = col_integer(), 
        NIEDERSCHLAGSINDIKATOR = col_integer(), 
        STATIONS_ID = col_integer()))
#View(weather_kiel_holtenau)
head(df_a3)
```

```
## # A tibble: 6 x 7
##   STATIONS_ID MESS_DATUM          TEMPERATUR RELATIVE_FEUCHTE NIEDERSCHLAGSDA~
##         <int> <dttm>                   <dbl>            <dbl>            <int>
## 1        2564 2019-04-14 00:00:00        3.6             75.6                0
## 2        2564 2019-04-14 00:10:00        3.6             75.6                6
## 3        2564 2019-04-14 00:20:00        3.6             76.5                0
## 4        2564 2019-04-14 00:30:00        3.5             75.6                0
## 5        2564 2019-04-14 00:40:00        3.6             75.3                0
## 6        2564 2019-04-14 00:50:00        3.7             75.1                2
## # ... with 2 more variables: NIEDERSCHLAGSHOEHE <dbl>,
## #   NIEDERSCHLAGSINDIKATOR <int>
```




(Idea - check, if data is openly available as well. This could lead to an excurse about how to generate a raw dataset from this weatherstation.)

Examine the data - - summarise, glimpse an other exploration functions

Parse the date-time Variable into a productive format

The result should be a tidy data frame.

Create an overview over the units of measured variables.

possible: Formats and representation of the data. European vs. US number formats etc.

possible: check time-series package, if this would help us in any way to make data even more accessible?

## Application - Create new date-time Variables

week of year (mutate)
separate daytime vs. nighttime
separate seasons (spring, summer, fall, winter)


## Application - Exploration - Analysis

Calculate Average Temperatures (d/m/season/y)
Calculate Averages for humdity, rainfall etc.
Plus many more insights, that are not apparent at this draft stage.


## Application - Visualisation of core findings

- How has the climate changed during the observed intervall?
- How can relevant intervalls be compared?
- can we find historic KPIs to compare our findings (eg. average temperature in January in Kiel 1900)

What Visualisations make sense for our kind of data/insights? Research and try&error

## Wrap up - outlook date-time / time-series
what potential problems have not been adressed?

## Wrap up - What's next/out there?
Insights from data / data vis

What potential problems have not been adressed? (Time series package? etc?)
