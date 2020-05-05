# Wrangling data: dplyr {#dplyr}

The R package `dplry` is described as a grammar of data manipulation. It provides commands for the most frequently used types of data transformations, either for the purpose of exploration, data cleaning, or augmenting data.

The package is part of the core tidyverse, which means that we can load it either explictly via `library(dplyr)` or implictly via `library(tidyverse)`. Note that in the following, all commands that are not `dplyr` commands are made explicit via `<package>::<command>`. To demonstrate the main features, we use data on Spotify's daily top 200 charts in Germany in the course of one year. 


```r
library(tidyverse)     # alternatively use library(tidyverse) which covers dplyr + more
df <- readr::read_csv("data/spotify_charts_germany.csv")
```

## Two typial workflows
Before looking in detail into specific functions, let's start with two typical workflows. We will note that 

- dplyr works with the pipe (`%`) such that multiple operations can be combined one after the other, without the need to create intermediate results. 
- function names are quite expressive, basically telling us what they are doing.
- there is a strong analogy to SQL: due to this analogy it is even possible to run dplyr commands with a database backend (the package `dbplyr` needs to be installed)

The first workflow returns an ordered list of the 10 tracks with the highest number of streams on a single day, together with information on the number of streams, day, artist, and track name. 


```r
df %>%                                              # data
  arrange(-Streams) %>%                             # order rows by some variable 
  select(Streams, date, Artist, Track.Name) %>%     # select columns by name
  slice(1:10)                                       # select rows by position
```

```
## # A tibble: 10 x 4
##    Streams date       Artist          Track.Name                                
##      <dbl> <date>     <chr>           <chr>                                     
##  1 1964217 2019-12-24 Mariah Carey    All I Want for Christmas Is You           
##  2 1939974 2019-12-24 Wham!           Last Christmas                            
##  3 1788621 2019-06-21 Capital Bra     Tilidin                                   
##  4 1603796 2019-12-24 Chris Rea       Driving Home for Christmas - 2019 Remaster
##  5 1538169 2019-06-22 Capital Bra     Tilidin                                   
##  6 1482692 2019-06-24 Capital Bra     Tilidin                                   
##  7 1469042 2019-12-24 Shakin' Stevens Merry Christmas Everyone                  
##  8 1461501 2019-05-17 Samra           Wieder Lila                               
##  9 1408165 2019-09-23 Capital Bra     110                                       
## 10 1378266 2019-09-27 Capital Bra     110
```
The second workflow returns the average number of streams per day of week since the beginning of the year 2020. For this operation, the day of week is derived from the date and added as an additional variable via the `mutate` function. 


```r
df %>%                                 # data              
  filter(date>="2020-01-01") %>%       # select rows where condition evaluates to TRUE
                                       # create an additional variable 
  mutate(day_of_week=lubridate::wday(date, label=TRUE, abbr=FALSE, 
                                     locale="American_America.1252", week_start=1)) %>%
  group_by(day_of_week) %>%            # group the data 
  summarise(streams = mean(Streams))   # aggregate per group via functions such as mean, min, etc. 
```

```
## # A tibble: 7 x 2
##   day_of_week streams
##   <ord>         <dbl>
## 1 Monday      125182.
## 2 Tuesday     125230.
## 3 Wednesday   122318.
## 4 Thursday    125505.
## 5 Friday      156173.
## 6 Saturday    141063.
## 7 Sunday      112455.
```

Note that in this particular case, we can write the code more concise by generating the new variable `day_of_week` inside the `group_by` function.


```r
df %>%                                              
  filter(date>="2020-01-01") %>%                    
  group_by(day_of_week=lubridate::wday(date, label=TRUE, abbr=FALSE, locale="American_America.1252", week_start=1)) %>% 
  summarise(streams = mean(Streams))          
```

## Manipulating rows
### Extract rows
The `filter` function is the most frequently used function to extract a subset of rows. The command extracts all rows where the filter condition(s) evaluate to TRUE. The `distinct` function returns distinct rows by removing duplicates (either for the whole data or the specified variables).


```r
df %>% 
  filter(stringr::str_detect(Track.Name, "Santa")) %>% # extract rows where condition is TRUE
  distinct(Artist, Track.Name)          # extract distinct combinations of the two variables
```

```
## # A tibble: 13 x 2
##    Artist          Track.Name                                                   
##    <chr>           <chr>                                                        
##  1 Ariana Grande   Santa Tell Me                                                
##  2 Kylie Minogue   Santa Baby                                                   
##  3 Sia             Santa's Coming For Us                                        
##  4 Michael Bublé   Santa Claus Is Coming to Town                                
##  5 Bruce Springst~ Santa Claus Is Comin' to Town - Live at C.W. Post College, G~
##  6 Frank Sinatra   Santa Claus Is Comin' to Town                                
##  7 Eartha Kitt     Santa Baby (with Henri René & His Orchestra)                 
##  8 Robbie Williams Santa Baby (feat. Helene Fischer)                            
##  9 Gene Autry      Here Comes Santa Claus (Right Down Santa Claus Lane)         
## 10 Rod Stewart     Santa Claus Is Coming To Town                                
## 11 Ariana Grande   Santa Baby                                                   
## 12 The Jackson 5   Santa Claus Is Coming To Town                                
## 13 Mariah Carey    Oh Santa!
```

We can select rows by position via `slice`. Alternatively, we can use the functions `top_n` of `top_fraq` to extract a specified number or fraction of rows. Different from `slice` these functions operate also on grouped data, and we can specify according to which variables the top rows shall be chosen. 


```r
df %>%                        
  group_by(date) %>%        # group by date 
  top_n(1, wt=Streams) %>%  # select 1 row per date, the one with the highest number of streams
  slice(1:10) %>%           # return only the first 10 rows of the resulting data frame
  select(date, Streams, Track.Name, Artist)
```

```
## # A tibble: 366 x 4
## # Groups:   date [366]
##    date       Streams Track.Name  Artist     
##    <date>       <dbl> <chr>       <chr>      
##  1 2019-03-30 1040382 Cherry Lady Capital Bra
##  2 2019-03-31  771685 Cherry Lady Capital Bra
##  3 2019-04-01  861671 Cherry Lady Capital Bra
##  4 2019-04-02  818911 Cherry Lady Capital Bra
##  5 2019-04-03  783832 Cherry Lady Capital Bra
##  6 2019-04-04  770632 Cherry Lady Capital Bra
##  7 2019-04-05  917254 Harami      Samra      
##  8 2019-04-06  836777 Cherry Lady Capital Bra
##  9 2019-04-07  698789 Harami      Samra      
## 10 2019-04-08  798083 Harami      Samra      
## # ... with 356 more rows
```

Another useful feature is selecting rows randomly via `sample_n` or `sample_frac`.

```r
df %>% sample_n(5, replace = TRUE) # Select 5 rows randomly with replacement
```

```
## # A tibble: 5 x 15
##   Position Track.Name Artist Streams date       danceability energy loudness
##      <dbl> <chr>      <chr>    <dbl> <date>            <dbl>  <dbl>    <dbl>
## 1       49 Monster    LUM!X   121935 2019-12-29        0.83   0.794    -6.28
## 2       38 Sucker     Jonas~  201504 2019-04-18        0.842  0.734    -5.07
## 3        8 Dance Mon~ Tones~  357890 2020-02-06        0.825  0.593    -6.40
## 4      106 Eiskalt    Lored~   90070 2020-03-11        0.786  0.606    -6.00
## 5       37 Skifahren~ The C~  146074 2020-03-08       NA     NA        NA   
## # ... with 7 more variables: speechiness <dbl>, acousticness <dbl>,
## #   instrumentalness <dbl>, liveness <dbl>, valence <dbl>, tempo <dbl>,
## #   duration_ms <dbl>
```
### Arranging rows
The function `arrange` is used to order rows by some variable(s). Use minus (`-`) or the `desc` function for arranging in descending order. The following code returns the five most danceable chart tracks of 2019-03-30 by arranging first by date (ascending) and second by danceability (descending). 

```r
df %>% 
  arrange(date, -danceability) %>% 
  slice(1:5) %>% 
  select(Track.Name, date, danceability)
```

```
## # A tibble: 5 x 3
##   Track.Name           date       danceability
##   <chr>                <date>            <dbl>
## 1 Dresscode Gucci      2019-03-30        0.966
## 2 my strange addiction 2019-03-30        0.939
## 3 Gib Ihm              2019-03-30        0.928
## 4 Old Town Road        2019-03-30        0.908
## 5 bury a friend        2019-03-30        0.905
```

## Manipulating columns
### Extract and rename columns
Subset of columns can be extracted via the `select` function. Selection is possible by name or position. Reversely, one can exclude specific columns via negative selection (using `-`). Noteworthy are are the many  helper functions, which are convenient for rapid exploration, but not recommendable for stable software: `start_with`, `last_col`, `everything`, `contains`, etc. One can rename columns while selecting them. If we want to rename a column while preserving the other columns we use the `rename` function.

```r
df %>% select(Position, Track.Name)       # select via column name
df %>% select(1, 2)                       # select via column position
df %>% select(-Track.Name)                # select all columns except Track.Name
df %>% select(starts_with("dance"))       # select all columns starting with "dance" 
df %>% select(danceability, everything()) # reorder danceability first, then remaining columns
df %>% select(song = Track.Name)          # select one column (Track.name) and rename it (song)
df %>% rename(song = Track.Name)          # renames one column, but preserves all the others 
```
### Create new columns
The function `mutate` creates a new variable or overwrites an existing one. Note that we must assign back to make a permanent change to the data. 


```r
df %>% 
  mutate(duration_s = round(duration_ms / 1000)) %>%  # create new variable
  mutate(Track.Name = as.factor(Track.Name)) %>%      # change existing variable
  select(Track.Name,starts_with("duration")) %>%
  head(5)
```

```
## # A tibble: 5 x 3
##   Track.Name     duration_ms duration_s
##   <fct>                <dbl>      <dbl>
## 1 Cherry Lady         135597        136
## 2 Affalterbach        173220        173
## 3 Blackberry Sky      156497        156
## 4 Wolke 10            172827        173
## 5 Puerto Rico         193573        194
```

## Scoped functions
There are scoped variants of ,`mutate` which affect multiple columns at once: 

- `mutate_all`: all columns
- `mutate_at`: all specified columns
- `mutate_if`: all columns that satisfy a condition 

Equivalent scoped variants exist for `select`and `summarise` as well.


```r
df %>% mutate_all(as.character) %>% str()  # change ALL columns to character type
```

```
## tibble [73,200 x 15] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ Position        : chr [1:73200] "1" "2" "3" "4" ...
##  $ Track.Name      : chr [1:73200] "Cherry Lady" "Affalterbach" "Blackberry Sky" "Wolke 10" ...
##  $ Artist          : chr [1:73200] "Capital Bra" "Shindy" "Eno" "MERO" ...
##  $ Streams         : chr [1:73200] "1040382" "822209" "704316" "681426" ...
##  $ date            : chr [1:73200] "2019-03-30" "2019-03-30" "2019-03-30" "2019-03-30" ...
##  $ danceability    : chr [1:73200] "0.838" "0.819" "0.805" "0.77" ...
##  $ energy          : chr [1:73200] "0.549" "0.674" "0.625" "0.797" ...
##  $ loudness        : chr [1:73200] "-7.145" "-4.663" "-8.589" "-4.985" ...
##  $ speechiness     : chr [1:73200] "0.0755" "0.327" "0.0434" "0.0693" ...
##  $ acousticness    : chr [1:73200] "0.877" "0.013" "0.25" "0.0662" ...
##  $ instrumentalness: chr [1:73200] "0.000964" "0" "0.00527" "3.81e-06" ...
##  $ liveness        : chr [1:73200] "0.115" "0.384" "0.0954" "0.0858" ...
##  $ valence         : chr [1:73200] "0.654" "0.766" "0.647" "0.393" ...
##  $ tempo           : chr [1:73200] "114.445" "115.897" "102.996" "100.003" ...
##  $ duration_ms     : chr [1:73200] "135597" "173220" "156497" "172827" ...
```


```r
# Apply the function round to ALL SPECIFIED columns
df %>% 
  mutate_at(vars(danceability, valence), round, digits=1) %>% 
  select(Track.Name,danceability, valence)
```

```
## # A tibble: 73,200 x 3
##    Track.Name             danceability valence
##    <chr>                         <dbl>   <dbl>
##  1 Cherry Lady                     0.8     0.7
##  2 Affalterbach                    0.8     0.8
##  3 Blackberry Sky                  0.8     0.6
##  4 Wolke 10                        0.8     0.4
##  5 Puerto Rico                     0.7     0.6
##  6 Wir ticken                      0.6     0.8
##  7 Pass auf wen du liebst          0.6     0.3
##  8 Alleen                          0.7     0.9
##  9 Ya Salame                       0.7     0.4
## 10 DEUTSCHLAND                     0.5     0.2
## # ... with 73,190 more rows
```

```r
  head(5)
```

```
## [1] 5
```
If there is no predefined function, one can define an anonymous function (which cannot be used outside this context) on the fly:

```r
# transmute is a variant of mutate
df %>% 
  mutate_at(vars(danceability, valence), function(x) x*100) %>%
  select(Track.Name,danceability, valence)
```

```
## # A tibble: 73,200 x 3
##    Track.Name             danceability valence
##    <chr>                         <dbl>   <dbl>
##  1 Cherry Lady                    83.8    65.4
##  2 Affalterbach                   81.9    76.6
##  3 Blackberry Sky                 80.5    64.7
##  4 Wolke 10                       77      39.3
##  5 Puerto Rico                    68.7    62.9
##  6 Wir ticken                     63      82  
##  7 Pass auf wen du liebst         59.2    31.5
##  8 Alleen                         73.2    92.7
##  9 Ya Salame                      71.4    40.8
## 10 DEUTSCHLAND                    52.1    23.7
## # ... with 73,190 more rows
```

```r
  head(5)
```

```
## [1] 5
```

The typical use case for `mutate_if` is changing the variable types of all variables satisfying a specific condition. 

```r
df %>% 
  mutate_if(is.character, as.factor) %>%  # turns all character variables into factors
  glimpse()
```

```
## Rows: 73,200
## Columns: 15
## $ Position         <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15,...
## $ Track.Name       <fct> Cherry Lady, Affalterbach, Blackberry Sky, Wolke 1...
## $ Artist           <fct> Capital Bra, Shindy, Eno, MERO, Fero47, Capital Br...
## $ Streams          <dbl> 1040382, 822209, 704316, 681426, 557781, 534339, 4...
## $ date             <date> 2019-03-30, 2019-03-30, 2019-03-30, 2019-03-30, 2...
## $ danceability     <dbl> 0.838, 0.819, 0.805, 0.770, 0.687, 0.630, 0.592, 0...
## $ energy           <dbl> 0.549, 0.674, 0.625, 0.797, 0.766, 0.692, 0.572, 0...
## $ loudness         <dbl> -7.145, -4.663, -8.589, -4.985, -6.739, -4.951, -8...
## $ speechiness      <dbl> 0.0755, 0.3270, 0.0434, 0.0693, 0.1050, 0.4270, 0....
## $ acousticness     <dbl> 8.77e-01, 1.30e-02, 2.50e-01, 6.62e-02, 3.10e-01, ...
## $ instrumentalness <dbl> 9.64e-04, 0.00e+00, 5.27e-03, 3.81e-06, 0.00e+00, ...
## $ liveness         <dbl> 0.1150, 0.3840, 0.0954, 0.0858, 0.1960, 0.1670, 0....
## $ valence          <dbl> 0.6540, 0.7660, 0.6470, 0.3930, 0.6290, 0.8200, 0....
## $ tempo            <dbl> 114.445, 115.897, 102.996, 100.003, 140.039, 179.8...
## $ duration_ms      <dbl> 135597, 173220, 156497, 172827, 193573, 203067, 20...
```

Note that the condition above (`is.character`) refers to the column as a whole, i.e. the condition returns a single TRUE or FALSE. If we want to mutate a column conditional on the single elements within the column, we use the regular `mutate` function combined with an `if_else`:

```r
df %>% 
  mutate(Top10 = if_else(Position<=10, "Top 10", "Top 11-200")) %>%
  select(Position, Top10, Track.Name) %>%
  slice(8:12)
```

```
## # A tibble: 5 x 3
##   Position Top10      Track.Name 
##      <dbl> <chr>      <chr>      
## 1        8 Top 10     Alleen     
## 2        9 Top 10     Ya Salame  
## 3       10 Top 10     DEUTSCHLAND
## 4       11 Top 11-200 Gib Ihm    
## 5       12 Top 11-200 Jay Jay
```


## Summarise
The `summarise` function is the generic way of calculating summary stats for specific variables. Within the function we can apply base R summary functions (`sum`, `mean` or `max`), one of dplyr's specific summary functions (`n`, `n_distinct`) or a user defined summary function. In the standard case the `summarise` function returns one row.

```r
df %>% 
  filter(date == max(date)) %>%
  summarise(observations = n(),                       # number of observations (dplyr function)
            artists = n_distinct(Artist),             # number of distinct observations (dplyr)
            total_streams = sum(Streams),             # sum (base R) 
            mean_valence = mean(valence, na.rm=TRUE)) # mean (base R)
```

```
## # A tibble: 1 x 4
##   observations artists total_streams mean_valence
##          <int>   <int>         <dbl>        <dbl>
## 1          200     127      20562166        0.520
```

However, we can also apply `summarise` to grouped data. Then one row is returned per group.

```r
df %>% 
  group_by(month = stringr::str_sub(date, 1, 7)) %>%
  summarise(artists = n_distinct(Artist),             # number of distinct observations (dplyr)
            total_streams = sum(Streams),             # sum (base R) 
            mean_valence = mean(valence, na.rm=TRUE)) # mean (base R)
```

```
## # A tibble: 13 x 4
##    month   artists total_streams mean_valence
##    <chr>     <int>         <dbl>        <dbl>
##  1 2019-03     125      53691248        0.503
##  2 2019-04     166     777088420        0.512
##  3 2019-05     196     797746624        0.512
##  4 2019-06     171     824731704        0.504
##  5 2019-07     162     832142863        0.517
##  6 2019-08     161     763883839        0.518
##  7 2019-09     153     805779190        0.505
##  8 2019-10     174     875914502        0.504
##  9 2019-11     195     805356793        0.512
## 10 2019-12     296     952177372        0.525
## 11 2020-01     201     808308854        0.504
## 12 2020-02     215     775863340        0.511
## 13 2020-03     182     726353808        0.521
```

The `count` function is a useful shortcut for `group_by` followed by `summarise(n = n())`.

```r
df %>% count(Artist)
```

```
## # A tibble: 585 x 2
##    Artist                     n
##    <chr>                  <int>
##  1 *NSYNC                    13
##  2 102 Boyz                   9
##  3 18 Karat                 175
##  4 24kGoldn                  26
##  5 5 Seconds of Summer      109
##  6 88-Keys                    1
##  7 a-ha                       1
##  8 A Boogie Wit da Hoodie   203
##  9 A$AP Rocky                 7
## 10 AchtVier                   1
## # ... with 575 more rows
```

```r
# df %>% group_by(Artist) %>% summarise(n = n()) %>% ungroup()
```

Sometimes, we want to add the (group) aggregates as a new column to the existing data frame. In this case we just use `mutate` rather than `summarise`. 

```r
df %>% 
  group_by(date) %>%
  mutate(Total_Streams = sum(Streams), Share = Streams/Total_Streams) %>%
  select(Streams, Total_Streams, Share, Artist)
```

```
## Adding missing grouping variables: `date`
```

```
## # A tibble: 73,200 x 5
## # Groups:   date [366]
##    date       Streams Total_Streams  Share Artist     
##    <date>       <dbl>         <dbl>  <dbl> <chr>      
##  1 2019-03-30 1040382      30400557 0.0342 Capital Bra
##  2 2019-03-30  822209      30400557 0.0270 Shindy     
##  3 2019-03-30  704316      30400557 0.0232 Eno        
##  4 2019-03-30  681426      30400557 0.0224 MERO       
##  5 2019-03-30  557781      30400557 0.0183 Fero47     
##  6 2019-03-30  534339      30400557 0.0176 Capital Bra
##  7 2019-03-30  487419      30400557 0.0160 Ufo361     
##  8 2019-03-30  466665      30400557 0.0154 KC Rebell  
##  9 2019-03-30  449259      30400557 0.0148 Luciano    
## 10 2019-03-30  415859      30400557 0.0137 Rammstein  
## # ... with 73,190 more rows
```


## Combining tables
## Database backend



