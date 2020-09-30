---
title: "p8105_hw2_xz3032"
output: html_document
---

```{r}
library(tidyverse)
library(readxl)
```

##Problem 1

Read the Mr. Trashwheel dataset

```{r}
trashwheel_df = 
  read_xlsx(
    "~/Desktop/DataScience/Trash-Wheel-Collection-Totals-8-6-19.xlsx",
    sheet = 1,
    range = "A2:N408" )%>%
    janitor::clean_names() %>%
    drop_na(dumpster) %>%
    mutate(
      sports_balls = round(sports_balls,0),
      sports_balls = as.integer(sports_balls)
    )
```


##Problem 2

Read NYC transit data

```{r}
transit_df = 
  read_csv("~/Desktop/DataScience/NYC_Transit_Subway_Entrance_And_Exit_Data.csv") %>%
  janitor::clean_names() %>%
  select(line, station_name, station_latitude, station_longitude, entry, vending, entrance_type, ada) 
```

## Short Describe of the Dataset

```{r}
The Dataset contains `r names(transit_df)` 
```