# Wrangling data: dplyr {#dplyr}



```r
library(tidyverse)
```



```r
df <- read_csv("data/diamonds.csv")
```

```
## Parsed with column specification:
## cols(
##   carat = col_double(),
##   cut = col_character(),
##   color = col_character(),
##   clarity = col_character(),
##   depth = col_double(),
##   table = col_double(),
##   price = col_double(),
##   x = col_double(),
##   y = col_double(),
##   z = col_double()
## )
```

```r
df %>% names()
```

```
##  [1] "carat"   "cut"     "color"   "clarity" "depth"   "table"   "price"  
##  [8] "x"       "y"       "z"
```

```r
df %>% group_by(color) %>% summarise(MEANPRICE = mean(price))
```

```
## # A tibble: 7 x 2
##   color MEANPRICE
##   <chr>     <dbl>
## 1 D         3170.
## 2 E         3077.
## 3 F         3725.
## 4 G         3999.
## 5 H         4487.
## 6 I         5092.
## 7 J         5324.
```


