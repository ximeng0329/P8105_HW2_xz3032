p8105\_hw2\_xz3032
================

This is the homework for data science module 3: data wrangling

``` {r}
library(tidyverse)
library(readxl)
```

# Problem 1

Read the Mr. Trashwheel dataset

``` {r}
trashwheel_df = 
  read_xlsx(
    "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
    sheet = 1,
    range = "A2:N408" ) %>%
    janitor::clean_names() %>%
    drop_na(dumpster) %>%
    mutate(
      sports_balls = round(sports_balls,0),
      sports_balls = as.integer(sports_balls)
    )
```

Read precipitation data for 2018 and 2017

``` {r}
p2018 = read_excel("./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
                   sheet = "2018 Precipitation",
                   skip = 1) %>%
  janitor::clean_names() %>%
  drop_na(month) %>%
  mutate(year = 2018) %>%
  relocate(year)

p2017 = read_excel("./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
                   sheet = "2017 Precipitation",
                   skip = 1) %>%
  janitor::clean_names() %>%
  drop_na(month) %>%
  mutate(year = 2017) %>%
  relocate(year)
```

Now combine the annual precipitation dataframe

``` {r}
month_df = tibble(
  month = 1:12,
  month_name = month.abb
)

precip_df = bind_rows(p2018, p2017)

precip_df = left_join(precip_df, month_df, by = "month")
```

This dataset contains information from the Mr. Trashwheel trash
collector in Baltimore, Maryland. As trash enters the inner harbor, the
trashwheel collects that trash, and stores it in a dumpster. The dataset
contains information on year, month, and trash collected, include some
specific kinds of trash. There are a total of `r nrow(trashwheel_df)`
rows in our final dataset. Additional data sheets include month
precipitation data. In this dataset:

  - The median number of sports balls found in a dumpster in 2017 was `r
    trashwheel_df %>% filter(year == 2017) %>% pull(sports_balls) %>%
    median()`
  - The total precipitation in 2018 was `r precip_df %>% filter(year
    == 2018) %>% pull(total) %>% sum()` inches.

\#Problem 2

Read NYC transit data

``` {r}
transit_df = 
  read_csv("./data/NYC_Transit_Subway_Entrance_And_Exit_Data.csv") %>%
  janitor::clean_names() %>%
  select(line, station_name, station_latitude, station_longitude, entry, vending, entrance_type, ada, starts_with("route")) %>%
  mutate(entry = recode(entry, `YES` = TRUE , `NO` = FALSE)) %>%
  mutate(route8 = as.character(route8),
         route9 = as.character(route9),
         route10 = as.character(route10),
         route11 = as.character(route11)) %>%
  pivot_longer(
    route1:route11,
    names_to = "route_name",
    names_prefix = "route" ,
    values_to = "route_number"
  )
```

# Short Descripe of the Dataset

The data set contains variables: `r names(transit_df)`. I first cleaned
the data by gave variables clean names using janitor::clean\_names
function. Then I select the variables I need. I also changed the entry
variables from character to logical. Finally I combined routes variables
and made the data from into a longer version. The data set dimension is
`r nrow(transit_df)`\*`r ncol(transit_df)`.

# Answer Questions

How many distinct stations are there? (465)

``` {r}
select(transit_df,line,station_name,ada) %>%
  distinct(line, station_name) %>%
  count()
```

How many stations are ADA compliance? (84)

``` {r}
select(transit_df,line,station_name,ada) %>%
  filter(ada == TRUE) %>%
  distinct(line, station_name) %>%
  count()
```

What proportion of station entrances / exits without vending allow
entrance? (759 out of 2013)

``` {r}
count_1 = select(transit_df, entry, vending) %>%
                filter(entry == TRUE) %>%
                filter(vending == "NO") %>%
                count()
count_2 = select(transit_df, entry, vending) %>%
                filter(vending == "NO") %>%
                count()
count_1/count_2
```

## Problem 3

Clean pols-monnth data

``` {r}
pols_month = read_csv("./data/fivethirtyeight_datasets/pols-month.csv") %>%
  janitor::clean_names() %>%
  separate(mon, into = c("year", "month", "day")) %>%
  mutate(year = as.integer(year),
         month = month.abb[as.factor(month)]) %>%
  mutate(president = case_when(prez_dem == 1 ~ "prez_dem", prez_gop==1  ~ "prez_gop")) %>%
  select(-day, -prez_gop, -prez_dem)
```

Clean snp data

``` {r}
snp_df = read_csv("./data/fivethirtyeight_datasets/snp.csv") %>%
  janitor::clean_names() %>%
  separate(date, into = c("month", "day", "year")) %>%
    mutate(year = as.integer(year),
         month = month.abb[as.factor(month)]
         ) %>%
  select(-day) %>%
  relocate(year)
```

Clean unemployment data

``` {r}
unemploy_df = read_csv("./data/fivethirtyeight_datasets/unemployment.csv") %>%
  janitor::clean_names() %>%
  pivot_longer(
    jan:dec,
    names_to = "month",
    values_to = "unemployment"
  ) %>%
   mutate(year = as.integer(year),
          month = month.abb[as.factor(month)])
```

Merge snp into rows, and then merge with unemployment dataset

``` {r}
snp_pols = left_join(pols_month, snp_df, by = c("year", "month"))

final_df = left_join(snp_pols, unemploy_df, by = c("year", "month"))
```

Short Description: “pols-month” has information related to the number of
national politicians who are democratic or republican at any given time.
variables including `r names(pols_month)`. The file “snp” contains 2
variables related to Standard & Poor’s stock market index (S\&P),
varibales are `r names(snp_df)`. The unemployment data frame contains
two variables including `r names(unemploy_df)`. I combined the three
data frames by their same key variables month and year. The final
product is a data set with dimension `r nrow(final_df)` \* `r
ncol(final_df)`. The data are from year `r range(final_df$year)`.
