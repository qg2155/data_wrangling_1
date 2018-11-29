data\_import
================
Helen Guan

``` r
library(tidyverse)
```

    ## ── Attaching packages ───────────────────────────────────────────────────────── tidyverse 1.2.1 ──

    ## ✔ ggplot2 3.0.0     ✔ purrr   0.2.5
    ## ✔ tibble  1.4.2     ✔ dplyr   0.7.6
    ## ✔ tidyr   0.8.1     ✔ stringr 1.3.1
    ## ✔ readr   1.1.1     ✔ forcats 0.3.0

    ## ── Conflicts ──────────────────────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()

``` r
library(rvest)
```

    ## Loading required package: xml2

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:purrr':
    ## 
    ##     pluck

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
library(httr)
```

### Scraping a table

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_xml = read_html(url)
```

Get the tables from HTML

``` r
drug_use_xml %>%
  html_nodes(css = "table") %>% 
  .[[1]] %>%  #give me the first element (table) of that list
  html_table() %>% 
  slice(-1) %>%  #take out the row with notes -1 because first row of table
  as_tibble() #make list into a dataframe
```

    ## # A tibble: 56 x 16
    ##    State `12+(2013-2014)` `12+(2014-2015)` `12+(P Value)` `12-17(2013-201…
    ##    <chr> <chr>            <chr>            <chr>          <chr>           
    ##  1 Tota… 12.90a           13.36            0.002          13.28b          
    ##  2 Nort… 13.88a           14.66            0.005          13.98           
    ##  3 Midw… 12.40b           12.76            0.082          12.45           
    ##  4 South 11.24a           11.64            0.029          12.02           
    ##  5 West  15.27            15.62            0.262          15.53a          
    ##  6 Alab… 9.98             9.60             0.426          9.90            
    ##  7 Alas… 19.60a           21.92            0.010          17.30           
    ##  8 Ariz… 13.69            13.12            0.364          15.12           
    ##  9 Arka… 11.37            11.59            0.678          12.79           
    ## 10 Cali… 14.49            15.25            0.103          15.03           
    ## # ... with 46 more rows, and 11 more variables: `12-17(2014-2015)` <chr>,
    ## #   `12-17(P Value)` <chr>, `18-25(2013-2014)` <chr>,
    ## #   `18-25(2014-2015)` <chr>, `18-25(P Value)` <chr>,
    ## #   `26+(2013-2014)` <chr>, `26+(2014-2015)` <chr>, `26+(P Value)` <chr>,
    ## #   `18+(2013-2014)` <chr>, `18+(2014-2015)` <chr>, `18+(P Value)` <chr>

``` r
url = "https://www.bestplaces.net/cost_of_living/city/new_york/new_york"

nyc_cost_xml = read_html(url)
```

``` r
nyc_cost_xml %>% 
  html_nodes(css = "table") %>%
  .[[1]] %>% 
  html_table(header = TRUE) 
```

    ##     COST OF LIVING New York New York      USA
    ## 1          Overall      180      122      100
    ## 2          Grocery      125    111.6      100
    ## 3           Health      110      109      100
    ## 4          Housing      313      145      100
    ## 5 Median Home Cost $662,100 $282,000 $216,200
    ## 6        Utilities      128      118      100
    ## 7   Transportation      107      110      100
    ## 8    Miscellaneous      120      110      100

### Harry Potter

``` r
hpsaga_html = read_html("https://www.imdb.com/list/ls000630791/") 
 
titles = hpsaga_html %>% 
    html_nodes(css = ".lister-item-header a") %>% 
    html_text()

money = hpsaga_html %>% 
    html_nodes(css = ".text-small:nth-child(7) span:nth-child(5)") %>% 
    html_text() 

hpsaga_df = tibble(
  title = titles,
  gross_rev = money
  ) 
```

Napolean dynamite

``` r
amazon_html = read_html("https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1")

review_titles = amazon_html %>% 
   html_nodes("#cm_cr-review_list .review-title") %>%
  html_text()

review_ratings = amazon_html %>% 
  html_nodes(css = "#cm_cr-review_list .review-rating") %>% 
  html_text()
  
review_text = amazon_html %>%
    html_nodes(".review-data:nth-child(4)") %>%
    html_text()

reviews = tibble(
  title = review_titles,
  stars = review_ratings,
  text = review_text
)
```

APIs
----

Get water data

As CSV

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/waf7-5gvc.csv") %>%
  content("parsed") # guess what class each variables are 
```

    ## Parsed with column specification:
    ## cols(
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_integer(),
    ##   year = col_integer()
    ## )

As JSON

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/waf7-5gvc.json") %>% 
  content("text") %>% 
  jsonlite::fromJSON() %>%
  as_tibble()
```

Pokemon API

``` r
poke = GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()
```
