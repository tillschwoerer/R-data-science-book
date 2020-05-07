# Wrangling data: dplyr {#dplyr}

The R package `dplry` is described as a grammar of data manipulation. It provides commands for the most frequently used types of data transformations, either for the purpose of exploration, data cleaning, or augmenting data.

The package is part of the core tidyverse, which means that we can load it either explictly via `library(dplyr)` or implictly via `library(tidyverse)`. Note that in the following, all commands that are not `dplyr` commands are made explicit via `<package>::<command>`. To demonstrate the main features, we use data on Spotify's daily top 200 charts in Germany in the course of one year. 


```r
library(tidyverse)     # alternatively use library(tidyverse) which covers dplyr + more
df <- readr::read_csv("data/spotify_charts_germany.csv")
```

## Two typial workflows {#typical-workflows}
Before looking in detail into specific functions, let's start with two typical workflows. We will note that 

- dplyr works with the pipe (`%>%`) such that multiple operations can be combined one after the other, without the need to create intermediate results. 
- function names are quite expressive, basically telling us what they are doing.
- there is a strong analogy to SQL: due to this analogy it is even possible to run dplyr commands with a database backend (the package `dbplyr` needs to be installed)

The first workflow returns an ordered list of the 5 tracks with the highest number of streams on a single day:


```r
df %>%                                              # data
  select(Streams, date, Artist, Track.Name) %>%     # select columns by name
  arrange(-Streams) %>%                             # order rows by some variable 
  slice(1:5)                                        # select rows by position
```

```
## # A tibble: 5 x 4
##   Streams date       Artist       Track.Name                                
##     <dbl> <date>     <chr>        <chr>                                     
## 1 1964217 2019-12-24 Mariah Carey All I Want for Christmas Is You           
## 2 1939974 2019-12-24 Wham!        Last Christmas                            
## 3 1788621 2019-06-21 Capital Bra  Tilidin                                   
## 4 1603796 2019-12-24 Chris Rea    Driving Home for Christmas - 2019 Remaster
## 5 1538169 2019-06-22 Capital Bra  Tilidin
```
The second workflow returns the average number of streams per day of week since the beginning of the year 2020. For this operation, the day of week is derived from the date and added as an additional variable via the `mutate` function. 


```r
df %>%                                 # data              
  filter(date>="2020-01-01") %>%       # select rows where condition evaluates to TRUE
                                       # create an additional variable 
  mutate(day_of_week=lubridate::wday(date, label=TRUE, abbr=FALSE, week_start=1)) %>%
  group_by(day_of_week) %>%            # group the data 
  summarise(streams = mean(Streams))   # aggregate per group via functions such as mean, min, etc. 
```

```
## # A tibble: 7 x 2
##   day_of_week streams
##   <ord>         <dbl>
## 1 Montag      125182.
## 2 Dienstag    125230.
## 3 Mittwoch    122318.
## 4 Donnerstag  125505.
## 5 Freitag     156173.
## 6 Samstag     141063.
## 7 Sonntag     112455.
```

Note that in this particular case, we can write the code more concise by generating the new variable `day_of_week` inside the `group_by` function.


```r
df %>%                                              
  filter(date>="2020-01-01") %>%                    
  group_by(day_of_week=lubridate::wday(date, label=TRUE, abbr=FALSE, week_start=1)) %>% 
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

We can select rows by position via `slice`. If we want to display the first or last n rows, we can also use the base R functions `head` and `tail`. The functions `top_n` of `top_fraq` allow us to extract the  specified number/fraction of rows, according to the ordering of a specified variable. In addition, `top_n` and `top_frac` also operate also on grouped data


```r
df %>% slice(c(1,3,5))   # selects rows by position (in the given order)
```

```
## # A tibble: 3 x 15
##   Position Track.Name Artist Streams date       danceability energy loudness
##      <dbl> <chr>      <chr>    <dbl> <date>            <dbl>  <dbl>    <dbl>
## 1        1 Cherry La~ Capit~ 1040382 2019-03-30        0.838  0.549    -7.14
## 2        3 Blackberr~ Eno     704316 2019-03-30        0.805  0.625    -8.59
## 3        5 Puerto Ri~ Fero47  557781 2019-03-30        0.687  0.766    -6.74
## # ... with 7 more variables: speechiness <dbl>, acousticness <dbl>,
## #   instrumentalness <dbl>, liveness <dbl>, valence <dbl>, tempo <dbl>,
## #   duration_ms <dbl>
```

```r
df %>% head(3)           # selects first n rows (in the given order)
```

```
## # A tibble: 3 x 15
##   Position Track.Name Artist Streams date       danceability energy loudness
##      <dbl> <chr>      <chr>    <dbl> <date>            <dbl>  <dbl>    <dbl>
## 1        1 Cherry La~ Capit~ 1040382 2019-03-30        0.838  0.549    -7.14
## 2        2 Affalterb~ Shindy  822209 2019-03-30        0.819  0.674    -4.66
## 3        3 Blackberr~ Eno     704316 2019-03-30        0.805  0.625    -8.59
## # ... with 7 more variables: speechiness <dbl>, acousticness <dbl>,
## #   instrumentalness <dbl>, liveness <dbl>, valence <dbl>, tempo <dbl>,
## #   duration_ms <dbl>
```

```r
df %>% top_n(3, Streams) # selects top n rows (based on the variable Streams)
```

```
## # A tibble: 3 x 15
##   Position Track.Name Artist Streams date       danceability energy loudness
##      <dbl> <chr>      <chr>    <dbl> <date>            <dbl>  <dbl>    <dbl>
## 1        1 Tilidin    Capit~ 1788621 2019-06-21        0.631  0.673    -4.89
## 2        1 All I Wan~ Maria~ 1964217 2019-12-24        0.335  0.625    -7.46
## 3        2 Last Chri~ Wham!  1939974 2019-12-24        0.735  0.478   -12.5 
## # ... with 7 more variables: speechiness <dbl>, acousticness <dbl>,
## #   instrumentalness <dbl>, liveness <dbl>, valence <dbl>, tempo <dbl>,
## #   duration_ms <dbl>
```



```r
df %>%                        
  group_by(date) %>%        # group by date 
  top_n(1, wt=Streams) %>%  # select 1 row per date, the one with the highest number of streams
  select(date, Streams, Track.Name, Artist) %>%
  head(5)
```

```
## # A tibble: 5 x 4
## # Groups:   date [5]
##   date       Streams Track.Name  Artist     
##   <date>       <dbl> <chr>       <chr>      
## 1 2019-03-30 1040382 Cherry Lady Capital Bra
## 2 2019-03-31  771685 Cherry Lady Capital Bra
## 3 2019-04-01  861671 Cherry Lady Capital Bra
## 4 2019-04-02  818911 Cherry Lady Capital Bra
## 5 2019-04-03  783832 Cherry Lady Capital Bra
```

Another useful feature is selecting rows randomly via `sample_n` or `sample_frac` (output hidden).

```r
df %>% sample_n(5)                     # Select 5 rows randomly with replacement
df %>% sample_frac(0.1, replace=TRUE)  # Select a 10% random sample with replacement
```

### Arranging rows
The function `arrange` is used to order rows by some variable(s). Use minus (`-`) or the `desc` function for arranging in descending order. The following code returns the five most danceable chart tracks of 2019-03-30 by arranging first by date (ascending) and second by danceability (descending). 

```r
df %>% 
  arrange(date, -danceability) %>%     # orders the data first by date (asc), then by danceability (desc)
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
df %>% mutate_all(as.character) %>% head(5) # change ALL columns to character type
```

```
## # A tibble: 5 x 15
##   Position Track.Name Artist Streams date  danceability energy loudness
##   <chr>    <chr>      <chr>  <chr>   <chr> <chr>        <chr>  <chr>   
## 1 1        Cherry La~ Capit~ 1040382 2019~ 0.838        0.549  -7.145  
## 2 2        Affalterb~ Shindy 822209  2019~ 0.819        0.674  -4.663  
## 3 3        Blackberr~ Eno    704316  2019~ 0.805        0.625  -8.589  
## 4 4        Wolke 10   MERO   681426  2019~ 0.77         0.797  -4.985  
## 5 5        Puerto Ri~ Fero47 557781  2019~ 0.687        0.766  -6.739  
## # ... with 7 more variables: speechiness <chr>, acousticness <chr>,
## #   instrumentalness <chr>, liveness <chr>, valence <chr>, tempo <chr>,
## #   duration_ms <chr>
```


```r
df %>% 
  mutate_at(vars(danceability, valence), round, digits=1) %>% # Round all specified columns
  select(Track.Name,danceability, valence, energy) %>%        # We see that energy was not rounded
  head(5)
```

```
## # A tibble: 5 x 4
##   Track.Name     danceability valence energy
##   <chr>                 <dbl>   <dbl>  <dbl>
## 1 Cherry Lady             0.8     0.7  0.549
## 2 Affalterbach            0.8     0.8  0.674
## 3 Blackberry Sky          0.8     0.6  0.625
## 4 Wolke 10                0.8     0.4  0.797
## 5 Puerto Rico             0.7     0.6  0.766
```
If there is no predefined function, one can define an anonymous function (which cannot be used outside this context) on the fly:

```r
df %>% 
  mutate_at(vars(danceability, valence), function(x) x*100) %>% # Here we define a custom function in-line
  select(Track.Name,danceability, valence) %>%
  head(5)
```

```
## # A tibble: 5 x 3
##   Track.Name     danceability valence
##   <chr>                 <dbl>   <dbl>
## 1 Cherry Lady            83.8    65.4
## 2 Affalterbach           81.9    76.6
## 3 Blackberry Sky         80.5    64.7
## 4 Wolke 10               77      39.3
## 5 Puerto Rico            68.7    62.9
```

The typical use case for `mutate_if` is changing the variable types of all variables satisfying a specific condition. 

```r
df %>% 
  mutate_if(is.character, as.factor) %>%  # IF column has type character, change it to factor
  glimpse()                               # We see that Track.Name and Artist were coerced to factor
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
# df %>% group_by(Artist) %>% summarise(n = n()) %>% ungroup() %>% head(5)
df %>% count(Artist) %>% head(5)
```

```
## # A tibble: 5 x 2
##   Artist                  n
##   <chr>               <int>
## 1 *NSYNC                 13
## 2 102 Boyz                9
## 3 18 Karat              175
## 4 24kGoldn               26
## 5 5 Seconds of Summer   109
```

Sometimes, we want to add the (group) aggregates as a new column to the existing data frame. In this case we just use `mutate` rather than `summarise`. 

```r
df %>% 
  group_by(date) %>%
  mutate(Total_Streams = sum(Streams), Share = Streams/Total_Streams) %>%
  select(Streams, Total_Streams, Share, Artist) %>%
  head(5)
```

```
## Adding missing grouping variables: `date`
```

```
## # A tibble: 5 x 5
## # Groups:   date [1]
##   date       Streams Total_Streams  Share Artist     
##   <date>       <dbl>         <dbl>  <dbl> <chr>      
## 1 2019-03-30 1040382      30400557 0.0342 Capital Bra
## 2 2019-03-30  822209      30400557 0.0270 Shindy     
## 3 2019-03-30  704316      30400557 0.0232 Eno        
## 4 2019-03-30  681426      30400557 0.0224 MERO       
## 5 2019-03-30  557781      30400557 0.0183 Fero47
```


## Combining tables
## Database backend
### Motivation
As mentioned before, the `dplyr` syntax reveals strong analogies with SQL. What is more, it is even possible to use `dplyr` with a database backend. 

**What does this mean? And when is this useful?**

In a company setting, raw data is usually stored in some form of database. When we want to work with the data in R, the standard way would be to open a connection to the database and read in the data into R's memory. However, if the size of data is large, there may be problems with this approach: 

- Large data require long reading time 
- Data sets might not even fit into memory
- Computations might have low performance

If we want to work on the raw data (e.g. for statistical / machine learning modelling), this constitutes a problem: either we need a system with larger memory / higher performance. Or we must restrict ourselves to a smaller sample of the data. Or we could connect R to a technology for distributed machine learning, such as Apache Spark.

In some cases, however, we don't actually need to work on the raw data. We would be happy to let the database do the calculations for us (these are built to store and process huge amounts of data), and just read in the resulting data, which is often much smaller in size. This is precisely the use case for dplyr with a database backend.

The idea is to write regular dplyr code. The code is translated into SQL under the hood. The data is retrieved from the database and only the results are actually read into R's memory.

### Set up
First, we need to install a few things: 

- **Database:** In a company setting, the database will already be there. If you want to install a database on your computer, popular choices are PostgreSQL or MySQL. Here is an [overview of possible choices](https://db.rstudio.com/databases/)S For this book we will use an in-memory SQLite database. The benefit is that everyone will be able to run the code without the need to set up a proper database.    
- **DBI backend package:** DBI stands for database interface. We need a package that corresponds to our database. In our case we will use the package `RSQLite`. With many other databases, the package `odbc` would be proper choice.
- **`dbplyr` package** This package needs to be installed, but we never need to load it explictly. Once installed, it is sufficient to load the regular `dplyr` package.

Second, we need to connect R to the database. The arguments look slightly different, depending on the database that you are using. Usually, you would also need to specify a user name and password.

```r
con <- DBI::dbConnect(RSQLite::SQLite(), dbname = ":memory:")
```

Third, we need to have data in our database. In a company setting, the data would already be there. In our case, we create a table `spotify-charts-germany` in the database and copy the corresponding data from R's memory (df) into this table.


```r
copy_to(con, df, "spotify-charts-germany")
```

### Querying the database
First, we register the database table via the `tbl` function. 

```r
spotify_db <- tbl(con, "spotify-charts-germany")
```

Now we can query this database table using regular `dplyr` syntax. Note that this works smoothly for the majority but not for all `dplyr` commands. For instance the `slice` function is not implemented, i.e. it has no translation to SQL. Hence, in the following statement we extract the first five rows via `head(5)` instead of `slice(1:5)`. Otherwise the sequence of commands looks identical to the one [presented above based on a normal R data frame/tibble](#typical-workflows).


```r
spotify_db %>%                                      # reference to the database table
  select(Streams, date, Artist, Track.Name) %>%     # select columns by name
  arrange(-Streams) %>%                             # order rows by some variable   
  head(5)                                           # select rows by position  
```

```
## # Source:     lazy query [?? x 4]
## # Database:   sqlite 3.29.0 [:memory:]
## # Ordered by: -Streams
##   Streams  date Artist       Track.Name                                
##     <dbl> <dbl> <chr>        <chr>                                     
## 1 1964217 18254 Mariah Carey All I Want for Christmas Is You           
## 2 1939974 18254 Wham!        Last Christmas                            
## 3 1788621 18068 Capital Bra  Tilidin                                   
## 4 1603796 18254 Chris Rea    Driving Home for Christmas - 2019 Remaster
## 5 1538169 18069 Capital Bra  Tilidin
```

We can actually see the SQL generated by dplyr in the background via `show_query`. 


```r
spotify_db %>%
  select(Streams, date, Artist, Track.Name) %>%
  arrange(-Streams) %>%
  head(5) %>% 
  show_query()                                      # shows the translation into SQL
```

```
## <SQL>
## SELECT `Streams`, `date`, `Artist`, `Track.Name`
## FROM `spotify-charts-germany`
## ORDER BY -`Streams`
## LIMIT 5
```

Alternatively, we could achieve the same by writing the SQL query ourselves, and send the query to the database.

```r
query <-  "SELECT Streams, date, Artist, `Track.Name` 
          FROM `spotify-charts-germany`
          ORDER BY Streams DESC
          LIMIT 5"
RMySQL::dbSendQuery(con, query)
```

It is important to understand that the data is not in R's memory until we explicitly `collect` the data. Once the data is collected, it behaves like any regular R data frame.

```r
rdata <- 
  spotify_db %>%
  select(Streams, date, Artist, Track.Name) %>%
  arrange(-Streams) %>%
  head(5) %>% 
  collect()                   # this pulls the data into R's memory
class(rdata)                  # this is a regular R data frame / tibble
```

```
## [1] "tbl_df"     "tbl"        "data.frame"
```




