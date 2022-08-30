---

title: "Life Expectancy"
summary: "TidyTuesday Week 14 2018 data visualisations."
author: "Tom Jenkins"
date: "2022-08-30"
# Header image (featured.png)
# To use, add an image named `featured.jpg/png` to your page's folder. 
tags: ["R", "TidyTuesday", "ECharts"]
weight: 2
featured: true
image:
  preview_only: true

---

<br/>

Data visualisation for [TidyTuesday](https://github.com/rfordatascience/tidytuesday) Week 14 2018. The data set is available on the TidyTuesday GitHub [page](https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-07-03) and the R code used to create the interactive graphics is available at the end of this page.

<br/>

<iframe height="500px" width="100%" frameborder="no" 
src="03_life_continents.html"></iframe>

<br/>

<iframe height="500px" width="100%" frameborder="no" 
src="03_life_europe.html"></iframe>

<br/>

<iframe height="500px" width="100%" frameborder="no" 
src="03_life_uk.html"></iframe>

<style>
pre {
  max-height: 500px;
  overflow-y: auto;
}
</style>

<br/>
<br/>
<br/>

```
# -------------------------- #
#
# ECharts Data Visualisations ####
#
# Data from Tidy Tuesday:
# https://github.com/rfordatascience/tidytuesday
# Week 14 2018
# Life Expectancy
#
# -------------------------- #

# Load packages
library(tidyverse)
library(htmlwidgets)
library(echarts4r)
library(echarts4r.assets)

# Import data
# https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-07-03
life_df = read_csv("tidytuesday_data/2018-07-03-global_life_expectancy.csv")

# Round life expectancy to one decimal place
life_df = mutate(life_df, life_expectancy = round(life_expectancy, digits = 1))
life_df

# Summaries
n_distinct(life_df$country)
range(life_df$year)
range(life_df$life_expectancy)


# ----------------- #
#
# Line Race: Continent ####
#
# ----------------- #

# Extract data at a continent level
continents = c("Africa","Asia","Europe","Northern America","Latin America and the Caribbean","Oceania","World")

# Extract data at a continent level
continents = c("Africa","Asia","Europe","Northern America","Latin America and the Caribbean","Oceania","World")
life_continent = life_df %>%
  filter(country %in% continents) %>%
  select(-code, continent = country) %>%
  mutate(continent = str_replace_all(continent, "Latin America and the Caribbean", "Latin America"))
life_continent

# Default EChart palette
default_pal = c("#5470c6","#91cc75","#fac858","#ee6666","#73c0de","#3ba272","#fc8452")

# Add colours for each continent to tibble
life_continent = life_continent %>%
  mutate(
    continent_cols = case_when(
      continent == "Africa" ~ default_pal[1],
      continent == "Asia" ~ default_pal[2],
      continent == "Europe" ~ default_pal[3],
      continent == "Northern America" ~ default_pal[4],
      continent == "Latin America" ~ default_pal[5],
      continent == "Oceania" ~ default_pal[6],
      continent == "World" ~ default_pal[7]
    )
  )
life_continent

# Plot line chart
plt_continent = life_continent %>%
  group_by(continent) %>%
  e_charts(x = year, renderer = "svg") %>%
  e_x_axis(type = "category", margin = 50) %>%
  e_line(
    serie = life_expectancy,
    legend = TRUE,
    showSymbol = FALSE,
    endLabel = list(
      show = TRUE,
      fontSize = 14,
      fontWeight = "bold",
      color = "inherit",
      formatter = "{a}: {@[1]}",
      valueAnimation = TRUE
    )
  ) %>%
  e_add_nested("lineStyle", continent_cols) %>%
  e_axis_labels(x = "Year") %>%
  e_title(
    text = "Global Life Expectancy (1950–2015)",
    subtext = "Average age of death"
  ) %>%
  e_legend(
    selectedMode = FALSE,
    width = 380,
    textStyle = list(
      fontSize = 12
    )
  ) %>%
  e_animation(duration = 7500) %>%
  e_tooltip(
    trigger = "axis",
    order = "valueDesc",
    axisPointer = list(
      label = list(
        show = TRUE
      )
    )
  ) %>%
  e_grid(right = 200) %>%
  e_toolbox_feature("saveAsImage", title = ".svg", name = "image")
plt_continent
saveWidget(plt_continent, file = "widgets/03_life_continents.html")


# ----------------- #
#
# UK Life Expectancy ####
#
# ----------------- #

# Extract data for United Kingdom
life_uk = life_df %>%
  filter(country %in% c("United Kingdom"))
life_uk

# JS function to ensure no commas in x axis
format_xaxis = htmlwidgets::JS(
  'function(value){
return value.toString();
}'
)

# Colours
uk_col = "#c1232b"
war_col = "#27727b"
plague_col = "#26c0c0"
flu_col = "#fe8463"
un_col = "#fad860"

# Plot line chart
plt_uk = life_uk %>%
  e_charts(x = year, renderer = "svg") %>%
  e_line(
    serie = life_expectancy,
    name = "United Kingdom",
    legend = TRUE,
    showSymbol = FALSE,
    # Line dynamic labelling
    endLabel = list(
      show = TRUE,
      fontSize = 15,
      fontWeight = "bold",
      color = "inherit",
      formatter = "{@[0]}\n{@[1]}",
      valueAnimation = TRUE
    )
  ) %>%
  # Line colour
  e_color(uk_col) %>%
  # X axis parameters
  e_x_axis(
    type = "value",
    axisLabel = list(formatter = format_xaxis),
    splitLine = list(show = FALSE),
    min = "dataMin",
    max = "dataMax"
  ) %>%
  e_axis_labels(x = "Year") %>%
  # Y axis parameters
  e_y_axis(
    axisLine = list(show = FALSE),
    axisTick = list(show = FALSE)
  ) %>%
  # Animation options
  e_animation(show = FALSE) %>%
  # Title and Subtitle
  e_title(
    text = "UK Life Expectancy (1543–2015)",
    subtext = "Average age of death"
  ) %>%
  # Legend options
  e_legend(selectedMode = FALSE) %>%
  # Toolbox features
  e_toolbox_feature("dataZoom", title = list(zoom = "Zoom", back = "Reset")) %>%
  e_toolbox_feature("saveAsImage", title = ".svg", name = "image") %>%
  # Mark period of the First World War
  e_mark_area(
    data = list(
      list(xAxis = 1914, yAxis = 0, name = "World War I",
           itemStyle = list(color = war_col, opacity = 0.5),
           label = list(fontWeight = "bold", offset = c(-30, 0), color = war_col)
      ),
      list(xAxis = 1918, yAxis = 100)
    )
  ) %>%
  # Mark period of the Second World War
  e_mark_area(
    data = list(
      list(xAxis = 1939, yAxis = 0, name = "World War II",
           itemStyle = list(color = war_col, opacity = 0.5),
           label = list(fontWeight = "bold", offset = c(30, 0), color = war_col)
      ),
      list(xAxis = 1945, yAxis = 100)
    )
  ) %>%
  # Mark Spanish flu outbreak
  e_mark_point(
    data = list(
      symbol = "arrow",
      symbolSize = 25,
      value = "Spanish Flu Outbreak",
      xAxis = 1918, yAxis = 47,
      itemStyle = list(
        color = flu_col
      ),
      label = list(
        color = "white",
        textBorderColor = flu_col,
        textBorderWidth = 3,
        fontSize = 12,
        fontWeight = "bold",
        offset = c(0, 30)
      )
    )
  ) %>%
  # Mark Great Plague of London
  e_mark_point(
    data = list(
      symbol = "arrow",
      symbolSize = 25,
      symbolRotate = -180,
      symbolOffset = c(0,-30),
      value = "Great Plague of London",
      xAxis = 1665, yAxis = 32,
      itemStyle = list(
        color = plague_col
      ),
      label = list(
        color = "white",
        textBorderColor = plague_col,
        textBorderWidth = 3,
        fontSize = 12,
        fontWeight = "bold",
        offset = c(0,-30)
      )
    )
  ) %>%
  # Unknown event 1
  e_mark_point(
    data = list(
      symbol = "pin",
      symbolSize = 50,
      symbolRotate = -180,
      value = "?",
      xAxis = 1558, yAxis = 22.4,
      itemStyle = list(
        color = un_col
      ),
      label = list(
        color = "white",
        fontSize = 25,
        fontWeight = "bold",
        offset = c(0,10)
      )
    )
  ) %>%
  # Unknown event 2
  e_mark_point(
    data = list(
      symbol = "pin",
      symbolSize = 50,
      symbolRotate = -180,
      value = "?",
      xAxis = 1728, yAxis = 25.3,
      itemStyle = list(
        color = un_col
      ),
      label = list(
        color = "white",
        fontSize = 25,
        fontWeight = "bold",
        offset = c(0,10)
      )
    )
  ) %>% 
  # Button to zoom to 1900-1960
  e_zoom(
    dataZoomIndex = 0,
    startValue = 1900,
    endValue = 1960,
    btn = "zoomBtn"
  ) %>% 
  e_button(
    id = "zoomBtn",
    position = "top",
    class = "btn btn-primary",
    "Zoom to 1900-1960"
  ) %>% 
  # Button to restore
  e_restore(btn = "restoreBtn") %>% 
  e_button(
    id = "restoreBtn",
    position = "top",
    class = "btn btn-primary",
    "Reset"
  )
plt_uk
saveWidget(plt_uk, file = "widgets/03_life_uk.html")


# ----------------- #
#
# Europe Life Expectancy ####
#
# ----------------- #

# Countries to subset
europe = c("United Kingdom","Ireland","Norway","Finland","France","Germany","Sweden",
           "Portugal","Spain","Italy","Switzerland","Netherlands","Denmark","Belgium",
           "Luxembourg","Monaco","Austria","Poland","Luxembourg","Liechtenstein")

# Years to subset
years = seq(1900, 2015, by = 5)

# Extract data for Europe
life_europe = life_df %>%
  # Subset countries
  filter(country %in% europe) %>% 
  # Subset years
  filter(year %in% years) %>% 
  mutate(year = as.factor(year)) %>% 
  select(-code)
life_europe

# Plot timeline bar chart
plt_europe = life_europe %>% 
  group_by(year) %>% 
  e_charts(x = country, timeline = TRUE, renderer = "svg") %>% 
  e_bar(
    serie = life_expectancy,
    realtimeSort = TRUE,
    itemStyle = list(borderColor = "black", borderWidth = "1"),
    barWidth = "60%"
  ) %>% 
  e_timeline_opts(autoPlay = FALSE, playInterval = 1500, top = "55", loop = FALSE) %>%
  e_grid(top = 110, bottom = 50, left = 130) %>%
  e_legend(show = FALSE) %>% 
  e_flip_coords() %>% 
  e_y_axis(inverse = TRUE, axisLabel = list (fontWeight = "bold", fontSize = 13)) %>% 
  e_title(paste0("Life Expectancy in European Countries (1900–2015)"), 
          subtext = "Source: ourworldindata.org", 
          sublink = "https://ourworldindata.org/life-expectancy", 
          left = "center", top = 0) %>% 
  e_labels(
    position = "insideRight",
    formatter = "{@[0]}",
    fontWeight = "bold"
  ) %>% 
  e_color(color = "royalblue")
plt_europe
saveWidget(plt_europe, file = "widgets/03_life_europe.html")
```
<br/>

Data source: [ourworldindata.org](https://ourworldindata.org/)  
Article: [ourworldindata.org](https://ourworldindata.org/life-expectancy)
