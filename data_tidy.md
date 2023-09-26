Tidy Data
================

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.2     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

## PULSE data with `pivot_longer`

``` r
pulse_df = 
  haven::read_sas("data/public_pulse_data.sas7bdat") |> 
  janitor::clean_names() |> 
  pivot_longer(
    bdi_score_bl:bdi_score_12m, ##these are the columns I want to change
    names_to = "obs_time", ##this is what I want the new column containing the new variable to be called
    values_to = "bdi_score", ##this is what I want the existing values to be stored as
    names_prefix = "bdi_score_" ##this will help me get rid of unecessary prefixes in the names column
  ) |> 
  mutate(
    obs_time = replace(obs_time, obs_time == "bl", "00m")
  ) ##I want to replace the "bl" value with "00m" for consistency
```

## Learning assessment

``` r
litters_df = 
  read_csv("data/FAS_litters.csv") |> 
  janitor::clean_names() |> 
  select(litter_number, gd0_weight, gd18_weight) |> 
  pivot_longer(
    gd0_weight:gd18_weight,
    names_to = "gd",
    values_to = "weight"
  ) |> 
  mutate(
    gd = case_match(
      gd,
      "gd0_weight" ~ 0,
      "gd18_weight" ~18
    ) ##case_match is good for replacing multiple values (not just one-offs)
  )
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

To answer, is this version tidy? - it depends! What are you trying to do
with the data after? If we wanted to create a weight gain variable, it
would be harder to do it this way. Form should follow function!

## LOTR data

This is going to be a bit trickier because there are three separate
tables in the excel sheet.

``` r
lotr_fellowship_df = 
  readxl::read_excel("data/LotR_Words.xlsx", range = "B3:D6") |>
  mutate(movie = "fellowship")

lotr_two_towers_df = 
  readxl::read_excel("data/LotR_Words.xlsx", range = "F3:H6") |> 
  mutate(movie = "two towers")

lotr_return_df = 
  readxl::read_excel("data/LotR_Words.xlsx", range = "J3:L6") |> 
  mutate(movie = "return of the king")

lotr_df = 
  bind_rows(lotr_fellowship_df, lotr_two_towers_df, lotr_return_df) |> 
  pivot_longer(
    Female:Male,
    names_to = "gender",
    values_to = "word"
    ) |> 
  relocate(
    movie
    )
```
