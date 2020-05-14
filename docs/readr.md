# Reading data: readr {#readr}



```r
library(readr)  # reading data (we could also load it via package tidyverse)
library(dplyr)  # wrangling data (we could also load it via package tidyverse) 
```

We are working with crudly formatted contract data (_vertragsdaten.csv_), to demonstrate the functionalites of the `readr` package. This is how the data looks like:


```r
readr::read_lines(file = "data/vertragsdaten.csv", n_max = 5)
```

```
## [1] "0135710150010017;01;3;1;01;         156,50 ;22022020;H      ;00 ;17D2;V709;"
## [2] "0136300200006011;01;3;1;01;          60,40 ;22022020;H      ;00 ;17D2;V709;"
## [3] "0136920590002028;01;3;1;01;           1,30 ;22022020;H      ;10 ;14H2;V113;"
## [4] "0171400040019017;01;3;2;01;          61,00 ;22022020;H      ;00 ;17D2;V113;"
## [5] "0272150100001011;07;3;1;01;       9.842,50 ;22022020;H      ;00 ; 662;V002;"
```

The Challenges are:

- Add column names
- Deal with leading and trailing blanks (which can have a meaning in some columns)
- Correctly specify the column types (numbers, date, character, ...)
- To correctly identify numbers, we need to set the German locale. Itherwise, the decimal mark won't be correctly identified.
- In order to create readible reports, it is sometimes helpful to display Euro amounts not as a number, but as a character that includes the Euro symbol (€) and comma/big mark signs


```r
df <- readr::read_delim(file = "data/vertragsdaten.csv", 
                 
                 # Specify semicolon as separator
                 delim = ";", 
                 
                 # Set the missing column names
                 col_names = c("VVT_NR", "Gebiet", "Abt", "Zw", "FK", "Beitr_Neu", "EDV_Dat", 
                               "Merkmal", "Kz_MB", "Sparte", "Bedingung", "leere_Spalte"),
                 
                 # Change data types which are not correctly imported by default
                 # We skip the last column, which turns out to be empty
                 col_types = cols(EDV_Dat = col_date(format = "%d%m%Y"), leere_Spalte = col_skip()), 
                 
                 # We set the German locale in order to get numbers and dates right
                 locale = locale(date_names = "de", decimal_mark = ",", grouping_mark = "."), 
                 
                 # White spaces are sometimes important, so we don't want to trim them
                 trim_ws = FALSE)

head(df)
```

```
## # A tibble: 5 x 11
##   VVT_NR Gebiet   Abt    Zw FK    Beitr_Neu EDV_Dat    Merkmal Kz_MB Sparte
##   <chr>  <chr>  <dbl> <dbl> <chr> <chr>     <date>     <chr>   <chr> <chr> 
## 1 01357~ 01         3     1 01    "       ~ 2020-02-22 "H    ~ "00 " "17D2"
## 2 01363~ 01         3     1 01    "       ~ 2020-02-22 "H    ~ "00 " "17D2"
## 3 01369~ 01         3     1 01    "       ~ 2020-02-22 "H    ~ "10 " "14H2"
## 4 01714~ 01         3     2 01    "       ~ 2020-02-22 "H    ~ "00 " "17D2"
## 5 02721~ 07         3     1 01    "       ~ 2020-02-22 "H    ~ "00 " " 662"
## # ... with 1 more variable: Bedingung <chr>
```


The column `Beitr_Neu` is not yet correctly recognized as numeric, due to the fact that we did not trim white spaces. Hence, we need to correct this column in a separate step.


```r
df <- df %>% 
  mutate(Beitr_Neu = readr::parse_number(Beitr_Neu, 
                                  
                                  # We trim White psaces
                                  trim_ws = TRUE,
                                  
                                  # Set the German number separator marks
                                  locale = locale(decimal_mark = ",", grouping_mark = "."), 
                                  
                                  ))

head(df)
```

```
## # A tibble: 5 x 11
##   VVT_NR Gebiet   Abt    Zw FK    Beitr_Neu EDV_Dat    Merkmal Kz_MB Sparte
##   <chr>  <chr>  <dbl> <dbl> <chr>     <dbl> <date>     <chr>   <chr> <chr> 
## 1 01357~ 01         3     1 01        156.  2020-02-22 "H    ~ "00 " "17D2"
## 2 01363~ 01         3     1 01         60.4 2020-02-22 "H    ~ "00 " "17D2"
## 3 01369~ 01         3     1 01          1.3 2020-02-22 "H    ~ "10 " "14H2"
## 4 01714~ 01         3     2 01         61   2020-02-22 "H    ~ "00 " "17D2"
## 5 02721~ 07         3     1 01       9842.  2020-02-22 "H    ~ "00 " " 662"
## # ... with 1 more variable: Bedingung <chr>
```

