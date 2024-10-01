01OCT24_viz_and_eda_2
================
Stella Koo
2024-10-01

## Explatory Data Analysis (EDA)

Analyze and investigate data sets and summarize their main
characteristics, often employing data visualization methods. Discover
patterns, spot anomalies, test a hypothesis, or check assumptions.

- Conduct EDA using `dplyr` verbs (`group_by` and `summarize`)

`rnoaa::meteo_pull_monitors()` pulls weather data from National Oceanic
and Atmospheric Administration database (NOAA) for a list of weather
stations.

``` r
library(tidyverse)

weather_df = 
  rnoaa::meteo_pull_monitors(
    c("USW00094728", "USW00022534", "USS0023B17S"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2021-01-01",
    date_max = "2022-12-31") |>
  mutate(
    name = case_match(
      id, 
      "USW00094728" ~ "CentralPark_NY", 
      "USW00022534" ~ "Molokai_HI",
      "USS0023B17S" ~ "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10,
    month = lubridate::floor_date(date, unit = "month")) |>
  select(name, id, everything())
```

### Initial Numeric Explorations

``` r
weather_df |> 
  ggplot(aes(x = prcp)) + 
  geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](01OCT24_viz_and_eda_2_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Very skewed distribution. Majority of days have no precipitation.
Examining the relatively few days have very high precipitation might be
helpful.

``` r
weather_df |> 
  filter(prcp >= 1000)
```

    ## # A tibble: 3 × 7
    ##   name           id          date        prcp  tmax  tmin month     
    ##   <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ## 1 CentralPark_NY USW00094728 2021-08-21  1130  27.8  22.8 2021-08-01
    ## 2 CentralPark_NY USW00094728 2021-09-01  1811  25.6  17.2 2021-09-01
    ## 3 Molokai_HI     USW00022534 2022-12-18  1120  23.3  18.9 2022-12-01

``` r
weather_df |> 
  filter(tmax >= 20, tmax <= 30) |> 
  ggplot(aes(x = tmin, y = tmax, color = name, shape = name)) + 
  geom_point(alpha = .75)
```

![](01OCT24_viz_and_eda_2_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

The previous scatterplot shows that Central Park and Molokai report
temperature values differently from Waterhole.

### `group_by`

Only using `group_by()` without a function like `summarize()`,
`mutate()`, or `filter()`, it won’t change or reduce the data. The data
will be grouped internally, but all the rows will still be displayed.

- `ungroup()` will remove groups.

``` r
weather_df |>
  group_by(name, month)
```

    ## # A tibble: 2,190 × 7
    ## # Groups:   name, month [72]
    ##    name           id          date        prcp  tmax  tmin month     
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl> <date>    
    ##  1 CentralPark_NY USW00094728 2021-01-01   157   4.4   0.6 2021-01-01
    ##  2 CentralPark_NY USW00094728 2021-01-02    13  10.6   2.2 2021-01-01
    ##  3 CentralPark_NY USW00094728 2021-01-03    56   3.3   1.1 2021-01-01
    ##  4 CentralPark_NY USW00094728 2021-01-04     5   6.1   1.7 2021-01-01
    ##  5 CentralPark_NY USW00094728 2021-01-05     0   5.6   2.2 2021-01-01
    ##  6 CentralPark_NY USW00094728 2021-01-06     0   5     1.1 2021-01-01
    ##  7 CentralPark_NY USW00094728 2021-01-07     0   5    -1   2021-01-01
    ##  8 CentralPark_NY USW00094728 2021-01-08     0   2.8  -2.7 2021-01-01
    ##  9 CentralPark_NY USW00094728 2021-01-09     0   2.8  -4.3 2021-01-01
    ## 10 CentralPark_NY USW00094728 2021-01-10     0   5    -1.6 2021-01-01
    ## # ℹ 2,180 more rows

### Counting things

#### `group_by()` and `summarize()`

Counting number of observations in each location in the complete
dataset:

``` r
weather_df |>
  group_by(name) |>
  summarize(n_obs = n())
```

    ## # A tibble: 3 × 2
    ##   name           n_obs
    ##   <chr>          <int>
    ## 1 CentralPark_NY   730
    ## 2 Molokai_HI       730
    ## 3 Waterhole_WA     730

Can group by more than one variable (e.g each location and month):

- `n()` counts number of rows.

``` r
weather_df |>
  group_by(name, month) |>
  summarize(n_obs = n()) 
```

    ## `summarise()` has grouped output by 'name'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 72 × 3
    ## # Groups:   name [3]
    ##    name           month      n_obs
    ##    <chr>          <date>     <int>
    ##  1 CentralPark_NY 2021-01-01    31
    ##  2 CentralPark_NY 2021-02-01    28
    ##  3 CentralPark_NY 2021-03-01    31
    ##  4 CentralPark_NY 2021-04-01    30
    ##  5 CentralPark_NY 2021-05-01    31
    ##  6 CentralPark_NY 2021-06-01    30
    ##  7 CentralPark_NY 2021-07-01    31
    ##  8 CentralPark_NY 2021-08-01    31
    ##  9 CentralPark_NY 2021-09-01    30
    ## 10 CentralPark_NY 2021-10-01    31
    ## # ℹ 62 more rows

Can compute multiple summaries within each group:

``` r
weather_df |>
  group_by(month) |>
  summarize(
    n_obs = n(),
    n_days = n_distinct(date))
```

    ## # A tibble: 24 × 3
    ##    month      n_obs n_days
    ##    <date>     <int>  <int>
    ##  1 2021-01-01    93     31
    ##  2 2021-02-01    84     28
    ##  3 2021-03-01    93     31
    ##  4 2021-04-01    90     30
    ##  5 2021-05-01    93     31
    ##  6 2021-06-01    90     30
    ##  7 2021-07-01    93     31
    ##  8 2021-08-01    93     31
    ##  9 2021-09-01    90     30
    ## 10 2021-10-01    93     31
    ## # ℹ 14 more rows

To tabulate the frequency of a binary outcome across levels of a binary
predictor:

- E.g. Number of cold and not-cold days in Central Park and Waterhole.

``` r
weather_df |> 
  drop_na(tmax) |> 
  mutate(
    cold = case_when(
      tmax <  5 ~ "cold",
      tmax >= 5 ~ "not_cold",
      TRUE      ~ ""
  )) |> 
  filter(name != "Molokai_HI") |> 
  group_by(name, cold) |> 
  summarize(count = n())
```

    ## `summarise()` has grouped output by 'name'. You can override using the
    ## `.groups` argument.

    ## # A tibble: 4 × 3
    ## # Groups:   name [2]
    ##   name           cold     count
    ##   <chr>          <chr>    <int>
    ## 1 CentralPark_NY cold        96
    ## 2 CentralPark_NY not_cold   634
    ## 3 Waterhole_WA   cold       319
    ## 4 Waterhole_WA   not_cold   395

Can also re-organize into a more standard (non-tidy) 2x2 table using
`pivot_wider` or `janitor::tabyl`:

``` r
weather_df |> 
  drop_na(tmax) |> 
  mutate(cold = case_when(
    tmax <  5 ~ "cold",
    tmax >= 5 ~ "not_cold",
    TRUE     ~ ""
  )) |> 
  filter(name != "Molokai_HI") |> 
  janitor::tabyl(name, cold)
```

    ##            name cold not_cold
    ##  CentralPark_NY   96      634
    ##    Waterhole_WA  319      395

#### `count()`

Can also use `count()` instead of `group_by()` and `summarize()`:

- Creates a dataframe that can be manipulated directly.

``` r
weather_df |>
  count(month, name)
```

    ## # A tibble: 72 × 3
    ##    month      name               n
    ##    <date>     <chr>          <int>
    ##  1 2021-01-01 CentralPark_NY    31
    ##  2 2021-01-01 Molokai_HI        31
    ##  3 2021-01-01 Waterhole_WA      31
    ##  4 2021-02-01 CentralPark_NY    28
    ##  5 2021-02-01 Molokai_HI        28
    ##  6 2021-02-01 Waterhole_WA      28
    ##  7 2021-03-01 CentralPark_NY    31
    ##  8 2021-03-01 Molokai_HI        31
    ##  9 2021-03-01 Waterhole_WA      31
    ## 10 2021-04-01 CentralPark_NY    30
    ## # ℹ 62 more rows

``` r
weather_df |>
  count(month, name = "n_obs")
```

    ## # A tibble: 24 × 2
    ##    month      n_obs
    ##    <date>     <int>
    ##  1 2021-01-01    93
    ##  2 2021-02-01    84
    ##  3 2021-03-01    93
    ##  4 2021-04-01    90
    ##  5 2021-05-01    93
    ##  6 2021-06-01    90
    ##  7 2021-07-01    93
    ##  8 2021-08-01    93
    ##  9 2021-09-01    90
    ## 10 2021-10-01    93
    ## # ℹ 14 more rows

#### `table()` function

Base R’s `table` function also produces summaries. table\`’s output is
of class table and is hard to do any additional work with.

``` r
weather_df |>
  pull(month) |>
  table()
```

    ## 
    ## 2021-01-01 2021-02-01 2021-03-01 2021-04-01 2021-05-01 2021-06-01 2021-07-01 
    ##         93         84         93         90         93         90         93 
    ## 2021-08-01 2021-09-01 2021-10-01 2021-11-01 2021-12-01 2022-01-01 2022-02-01 
    ##         93         90         93         90         93         93         84 
    ## 2022-03-01 2022-04-01 2022-05-01 2022-06-01 2022-07-01 2022-08-01 2022-09-01 
    ##         93         90         93         90         93         93         90 
    ## 2022-10-01 2022-11-01 2022-12-01 
    ##         93         90         93

### General Summaries

Standard statistical summaries are regularly computed in `summarize()`
using functions like `mean()`, `median()`, `var()`, `sd()`, `mad()`,
`IQR()`, `min()`, and `max()`. To use these, indicate the variable to
which they apply and include any additional arguments as necessary.

``` r
weather_df |>
  group_by(month) |>
  summarize(
    mean_tmax = mean(tmax, na.rm = TRUE),
    mean_prec = mean(prcp, na.rm = TRUE),
    median_tmax = median(tmax),
    sd_tmax = sd(tmax))
```

    ## # A tibble: 24 × 5
    ##    month      mean_tmax mean_prec median_tmax sd_tmax
    ##    <date>         <dbl>     <dbl>       <dbl>   <dbl>
    ##  1 2021-01-01     10.9       39.5         5     12.2 
    ##  2 2021-02-01      9.82      42.6         2.8   12.2 
    ##  3 2021-03-01     13.7       55.5        NA     NA   
    ##  4 2021-04-01     16.8       14.7        18.0    9.29
    ##  5 2021-05-01     19.6       17.3        22.2    9.40
    ##  6 2021-06-01     24.3       14.1        28.3    8.28
    ##  7 2021-07-01     25.2       30.7        28.3    6.19
    ##  8 2021-08-01     25.2       30.6        28.3    6.72
    ##  9 2021-09-01     22.4       35.4        24.4    7.69
    ## 10 2021-10-01     18.2       36.0        20.6   10.1 
    ## # ℹ 14 more rows
