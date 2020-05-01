# High performance computing: data.table {#data.table}

Resources:

- [Vignette Intro](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html)
- [Vignette on reference semantics](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-reference-semantics.html)
- [Github Wiki](https://github.com/Rdatatable/data.table/wiki)
- [Cheatsheet](https://raw.githubusercontent.com/rstudio/cheatsheets/master/datatable.pdf)
- [data.table vs. dplyr](https://stackoverflow.com/questions/21435339/data-table-vs-dplyr-can-one-do-something-well-the-other-cant-or-does-poorly)

Suggested data: any

Possible approach: take different sequences of operations in dplyr and translate it into data.table syntax. Then summarise differences and pros/cons vis-à-vis dplyr 

Okay, let's get started with this chapter on high performance computing using the R package data.table.
First we will install exactly this package.


```r
print("Hello Data.Table")
```

```
## [1] "Hello Data.Table"
```

```r
install.packages("data.table" , repos="http://cran.us.r-project.org")
```

```
## package 'data.table' successfully unpacked and MD5 sums checked
```

```
## Warning: cannot remove prior installation of package 'data.table'
```

```
## Warning in file.copy(savedcopy, lib, recursive = TRUE): Problem C:
## \R\R-3.6.1\library\00LOCK\data.table\libs\x64\datatable.dll nach C:
## \R\R-3.6.1\library\data.table\libs\x64\datatable.dll zu kopieren: Permission
## denied
```

```
## Warning: restored 'data.table'
```

```
## 
## The downloaded binary packages are in
## 	C:\Users\tschwoer\AppData\Local\Temp\Rtmp4A4Abf\downloaded_packages
```



```r
library(tidyverse) 
```

```
## -- Attaching packages ---- tidyverse 1.2.1 --
```

```
## v ggplot2 3.3.0     v purrr   0.3.2
## v tibble  3.0.0     v dplyr   0.8.3
## v tidyr   1.0.0     v stringr 1.4.0
## v readr   1.3.1     v forcats 0.4.0
```

```
## Warning: Paket 'ggplot2' wurde unter R Version 3.6.3 erstellt
```

```
## Warning: Paket 'tibble' wurde unter R Version 3.6.3 erstellt
```

```
## -- Conflicts ------- tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(data.table)
```

```
## Warning: Paket 'data.table' wurde unter R Version 3.6.3 erstellt
```

```
## 
## Attache Paket: 'data.table'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     between, first, last
```

```
## The following object is masked from 'package:purrr':
## 
##     transpose
```

```r
inpOlympic <- if (file.exists("data/olympic-games.csv")) {
   "data/olympic-games.csv"
} 


inpWeather <- if (file.exists("data/weather_kiel_holtenau.csv")) {
   "data/weather_kiel_holtenau.csv"
} 

inpSpoti <- if (file.exists("data/spotify_charts_germany.csv")) {
   "data/spotify_charts_germany.csv"
} 

olympics <- fread(inpOlympic)
weather <- fread(inpWeather)
spotify <- fread(inpSpoti)
```
### Data exploration

Datatype data.table


```r
class(olympics)
```

```
## [1] "data.table" "data.frame"
```


## Subsetting rows
We can condition on the rows one time


```r
swimmers <- olympics[sport == "Swimming"]
```

and two or more times


```r
swimmers <- olympics[sport == "Swimming" & medal == "Gold"]
```

ordering data like


```r
var <- swimmers[order(height)]
```

Now we want to condition on the colums too


```r
cols <- c("athlete", "medal")
```


or let us use regualr expression to find someone


```r
ledecky <- swimmers[athlete %like% "Ledecky"]
```







