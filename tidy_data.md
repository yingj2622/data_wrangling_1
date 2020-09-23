Tidy Data
================

``` r
library(tidyverse)
```

    ## -- Attaching packages ------------------------------ tidyverse 1.3.0 --

    ## √ ggplot2 3.3.2     √ purrr   0.3.4
    ## √ tibble  3.0.3     √ dplyr   1.0.2
    ## √ tidyr   1.1.2     √ stringr 1.4.0
    ## √ readr   1.3.1     √ forcats 0.5.0

    ## -- Conflicts --------------------------------- tidyverse_conflicts() --
    ## x dplyr::filter() masks stats::filter()
    ## x dplyr::lag()    masks stats::lag()

## povit\_longer

Load the PULSE data,

``` r
pulse_data = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names()
```

    ## Warning in FUN(X[[i]], ...): strings not representable in native encoding will
    ## be translated to UTF-8

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00DF>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C5>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E5>' to native encoding

Wide format to long format

``` r
pulse_data_tidy = 
  pulse_data %>%
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  )
```

Rewrite, combine and extend(to add a mutate)

``` r
pulse_data = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names() %>%
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  relocate(id,visit) %>% 
  mutate(visit = recode(visit,"bl" = "oom"))
```

## pivot\_wider

Make up some data

``` r
analysis_result = tibble(
  group = c("treatment","treatment","placebo","placebo"),
  time = c("pre","post","pre","post"),
  mean = c(4,8,3.5,4)
)

analysis_result %>% 
  pivot_wider(
    names_from = 'time',
    values_from = "mean"
  )
```

    ## # A tibble: 2 x 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

## Binding rows

Using the LotR data

First step: import each table.

``` r
fellowship_ring = readxl::read_excel("./data/LotR_Words.xlsx",range = "B3:d6") %>% 
  mutate(movie = "fellowship_ring")

two_towers = readxl::read_excel("./data/LotR_Words.xlsx",range = "F3:H6") %>% 
  mutate(movie = "two_towers")

return_king = readxl::read_excel("./data/LotR_Words.xlsx",range = "j3:l6") %>% 
  mutate(movie = "return_king")
```

Bind all the row together

``` r
lotr_tidy = bind_rows(fellowship_ring,two_towers,return_king) %>% 
  janitor::clean_names() %>% 
  relocate(movie) %>% 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words")
```

## joining datasets

Import and clean the FAS datasets

``` r
pups_df = 
  read_csv("./data/FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex,`1` = "male",`2` = "female"))
```

    ## Parsed with column specification:
    ## cols(
    ##   `Litter Number` = col_character(),
    ##   Sex = col_double(),
    ##   `PD ears` = col_double(),
    ##   `PD eyes` = col_double(),
    ##   `PD pivot` = col_double(),
    ##   `PD walk` = col_double()
    ## )

``` r
litters_df = 
  read_csv("./data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  relocate("litter_number") %>% 
  separate(group, into = c("does","day_of_tx"),sep = 3)
```

    ## Parsed with column specification:
    ## cols(
    ##   Group = col_character(),
    ##   `Litter Number` = col_character(),
    ##   `GD0 weight` = col_double(),
    ##   `GD18 weight` = col_double(),
    ##   `GD of Birth` = col_double(),
    ##   `Pups born alive` = col_double(),
    ##   `Pups dead @ birth` = col_double(),
    ##   `Pups survive` = col_double()
    ## )

Join them

``` r
fas_df =
  left_join(pups_df,litters_df,by = "litter_number") %>% 
  arrange(litter_number) %>% 
  relocate(litter_number,sex,does,day_of_tx)
```
