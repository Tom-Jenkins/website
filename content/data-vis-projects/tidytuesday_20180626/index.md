---

title: "Alcohol Consumption"
summary: "TidyTuesday Week 13 2018 data visualisations."
author: "Tom Jenkins"
date: "2022-08-16"
# Header image (featured.png)
# To use, add an image named `featured.jpg/png` to your page's folder. 
tags: ["R", "TidyTuesday", "Leaflet"]
weight: 2
featured: true
image:
  preview_only: true

---

<br/>

Data visualisation for [TidyTuesday](https://github.com/rfordatascience/tidytuesday) Week 13 2018. The data set is available on the TidyTuesday GitHub [page](https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-06-26) and the R code used to create the interactive map is available at the end of this page.

<br/>

<iframe height="650px" width="100%" frameborder="no" 
src="02_choropleth.html"></iframe>

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
# Leaflet Data Visualisation ####
#
# Data from Tidy Tuesday:
# https://github.com/rfordatascience/tidytuesday
# Week 13 2018
# Alcohol Global
#
# -------------------------- #

# Load packages
library(tidyverse)
library(tidyselect)
library(htmlwidgets)
library(echarts4r)
library(echarts4r.assets)
library(countrycode)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
library(rnaturalearthhires)
library(leaflet)
library(leaflet.extras)
library(htmltools)

# Import data
# https://github.com/rfordatascience/tidytuesday/tree/master/data/2018/2018-06-26
alcohol = read_csv("tidytuesday_data/2018-06-26-alcohol_global.csv")

# Add data for continent and un.regionsub.name using countrycode package
alcohol = alcohol %>%
  mutate(continent = countrycode(alcohol$country, "country.name", "continent")) %>%
  mutate(region_un = countrycode(alcohol$country, "country.name", "un.regionsub.name"))
alcohol

# Print number of countries per category
table(alcohol$continent)
table(alcohol$region_un)

# Import country polygons from rnaturalworld package
country_poly = ne_countries(scale = "large", returnclass = "sf")


# ----------------- #
#
# Prepare Data ####
#
# ----------------- #

# Extract alcohol data for Europe
europe_alcohol = alcohol %>%
  filter(continent == "Europe") %>%
  select(country, contains("servings"), contains("litres")) %>%
  mutate(country = str_replace(country, "Bosnia-Herzegovina", "Bosnia and Herzegovina"))

# Extract polygons for Europe and join with alcohol data
europe_sf = country_poly %>%
  filter(continent == "Europe") %>%
  select(country = name_long) %>%
  left_join(europe_alcohol, by = "country") %>%
  drop_na %>%
  st_transform(crs = 4326)


# ----------------- #
#
# Leaflet Map ####
#
# ----------------- #

# Polygon highlighting options
poly_highlighting = highlightOptions(
  weight = 3,
  color = "white",
  fillOpacity = 1,
  bringToFront = TRUE
)

# Tooltip (label) options
tooltip_options = labelOptions(
  direction = "auto",
  textsize = "15px",
  opacity = 0.9,
  style = list(
    "font-family" = "Arial",
    "padding" = "6px",
    "border-color" = "black"
  )
)

# Colour bins
bins_servings = c(0, 50, 100, 150, 200, 250, 300, 400)
bins_total = c(0, 3, 6, 9, 12, 15)

# Define colour palettes and bins
pal_beer = colorBin("YlOrRd", domain = europe_sf$beer_servings, bins = bins_servings)
pal_spirit = colorBin("YlOrRd", domain = europe_sf$spirit_servings, bins = bins_servings)
pal_wine = colorBin("YlOrRd", domain = europe_sf$wine_servings, bins = bins_servings)
pal_total = colorBin("viridis", domain = europe_sf$total_litres_of_pure_alcohol, bins = bins_total, reverse = FALSE)

# Function to add variable legends to map
add_legend = function(map, var_id, var_pal, ...){
  addLegend(
    map = map,
    data = europe_sf,
    opacity = 0.7,
    position = "bottomright",
    layerId = var_id,
    title = var_id,
    pal = var_pal,
    # Required in order to select the correct variable via var_id
    values = select(europe_sf, starts_with(str_sub(var_id, 1, 4))),
    ...
  )
}

# Function to add variable choropleths to map
add_choropleth = function(map, var_id, var_pal, ...){
  # Variable ID
  var_colname = vars_select(colnames(europe_sf), starts_with(str_sub(var_id, 1, 4)))
  # Tooltip (label)
  tooltip = sprintf(
    "<b>%s</b><br/>%g %s",
    europe_sf$country,
    europe_sf[[var_colname]],
    ifelse(var_colname == "total_litres_of_pure_alcohol", "Litres", "Servings")
  ) %>% lapply(htmltools::HTML)
  # Polygons
  addPolygons(
    map = map,
    data = europe_sf,
    fillOpacity = 1,
    color = "black",
    weight = 1,
    opacity = 0.4,
    highlightOptions = poly_highlighting,
    label = tooltip,
    labelOptions = tooltip_options,
    group = var_id,
    # Required in order to select the correct variable via var_id
    fillColor = ~var_pal(europe_sf[[var_colname]]),
    ...
  )
}

# Variable IDs
beer_id = "Beer"
spirit_id = "Spirits"
wine_id = "Wine"
total_id = "Total Consumed (Litres)"

# Plot map
l1 = leaflet() %>%
  # Set view and zoom level
  setView(lng = 15, lat = 55.0, zoom = 4) %>%
  # Reset map to default setting
  addResetMapButton() %>% 
  # Add a scalebar
  addScaleBar(
    position = "bottomright",
    options = scaleBarOptions(imperial = FALSE)
  ) %>%
  # Choropleth polygons
  add_choropleth(var_id = beer_id, var_pal = pal_beer) %>% 
  add_choropleth(var_id = spirit_id, var_pal = pal_spirit) %>% 
  add_choropleth(var_id = wine_id, var_pal = pal_wine) %>%
  add_choropleth(var_id = total_id, var_pal = pal_total) %>% 
  # Legends
  add_legend(var_id = beer_id, var_pal = pal_beer) %>% 
  add_legend(var_id = spirit_id, var_pal = pal_spirit) %>%
  add_legend(var_id = wine_id, var_pal = pal_wine) %>% 
  add_legend(var_id = total_id, var_pal = pal_total) %>% 
  # Add layers control
  addLayersControl(
    options = layersControlOptions(collapsed = FALSE),
    baseGroups = c(beer_id, spirit_id, wine_id, total_id)
    ) %>% 
  # Base group title
  htmlwidgets::onRender(
    jsCode = "function() { $('.leaflet-control-layers-base').prepend('<label style=\"text-align:left\"><strong><font size=\"4\">Alcohol Consumption</font></strong></label>');}"
    ) %>% 
  # Switch legends when a different base group is selected
  # Code from here: https://gist.github.com/noamross/98c2053d81085517e686407096ec0a69
  htmlwidgets::onRender("
    function(el, x) {
      var initialLegend = 'Beer' // Set the initial legend to be displayed by layerId
      var myMap = this;
      for (var legend in myMap.controls._controlsById) {
        var el = myMap.controls.get(legend.toString())._container;
        if(legend.toString() === initialLegend) {
          el.style.display = 'block';
        } else {
          el.style.display = 'none';
        };
      };
    myMap.on('baselayerchange',
      function (layer) {
        for (var legend in myMap.controls._controlsById) {
          var el = myMap.controls.get(legend.toString())._container;
          if(legend.toString() === layer.name) {
            el.style.display = 'block';
          } else {
            el.style.display = 'none';
          };
        };
      });
    }")
# l1
saveWidget(l1, file = "widgets/02_choropleth.html")
```
<br/>

Data source: [FiveThirtyEight package](https://github.com/rudeboybert/fivethirtyeight)  
Article: [FiveThirtyEight.com](https://fivethirtyeight.com/features/dear-mona-followup-where-do-people-drink-the-most-beer-wine-and-spirits/)
