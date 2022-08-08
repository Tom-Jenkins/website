---

title: "Comic-Book Characters"
summary: "TidyTuesday Week 9 2018 data visualisations."
author: "Tom Jenkins"
date: "2022-08-05"
# Header image (featured.png)
# To use, add an image named `featured.jpg/png` to your page's folder. 
tags: ["R", "TidyTuesday", "ECharts"]
weight: 2
featured: true
image:
  preview_only: true

---

<br/>

Data visualisations for [TidyTuesday](https://github.com/rfordatascience/tidytuesday) Week 9 2018. The data are available on the TidyTuesday GitHub [page](https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-05-29) and the R code used to create the interactive graphics is available at the end of this page.

<br/>

<iframe height="500px" width="100%" frameborder="no"
src="01_characters_created.html"></iframe>

<iframe height="500px" width="50%" frameborder="no"
src="01_sunburst_marvel.html"></iframe>

<iframe height="500px" width="50%" align="left" frameborder="no"
src="01_sunburst_dc.html"></iframe>

<iframe height="1000px" width="100%" frameborder="no"
src="01_line_chart.html"></iframe>


<style>
pre {
  max-height: 500px;
  overflow-y: auto;
}
</style>

```
# -------------------------- # 
#
# ECharts Data Visualisations ####
#
# Data from TidyTuesday:
# https://github.com/rfordatascience/tidytuesday
# Week 9 2018
# Comic-Book Characters
#
# -------------------------- # 

# Load packages
library(tidyverse)
library(htmlwidgets)
library(echarts4r)
library(echarts4r.assets)
library(lubridate)

# Import data
# https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-05-29
comics = read_csv("tidytuesday_data/2018-05-29-comic_characters.csv")

# Change labels
comics$sex = str_remove_all(comics$sex, " Characters")
comics$align = str_replace_all(comics$align, "Good Characters", "Hero/Heroine")
comics$align = str_replace_all(comics$align, "Bad Characters", "Villain")
comics$alive = str_remove_all(comics$alive, " Characters")
comics$alive = str_replace_all(comics$alive, "Living", "Alive")

# Update Male to Hero and Female to Heroine
comics = comics %>% 
  mutate(align = case_when(sex == "Male" & align == "Hero/Heroine" ~ "Hero", TRUE ~ align)) %>% 
  mutate(align = case_when(sex == "Female" & align == "Hero/Heroine" ~ "Heroine", TRUE ~ align))

# Colour themes
marvel_col = "#F0141E"
dc_col = "#0178F0"


# ----------------- #
#
# Grouped Bar Chart ####
#
# ----------------- #

# Number of new characters created per year
characters_pub = comics %>% 
  group_by(publisher) %>% 
  count(year) %>% 
  mutate(year = as.character(year)) %>% 
  drop_na

# Plot bar
plt1 = characters_pub %>% 
  group_by(publisher) %>% 
  e_charts(x = year, renderer = "svg") %>% 
  e_bar(serie = n) %>% 
  e_axis_labels(x = "Year") %>% 
  e_title(
    text = "Comic-Book Characters",
    subtext = "Number of characters created"
  ) %>% 
  e_tooltip(trigger = "axis") %>% 
  e_color(color = c(dc_col, marvel_col)) %>% 
  e_toolbox_feature("dataZoom", title = list(zoom = "Zoom", back = "Reset")) %>% 
  e_toolbox_feature("saveAsImage", title = ".svg", name = "image")
plt1
saveWidget(plt1, file = "01_characters_created.html")


# ----------------- #
#
# Drilldown Sunburst ####
#
# ----------------- #

# Subset data
comics_sun = comics %>% 
  select(publisher, sex, align, alive) %>% 
  filter(sex %in% c("Male","Female")) %>%
  filter(align != "Reformed Criminals") %>% 
  drop_na

# Function to transform data frame to hierarchical nested tibble
df_to_hier = function(df, pub){
  
  # Filter publisher
  df = filter(df, publisher == pub)
  
  # First level
  first_level = df %>%
    count(sex)
  
  # Second level
  second_level = df %>% 
    group_by(sex) %>% 
    count(align)
  
  # Third level
  third_level = df %>%
    group_by(sex, align) %>%
    count(alive)
  
  # Create hierarchical nested tibble from data
  df_nest = tibble(
    # 1st level
    name = first_level$sex,
    value = first_level$n,
    # itemStyle = tibble(color = c("red","blue")),
    # 2nd level
    children = list(
      # Female
      tibble(
        name = filter(second_level, sex == "Female")$align,
        value = filter(second_level, sex == "Female")$n,
        # 3rd level
        children = list(
          tibble(
            name = filter(third_level, sex == "Female", align == "Heroine")$alive,
            value = filter(third_level, sex == "Female", align == "Heroine")$n
          ),
          tibble(
            name = filter(third_level, sex == "Female", align == "Villain")$alive,
            value = filter(third_level, sex == "Female", align == "Villain")$n
          )
        )
      ),
      # Male
      tibble(
        name = filter(second_level, sex == "Male")$align,
        value = filter(second_level, sex == "Male")$n,
        # 3rd level
        children = list(
          tibble(
            name = filter(third_level, sex == "Male", align == "Hero")$alive,
            value = filter(third_level, sex == "Male", align == "Hero")$n
          ),
          tibble(
            name = filter(third_level, sex == "Male", align == "Villain")$alive,
            value = filter(third_level, sex == "Male", align == "Villain")$n
          )
        )
      )
    )
  )
  return(df_nest)
}

# Create nested tibble for DC and Marvel
dc_nest = df_to_hier(comics_sun, "DC")
mv_nest = df_to_hier(comics_sun, "Marvel")

# Plot DC sunburst
dc_sun = dc_nest %>% 
  e_charts(renderer="svg") %>% 
  e_sunburst() %>% 
  e_title(
    text = "DC",
    subtext = "Proportion of characters per category"
  ) %>%
  e_theme("westeros") %>% 
  e_tooltip() %>% 
  e_toolbox_feature("restore", title = "Reset") %>% 
  e_toolbox_feature("saveAsImage", title = ".svg", name = "image")

# Plot Marvel sunburst
mv_sun = mv_nest %>% 
  e_charts(renderer="svg") %>% 
  e_sunburst() %>% 
  e_title(
    text = "Marvel",
    subtext = "Proportion of characters per category"
  ) %>%
  e_theme("westeros") %>% 
  e_tooltip() %>% 
  e_toolbox_feature("restore", title = "Reset") %>% 
  e_toolbox_feature("saveAsImage", title = ".svg", name = "image")

# Arrange both plots on same grid
e_arrange(dc_sun, mv_sun, cols = 2)

# Export as html widgets
saveWidget(mv_sun, file = "01_sunburst_marvel.html")
saveWidget(dc_sun, file = "01_sunburst_dc.html")


# ----------------- #
#
# Line Chart ####
#
# ----------------- #

# Total number of appearances per every ten years per align
comics_line = comics %>% 
  filter(sex %in% c("Male","Female")) %>%
  filter(align != "Reformed Criminals") %>% 
  select(publisher, align, appearances, date) %>%
  drop_na %>% 
  group_by(publisher, align, date) %>% 
  summarise(appearances = sum(appearances)) %>% 
  mutate(year_group = as.character(year(floor_date(date, "10 years")))) %>%
  mutate(year_group = str_c(year_group, "s")) %>% 
  group_by(publisher, align, year_group) %>% 
  summarise(appearances_yr = sum(appearances)) %>% 
  arrange(year_group)
comics_line

# Function to connect line charts
create_chart = function(data, pub){
  data %>% 
    filter(publisher == pub) %>% 
    group_by(align) %>%
    e_charts(x = year_group, renderer = "svg", height = "400px") %>% 
    e_line(serie = appearances_yr) %>% 
    e_title(
      text = pub,
      subtext = "Total number of appearances"
    ) %>% 
    e_axis_labels(x = "Year") %>% 
    e_theme("westeros") %>% 
    e_tooltip() %>% 
    e_toolbox_feature("restore", title = "Switch to Line Chart", icon = "M18.737,9.691h-5.462c-0.279,0-0.527,0.174-0.619,0.437l-1.444,4.104L8.984,3.195c-0.059-0.29-0.307-0.506-0.603-0.523C8.09,2.657,7.814,2.838,7.721,3.12L5.568,9.668H1.244c-0.36,0-0.655,0.291-0.655,0.655c0,0.36,0.294,0.655,0.655,0.655h4.8c0.281,0,0.532-0.182,0.621-0.45l1.526-4.645l2.207,10.938c0.059,0.289,0.304,0.502,0.595,0.524c0.016,0,0.031,0,0.046,0c0.276,0,0.524-0.174,0.619-0.437L13.738,11h4.999c0.363,0,0.655-0.294,0.655-0.655C19.392,9.982,19.1,9.691,18.737,9.691z") %>%
    e_toolbox_feature("magicType", type = list("bar"), icon = list(bar = "M11.709,7.438H8.292c-0.234,0-0.427,0.192-0.427,0.427v8.542c0,0.234,0.192,0.427,0.427,0.427h3.417c0.233,0,0.426-0.192,0.426-0.427V7.865C12.135,7.63,11.942,7.438,11.709,7.438 M11.282,15.979H8.719V8.292h2.563V15.979zM6.156,11.709H2.74c-0.235,0-0.427,0.191-0.427,0.426v4.271c0,0.234,0.192,0.427,0.427,0.427h3.417c0.235,0,0.427-0.192,0.427-0.427v-4.271C6.583,11.9,6.391,11.709,6.156,11.709 M5.729,15.979H3.167v-3.416h2.562V15.979zM17.261,3.167h-3.417c-0.235,0-0.427,0.192-0.427,0.427v12.812c0,0.234,0.191,0.427,0.427,0.427h3.417c0.234,0,0.427-0.192,0.427-0.427V3.594C17.688,3.359,17.495,3.167,17.261,3.167 M16.833,15.979h-2.562V4.021h2.562V15.979z")) %>% 
    e_toolbox_feature("dataZoom", title = list(zoom = "Zoom", back = "Reset")) %>% 
    e_toolbox_feature("saveAsImage", title = ".svg", name = "image") %>%
    e_group("comics_line")
}

# Plot line charts
mv_line = create_chart(comics_line, "Marvel") %>% e_connect_group("comics_line")
dc_line = create_chart(comics_line, "DC") %>% e_connect_group("comics_line")

# Arrange both plots on same grid
e_arrange(mv_line, dc_line, rows = 2)
```
<br/>


