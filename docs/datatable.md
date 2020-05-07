# High performance computing: data.table {#data.table}


Okay, let's get started with this chapter on high performance computing using the R package data.table. 


```
## [1] "Hello data.table"
```

If `data.table` is not yet installed, we install it via:

```r
install.packages("data.table" , repos="http://cran.us.r-project.org")
```

We load `data.table` package and - to demonstrate the differences - also the `tidyverse` library.

```r
library(tidyverse) 
library(data.table)
```

Data.table objects are similar to a data.frame object but with some extensions. Let us first load some data with the `data.table` file reader fread(), which is optimised for performance and creates a `data.table` object.


```r
inpOlympic <- if (file.exists("data/olympic-games.csv")) {
   "data/olympic-games.csv"
} 


inpSpoti <- if (file.exists("data/spotify_charts_germany.csv")) {
   "data/spotify_charts_germany.csv"
} 

olympics <- fread(inpOlympic)
spotify <- fread(inpSpoti)
```

## Motivation
The package `data.table` provides an enhanced data frame object called (big surprise: ) **data.table** and a different type of syntax for common types of data manipulations. In this sense it is similar to the tidyverse (especially the dplyr part of it). A `data.table` object has the following form DT[i,j,by] where i represent the rows, j the columns and by the grouping argument.


```r
class(olympics)   # We see that a data.table inherits properties of data.frames
```

```
## [1] "data.table" "data.frame"
```

```r
                  # and we can easily cast an data.frame object to data.table

matrix <- data.frame(c(1,2,3),c(4,5,6),c(7,8,9))
class(matrix)
```

```
## [1] "data.frame"
```

```r
setDT(matrix)
class(matrix)
```

```
## [1] "data.table" "data.frame"
```




So what are the main differences between `data.table` and `tidyverse`? Why or when would we want to use one or the other

- **Performance:** data.tables are heavily optimised for speed. If performance is crucial, then data.tables might be the better choice.
- **Syntax:** data.table syntax is concise while tidyverse syntax is more expressive. Code sequences that span multiple lines are easier to read in case of tidyverse syntax due to the pipe (`%>%`) operator. Overall, the choice is a matter of personal preferences here.
- **Mutability:** data.tables are mutable, i.e. they can be changed in place (by reference). If this is a desired feature, then we might opt for data.tables.
- **Quoted and unquoted names:** in data.tables we usually have the option to use either quoted column names (which makes writing functions easier) or unquoted column names (which is convenient for exploration). In dplyr the default is unquoted column names and it is a bit trickier to use quoted column names.

Before we go into the details, let's start with two motivating examples of differences between data.table and tidyverse syntax. First, we want to know the 5 sports with the highest average weight of players in the 2016 summer olympic games.

```r
# data.table syntax is short
olympics[game=="2016 Summer", .(weight = mean(weight, na.rm=T)), sport][order(-weight)][1:5]
```

```
##         sport   weight
## 1: Basketball 87.86786
## 2: Water Polo 85.44961
## 3:   Handball 83.26912
## 4: Volleyball 80.29329
## 5:     Rowing 79.87226
```


```r
# tidyverse syntax is more expressive and readable
olympics %>% 
  filter(game=="2016 Summer") %>%
  group_by(sport) %>%
  summarise(weight = mean(weight, na.rm=TRUE)) %>%
  arrange(-weight) %>%
  head()
```

Next, we want to change an existing column:

```r
# data.tables are mutable objects, i.e. changes happen in-place
olympics[,age:=as.numeric(age)]   # age is coerced from character to numeric
```


```r
# tidyverse's tibbles are immutable, i.e. we need to reassign to make permanent changes to the data
olympics <- olympics %>% mutate(age_numeric =as.numeric(age))
```



## Data exploration

As mentioned before the datatype data.table has the following structure: dt[i,j,by], where i conditon on the rows, j select the colums and by defines the groups by which results are aggregated. We'll start by setting conditions on the rows.



### Subsetting rows
We can condition on the rows one time, which gives us all swimmers.

again, for repetition the code tidyverse syntax


```r
swimmers <- olympics %>% filter(sport == "Swimming")
```
and in `data.table` syntax

```r
swimmers <- olympics[sport == "Swimming"]
olympics[sport=="Swimming"]                # or without assinging operator
```

```
##               game city    sport                                    discipline
##     1: 1952 Summer <NA> Swimming           Swimming Men's 400 metres Freestyle
##     2: 1912 Summer <NA> Swimming        Swimming Men's 200 metres Breaststroke
##     3: 1912 Summer <NA> Swimming        Swimming Men's 400 metres Breaststroke
##     4: 1920 Summer <NA> Swimming        Swimming Men's 200 metres Breaststroke
##     5: 1920 Summer <NA> Swimming        Swimming Men's 400 metres Breaststroke
##    ---                                                                        
## 23191: 2000 Summer <NA> Swimming    Swimming Men's 4 x 100 metres Medley Relay
## 23192: 2004 Summer <NA> Swimming Swimming Men's 4 x 100 metres Freestyle Relay
## 23193: 1924 Summer <NA> Swimming        Swimming Men's 200 metres Breaststroke
## 23194: 1980 Summer <NA> Swimming           Swimming Men's 100 metres Butterfly
## 23195: 1980 Summer <NA> Swimming           Swimming Men's 200 metres Butterfly
##                                  athlete     country age height weight      bmi
##     1:  Einar Ferdinand ""Einari"" Aalto     Finland  26     NA     NA       NA
##     2:              Arvo Ossian Aaltonen     Finland  22     NA     NA       NA
##     3:              Arvo Ossian Aaltonen     Finland  22     NA     NA       NA
##     4:              Arvo Ossian Aaltonen     Finland  30     NA     NA       NA
##     5:              Arvo Ossian Aaltonen     Finland  30     NA     NA       NA
##    ---                                                                         
## 23191: Klaas Erik ""Klaas-Erik"" Zwering Netherlands  19    189     80 22.39579
## 23192: Klaas Erik ""Klaas-Erik"" Zwering Netherlands  23    189     80 22.39579
## 23193:             Marius Edmund Zwiller      France  18     NA     NA       NA
## 23194:        Bogusaw Stanisaw Zychowicz      Poland  19    189     80 22.39579
## 23195:        Bogusaw Stanisaw Zychowicz      Poland  19    189     80 22.39579
##        sex  medal
##     1:   M   <NA>
##     2:   M   <NA>
##     3:   M   <NA>
##     4:   M Bronze
##     5:   M Bronze
##    ---           
## 23191:   M   <NA>
## 23192:   M Silver
## 23193:   M   <NA>
## 23194:   M   <NA>
## 23195:   M   <NA>
```


And two or more times with the '&' operator ('|' operator for the logical or),for example all gold medal winners in swimming.


```r
swimmers_gold <- olympics[sport == "Swimming" & medal == "Gold"]


swimmers_gold_without_doping <- swimmers_gold[!country=="Russia" & country!= "China"]  #the `!` operator negates the logical expression 
```

The `i` argument also allows for regular expressions


```r
Michael_Phelps <- olympics[sport=="Swimming" & athlete %like% "Phelps"]
```



The columns can be treated like variables so the expression `olympics$sport` is not necessary here but would work too. In additon we can select for the row numbers:

tidyverse:

```r
var <- swimmers %>% slice(1:5)
```
data.table

```r
var <- swimmers[1:5]
```

This is especially useful after ordering the data:


```r
height <- swimmers[order(height)]
```
or in decreasing order( here we pick the first element, so the tallest swimmer)

```r
height <- swimmers[order(-height)][1:1]
```
The order function, called on data.table objects calls the internal `forder()` function which is optimised on such objects. Counting rows is possible with different aproaches:


```r
usmedals <- olympics[sport == "Swimming" & !is.na(medal), length(medal)]
```
or with the .N approach

```r
usmedals <- olympics[sport == "Swimming" & !is.na(medal), .N]
usmedals
```

```
## [1] 3048
```

As you see above we come now to the second argument of data.table objects:

### Selecting Columns

Now we want to condition on the colums too. Here we can either write the column names as unquoted names inside a list. Note we can use the `.` as an abbreviation for `list` here.  

```r
swimmers_gold[, list(athlete, country, medal)][1:5]   # list with unquoted column names 
```

```
##                        athlete       country medal
## 1: Rebecca ""Becky"" Adlington Great Britain  Gold
## 2: Rebecca ""Becky"" Adlington Great Britain  Gold
## 3:      Nathan Ghar-Jun Adrian United States  Gold
## 4:      Nathan Ghar-Jun Adrian United States  Gold
## 5:      Nathan Ghar-Jun Adrian United States  Gold
```

```r
swimmers_gold[,    .(athlete,country, medal)][1:5]   # . is an abbreviation for lists
```

```
##                        athlete       country medal
## 1: Rebecca ""Becky"" Adlington Great Britain  Gold
## 2: Rebecca ""Becky"" Adlington Great Britain  Gold
## 3:      Nathan Ghar-Jun Adrian United States  Gold
## 4:      Nathan Ghar-Jun Adrian United States  Gold
## 5:      Nathan Ghar-Jun Adrian United States  Gold
```
Or we can write the quoted column names inside the `c()` function. 

```r
swimmers[,  c("athlete", "medal")][1:5] #
```

```
##                             athlete  medal
## 1: Einar Ferdinand ""Einari"" Aalto   <NA>
## 2:             Arvo Ossian Aaltonen   <NA>
## 3:             Arvo Ossian Aaltonen   <NA>
## 4:             Arvo Ossian Aaltonen Bronze
## 5:             Arvo Ossian Aaltonen Bronze
```

The tidyverse syntax for comparison was


```r
swimmers %>% select("athlete", "medal")
```

This syntax also allows us to define the column in a separate step.

```r
cols <- c("athlete", "medal")  # columns are defined in a separate step
swimmers_gold[, ..cols][1:5]        # note the .. syntax  
```

```
##                        athlete medal
## 1: Rebecca ""Becky"" Adlington  Gold
## 2: Rebecca ""Becky"" Adlington  Gold
## 3:      Nathan Ghar-Jun Adrian  Gold
## 4:      Nathan Ghar-Jun Adrian  Gold
## 5:      Nathan Ghar-Jun Adrian  Gold
```
or using the 'with' statement


```r
colums <- swimmers_gold[, cols, with = FALSE]
head(colums)
```

```
##                        athlete medal
## 1: Rebecca ""Becky"" Adlington  Gold
## 2: Rebecca ""Becky"" Adlington  Gold
## 3:      Nathan Ghar-Jun Adrian  Gold
## 4:      Nathan Ghar-Jun Adrian  Gold
## 5:      Nathan Ghar-Jun Adrian  Gold
## 6:      Nathan Ghar-Jun Adrian  Gold
```

and operations on the rows and colums at the same time:



```r
var <- olympics[sport == "Swimming" & medal == "Gold", ..cols]
```


We can also execute functions on the columns e.g. to find out the average age of all olympians


```r
olympics[, mean(age, na.rm=T)]
```

```
## [1] 25.45227
```
and last but not least we can directly plot within the `j` argument (even if the informative value is questionable here)


```r
spotify[,plot(danceability,Streams)]
```

### Grouping results

Now we'll consider the third parameter of the data.table object DT[i, j, by].
Let's count the competitions grouped by sports.
(Hint: if we have a look at the data in teamsports, every athlete is listed so caution with this result.)


```r
sports <- olympics[, .N, by = "sport"]
```
So using all three arguments we find all swimmers, and grouping they by the country

```r
sw <- olympics[sport== "Swimming", .N,by = country]
```
grouping by more colums

```r
sw <- olympics[sport== "Swimming", .N,by = .(country,discipline)]
```
If we want to order the data we have to change by to keyby

```r
sw <- olympics[sport== "Swimming", .N,keyby = .(country,discipline)]
```
Chaining operations on the data.table object:

```r
ordOlym <- olympics[,.N, by = .(country,game)][order(-game,country)]
```


### Editing Data

When we want to edit a Data set of the class dataframe like

```r
df = data.frame(name = c("D","a","t","a"), a = 1:4, b = 5:8, c = 9:12)
df
```

```
##   name a b  c
## 1    D 1 5  9
## 2    a 2 6 10
## 3    t 3 7 11
## 4    a 4 8 12
```

```r
df$c <- 13:16
df
```

```
##   name a b  c
## 1    D 1 5 13
## 2    a 2 6 14
## 3    t 3 7 15
## 4    a 4 8 16
```
this is done via copying the data set and a bad performance. `Datatable` provides the ':=' operator for an better performance. Here the Data object isn't copied but edited by reference.

Adding a column

```r
spotify[, ':='(duration_s=duration_ms/1000)]
```
this column is added 'by reference' which is done with a higher performance than adding a column to a `data.frame` object. In addition we do not have to assign the expression back to the variable.
Edit an column

```r
olympics[sex=='M', sex := 'Male']
olympics[sex=='F', sex := 'Female']
```
Deleting an column by reference

```r
spotify[, duration_ms := NULL]
```

### Side effects

Sometimes we're in the situation that we want to work with a `data.table` object but not on the object itself but on a copy. For example we want to add again the column duration_ms by reference, then the variable spotify_cols change too.


```r
spotify_cols <- names(spotify)   # Columnnames
spotify_cols
```

```
##  [1] "Position"         "Track.Name"       "Artist"           "Streams"         
##  [5] "date"             "danceability"     "energy"           "loudness"        
##  [9] "speechiness"      "acousticness"     "instrumentalness" "liveness"        
## [13] "valence"          "tempo"            "duration_s"
```

```r
spotify[, ':='(duration_ms=duration_s*1000)]
spotify_cols
```

```
##  [1] "Position"         "Track.Name"       "Artist"           "Streams"         
##  [5] "date"             "danceability"     "energy"           "loudness"        
##  [9] "speechiness"      "acousticness"     "instrumentalness" "liveness"        
## [13] "valence"          "tempo"            "duration_s"       "duration_ms"
```

If we want to prevent these side effect we have to work on a copy. So again:


```r
spotify[, duration_ms := NULL]   # deleting by reference
spotify_cols
```

```
##  [1] "Position"         "Track.Name"       "Artist"           "Streams"         
##  [5] "date"             "danceability"     "energy"           "loudness"        
##  [9] "speechiness"      "acousticness"     "instrumentalness" "liveness"        
## [13] "valence"          "tempo"            "duration_s"
```

```r
spotify_cols <- copy(names(spotify))
spotify[, ':='(duration_ms=duration_s*1000)]
spotify_cols
```

```
##  [1] "Position"         "Track.Name"       "Artist"           "Streams"         
##  [5] "date"             "danceability"     "energy"           "loudness"        
##  [9] "speechiness"      "acousticness"     "instrumentalness" "liveness"        
## [13] "valence"          "tempo"            "duration_s"
```




## Runtime comparision

Let us now compare the runtime of some expression made in `data.table` and `tidyverse`. Performance differs depending on the type of operation, and will likely also depend on the size of the data set. Here we test separately sorting, filtering, and grouping/aggregating for our data set of about 270.000 observations. 



```r
install.packages("microbenchmark")
```



```r
library(microbenchmark)              # package for performance comparisons
```

```
## Warning: Paket 'microbenchmark' wurde unter R Version 3.6.3 erstellt
```

```r
olympics_tbl <- as_tibble(olympics)  # create a tibble to compare performance

# Sorting
microbenchmark(
  olympics_tbl %>% arrange(age),     # dplyr
  olympics[order(age)],              # data.table
  times = 5                          # run 5 time to get more robust measurements
) 
```

```
## Unit: milliseconds
##                           expr      min       lq      mean   median       uq
##  olympics_tbl %>% arrange(age) 106.6092 208.8035 198.92638 209.8906 210.9465
##           olympics[order(age)]  39.0567  52.7196  62.80308  58.7847  78.2836
##       max neval
##  258.3821     5
##   85.1708     5
```

```r
# Filtering
microbenchmark(
  olympics_tbl %>% filter(height > 160),   # dplyr
  olympics[height > 160],                  # data.table
  times = 5
)
```

```
## Unit: milliseconds
##                                   expr     min      lq     mean  median      uq
##  olympics_tbl %>% filter(height > 160) 15.5657 16.4467 21.32812 17.8250 28.2513
##                 olympics[height > 160] 11.5033 12.0675 32.07294 12.9557 22.7597
##       max neval
##   28.5519     5
##  101.0785     5
```

```r
# Grouping
microbenchmark(
  olympics_tbl %>% group_by(athlete) %>% summarise(mean(height, na.rm=T)),  # dplyr
  olympics[, .(mean(height, na.rm=T)), keyby=athlete],                      # data.table
  times = 5
) 
```

```
## Unit: milliseconds
##                                                                            expr
##  olympics_tbl %>% group_by(athlete) %>% summarise(mean(height,      na.rm = T))
##                         olympics[, .(mean(height, na.rm = T)), keyby = athlete]
##        min        lq      mean    median        uq      max neval
##  2571.6518 3235.8109 4857.7607 5519.9962 5698.1381 7263.206     5
##   462.9999  470.9898  706.7133  503.1117  768.7061 1327.759     5
```


## Further resources
Here are links to further resources on the package `data.table`:

- [Vignette Intro](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html)
- [Vignette on reference semantics](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-reference-semantics.html)
- [Github Wiki](https://github.com/Rdatatable/data.table/wiki)
- [Cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/master/datatable.pdf)
- [data.table vs. dplyr](https://stackoverflow.com/questions/21435339/data-table-vs-dplyr-can-one-do-something-well-the-other-cant-or-does-poorly)







