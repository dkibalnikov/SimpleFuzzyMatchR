
# Simple Fuzzy Match R
Simple vectorised Fuzzy Matching function in R based on Levenshtein distance

## Problem

Base R function `agrep()` has several limitation in case vectorize matching:

1. Returns all matches:  
1.1 Either with number of match (1)  
1.2 Either the match case (2)

``` r
fruits <- c("apple", "apples", "aple", "melone", "applejuice", "peanapple", "pear")

agrep("apple", fruits) #(1)
#>[1] 1 2 3 5 6

agrep("apple", fruits, value = TRUE) #(2)
#>[1] "apple"      "apples"     "aple"       "applejuice" "peanapple" 
```

2. Returns *"matches substrings of each element of x"* (see documentation `agrep()`).  
But someone is likely to like strict matching: "apple", "apples", ~~"aple", "applejuice", "peanapple"~~ 
3. Enythere someone also likes vectorize form where in **f(x, y)**: x, y are vectors for matching and **f(x, y)** returns vector of best matches from y in sequance of x
4. **f(x, y)** should return `NA` if Levenshtein distance of matching is more than some max value  

## Solution
Simple function `get_match()` based on `adist()`:
``` r
library("purrr")

get_match <- function(x, y, max = 3, cost = 1) {
  dist_mtrx <- adist(x, y, partial = FALSE, costs = cost) 
  dimnames(dist_mtrx) <- list(x, y)
  
  get_best <- function(n, dist_mtrx, max){
    min_dist <- dist_mtrx[n, ] %>% min()
    if(min_dist < max) res <- dist_mtrx[n, ] %>% which.min() %>% names()
    else res <- NA
    return(res)
  }
  
  1:nrow(dist_mtrx) %>% 
    map_chr(~get_best(., dist_mtrx, max = max))
}

#test result
fruits2 <- c("pears","apples", "melone", "per", "apple", "orange")
get_match(fruits2, fruits)

#>[1] "pear"   "apples" "melone" "pear"   "apple"  NA  
```
© 2020 GitHub, Inc.
