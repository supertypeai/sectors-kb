---
title: "Stocks with Highest Market Capitalization since December 2020 on Indonesia Stock Exchange"
author: "Gerald Bryan"
author_link: "https://github.com/geraldbryan"
date: "2024-04-25"
language: "R"
---

# Animating the Top Stocks by Market Capitalization on IDX

Market capitalization, often referred to as market cap, is a measure of a company's total value in the stock market. It is calculated by multiplying the current share price by the total number of outstanding shares. In a way, it is the market's consensus on the value of a company, reflecting both the company's size and its perceived growth prospects. In Indonesia, it is well observed that the top companies by market capitalization (and hence, value) are the largest banks that command a household level of trust and brand recognition. 

# Analysis

Given the significance of market capitalization in stock assessment, we will examine the evolving rankings of stocks based on their market capitalization values since December 2020 on the Indonesia Stock Exchange. We can directly fetch the historical market capitalization using the [Sector's API](https://sectors.app/api). To access the API you only need to go to the [Sector's website](https://sectors.app), subscribe to the plan and call the API directly from your script or notebook.

Below is the code how to fetch the historical market capitalization data using the [Sector's API](https://sectors.app/api).

```r
```{r}
# Load necessary libraries
library(httr)
library(jsonlite)
library(dplyr)

stocks_list <- c("ICBP.JK", "BBNI.JK", "TPIA.JK", "HMSP.JK", "ASII.JK", "UNVR.JK", "BMRI.JK", "TLKM.JK", "BBRI.JK","BBCA.JK", "EMTK.JK", "BRIS.JK", "ARTO.JK", "DCII.JK", "BYAN.JK", "GOTO.JK", "ADRO.JK", "AMMN.JK","BREN.JK")

# Initialize an empty DataFrame
df_daily_hist <- data.frame()

# Generate the date list
date <- get_date_list("2019-01-01")

# Loop through symbols and date ranges
for (i in stocks_list) {
  for (j in 1:(length(date)-1)) {
    if (j == 1) {
      start_date <- date[[j]][1]
      end_date <- date[[j + 1]][1]
    } else {
      start_date <- date[[j]][1] + 1
      end_date <- date[[j + 1]][1]
    }
    
    # Format the dates as strings
    start_date <- format(start_date, "%Y-%m-%d")
    end_date <- format(end_date, "%Y-%m-%d")
    
    # Construct the URL
    url <- paste0("https://api.sectors.app/v1/daily/", i, "/?start=", start_date, "&end=", end_date)
    
    # Make the API request
    response <- GET(url, add_headers(Authorization = api_key))
    
    # Check if the request was successful
    if (status_code(response) == 200) {
      data <- content(response, "text")
      data <- fromJSON(data, flatten = TRUE)
      df_daily_hist <- rbind(df_daily_hist, as.data.frame(data))
    } else {
      # Handle error
      print(status_code(response))
    }
    
    print(paste("Finish collecting data for stock", i, "from", start_date, "to", end_date))
  }
}

df_daily_hist$month <- month(df_daily_hist$date)
df_daily_hist$year <- year(df_daily_hist$date)

df_daily_hist <- df_daily_hist %>% 
  filter(date>"2020-12-01") %>% 
  arrange(date) %>% 
  group_by(symbol,year,month) %>% 
  summarize(last_market_cap = last(market_cap), .groups = 'drop') 

df_daily_hist <- df_daily_hist %>%
  rename("market_cap"="last_market_cap")
```

Or if you haven't subscribe to the [Sector's API](https://sectors.app/api), I already upload [the dataset](../dataset/historical_marketcap.csv) for this analysis in this repository and it is provided by [Sectors](https://sectors.app), who have already compiled and curated the data for our use in this analysis, simplifying the process for us. The dataset is consist of the monthly market capitalization for each companies since December 2020, and using this data we will create a animated plot to see the top 10 stocks with highest market capitalization on Indonesia Stock Exchange.

Here is the glimpse of the data which you can directly fetch from [Sector's API](https://sectors.app/api) or you can directly access the data from [here](../dataset/historical_marketcap.csv) which is directly from [Sectors](https://sectors.app). But be sure to subscribe to [Sector's API](https://sectors.app/api) for more flexible data retrieval options and the opportunity to expand your analytical data sources.

|  date        | symbol  | market_cap   |
| ------------ | ------- | ------------ |
| 2023-08-31   | ADMR.JK | 5.560000e+13	|
| 2023-08-31   | ADRO.JK | 8.540300e+13 |
| 2023-08-31   | AIMS.JK | 6.300000e+10 |
| 2023-08-31   | AKRA.JK | 2.810300e+13	|
| 2023-08-31   | APEX.JK | 4.390000e+11	|

## Data Wrangling

If you have followed [the top_stock_volume_animation_plot recipe](./01_top_stock_volume_animation_plot.md), the code below is more or less the same like the one in that recipe. The differences are the data source and the data cleansing part, since in this recipe the data manipulation parts won't be that complicated. 

```r
df_daily_hist$month <- month(df_daily_hist$date)
df_daily_hist$year <- year(df_daily_hist$date)
```

First, we will extract the month and year from the date value. This is necessary to group the stocks by month so that we can select the top 10 stocks with the highest market capitalization in each month. After extracting this information, we will perform several data manipulation steps:

1. Take the top 10 stocks based on the market capitalization value in each month.
2. Create a `Rank` variable to annotate the position of each stock each month.
3. Finally, arrange the dataset by date and market capitalization value in ascending order.

All of the three steps that I have explained above will be executed using the code below

```r
df_daily_hist <- df_daily_hist %>% 
  group_by(month,year) %>% 
  top_n(10,market_cap) %>% 
  arrange(year,month, market_cap)

df_daily_hist <- df_daily_hist %>% 
  mutate(rank = rank(-market_cap))
```

To enhance readability, we will format the `market_cap` value using the previously created formatted number function. Additionally, we will include an `image` column containing each company's logo value for visualization purposes. Finally, we will format the date column for improved visualization.

```r
formatNumber <- function(number) {
  absNumber <- abs(number)
  
  if (absNumber > 1e12) {
    return(paste0(format(round(number / 1e12, 2), nsmall = 2), "T"))
  } else if (absNumber > 1e9) {
    return(paste0(format(round(number / 1e9, 2), nsmall = 2), "B"))
  } else if (absNumber > 1e6) {
    return(paste0(format(round(number / 1e6, 2), nsmall = 2), "M"))
  } else if (absNumber > 1e3) {
    return(paste0(format(round(number / 1e3, 2), nsmall = 2), "K"))
  } else {
    return(as.character(number))
  }
}

df_finished <- df_daily_hist %>%
  mutate(market_cap_text = formatNumber(market_cap))

location = "../chart_logo/"

df_finished <- df_finished %>% 
  mutate(image = paste0(location,symbol, ".png"))

df_finished$date_formatted <- paste0(df_finished$year,"-",df_finished$month)
```

Here is a preview of the processed data, prepared and ready for visualization.

|  date        | symbol  | market_cap   | month | year | rank | market_cap_text | image                       |
| ------------ | ------- | ------------ | ----- | ---- | ---- | --------------- | --------------------------- |
| 2020-12-31   | ICBP.JK | 1.116630e+14	| 12    | 2020 | 10   | 111.66T         | "../chart_logo/ICBP.JK.png" |
| 2020-12-31   | BBNI.JK | 1.140040e+14 | 12    | 2020 | 9    | 114.00T         | "../chart_logo/BBNI.JK.png" |
| 2020-12-31   | TPIA.JK | 1.618390e+14 | 12    | 2020 | 8    | 161.84T         | "../chart_logo/TPIA.JK.png" |
| 2020-12-31   | HMSP.JK | 1.750590e+14 | 12    | 2020 | 7    | 175.06T         | "../chart_logo/HMSP.JK.png" |
| 2020-12-31   | ASII.JK | 2.439130e+14 | 12    | 2020 | 6    | 243.91T         | "../chart_logo/ASII.JK.png" |
| 2020-12-31   | UNVR.JK | 2.804030e+14 | 12    | 2020 | 5    | 280.40T         | "../chart_logo/UNVR.JK.png" |
| 2020-12-31   | BMRI.JK | 2.922150e+14 | 12    | 2020 | 4    | 292.21T         | "../chart_logo/BMRI.JK.png" |
| 2020-12-31   | TLKM.JK | 3.278960e+14 | 12    | 2020 | 3    | 327.90T         | "../chart_logo/TLKM.JK.png" |
| 2020-12-31   | BBRI.JK | 5.092090e+14 | 12    | 2020 | 2    | 509.21T         | "../chart_logo/BBRI.JK.png" |
| 2020-12-31   | BBCA.JK | 8.262260e+14 | 12    | 2020 | 1    | 826.23T         | "../chart_logo/BBCA.JK.png" |


## Data Visualization

```r
custom_theme <- theme(
  axis.line=element_blank(),
  axis.text.x=element_blank(),
  axis.text.y=element_blank(),
  axis.ticks=element_blank(),
  axis.title.x=element_blank(),
  axis.title.y=element_blank(),
  legend.position="none",
  panel.background=element_rect(fill="black"), # Black background
  panel.border=element_blank(),
  panel.grid.major=element_blank(),
  panel.grid.minor=element_blank(),
  panel.grid.major.x = element_line( size=.1, color="grey" ),
  panel.grid.minor.x = element_line( size=.1, color="grey" ),
  plot.title=element_text(size=25, hjust=0.5, face="bold", colour="grey", vjust=-1.5, margin=margin(t=2, unit="line")),
  plot.subtitle=element_text(size=18, hjust=0.5, vjust=0,face="italic", color="grey"),
  plot.caption =element_text(size=12, hjust=0.5, face="italic", color="grey"),
  plot.margin = margin(2,2, 2, 4, "cm"),
  plot.background=element_rect(fill="black"),
)
```

The code above is used to define a `theme` for the plot, which does not affect the underlying data. This code is solely for configuring the visual appearance of the chart. While we can start to create the animated plot by creating the static plot using the code below

```r
colors <- c("#f87171", "#fb923c","#fbbf24", "#facc15",
            "#a3e635","#4ade80","#34d399","#2dd4bf",
            "#22d3ee","#38bdf8","#60a5fa","#818cf8",
            "#a78bfa","#c084fc","#e879f9","#f472b6",
            "#fb7185","#b91c1c","#b45309")


staticplot <- ggplot(df_finished, aes(rank,market_cap/2, group = symbol)) +
  geom_tile(aes(y = market_cap/2,
                height = market_cap,fill = as.factor(symbol),
                width = 0.9), alpha = 0.8, color = NA) +
  scale_fill_manual(values = colors)+
  geom_text(aes(y = 0, label = paste(symbol, " "), color="white"), vjust = 0.2, hjust = 1,size=7) +
  geom_text(aes(y=market_cap,label = paste0(" ",market_cap_text),color="white", hjust=0),size=4) +
  coord_flip(clip = "off", expand = FALSE) +
  geom_image(aes(image=image),size=0.05, asp=1.5) +
  scale_y_continuous(labels = scales::comma) +
  scale_x_reverse() +
  guides(color = FALSE, fill = FALSE) +
  custom_theme
```

Now that we've created the static plot, we can begin to make the animation and save it as .gif file

```r
anim <- staticplot + transition_states(date_formatted, transition_length = 4, state_length = 1) +
  view_follow(fixed_x = TRUE)  +
  labs(title = 'Market Capitalization: {closest_state}',
       subtitle  =  "Stocks with the Highest Market Capitalization on Indonesia Stock Exchange since December 2020")
       #caption  = "Source: https://sectors.app")

animate(anim, 300, fps = 8, width = 1200, height = 1000,
        renderer = gifski_renderer("top_market_cap.gif"))
```

and here is the result of the gif

![Stock with Highest Market Capitalization](../image/top_market_cap.gif)

Looking at the plot, it's clear that `BBCA.JK` consistently maintains the highest market capitalization on the Indonesia Stock Exchange since December 2020, while other positions fluctuate. For deeper insights into the market capitalization of companies or sectors on the Indonesia Stock Exchange, check out [Sectors](https://sectors.app). Also, stay tune to our next recipes!
