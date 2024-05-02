---
title: "Most Traded Stocks on Indonesia Stock Exchange in 2024"
author: "Gerald Bryan"
author_link: "https://github.com/geraldbryan"
date: "2024-04-23"
language: "R"
---


Have you ever thought about creating an animated plot that evolves over time, perfect for sharing on Instagram Reels or other content platforms? Well, now you can achieve that using R and data from the [Sectors](https://sectors.app). In this recipe, I'll guide you to create the animated plot of the most traded stocks on Indonesia Stock Exchange in 2024.

## Install Libraries

Before we dive into the project, let's set up our environment by installing essential packages. We'll use the 'tidyverse' package for data manipulation in R, along with 'ggplot' and 'gganimate' libraries to craft our visualization and animated plot.

```r
install.packages(c("tidyverse","gganimate"))
```

## Load Library
```r
library(tidyverse)
library(gganimate)
```

## Data Source and Data Manipulation

After installing the necessary packages and loading the libraries, we can start manipulating the data for our analysis. [The dataset](../dataset/most_traded_stocks.csv) for this task consists of daily transaction data from all stocks on the Indonesia Stock Exchange since 2024, and it's accessible on [Sectors](https://sectors.app). Our focus will be on the daily transaction volume of each stock. Here's a glimpse of the dataset we'll be working with.

|  date        | symbol  | volume   |
| ------------ | ------- | -------- |
| 2024-02-20   | ACRO.JK | 10730000 |
| 2024-02-20   | ASLI.JK | 658400   |
| 2024-02-20   | MANG.JK | 2931100  |
| 2024-02-20   | NICE.JK | 94226900 |
| 2024-02-20   | MSJA.JK | 50862300	|

We will start by calculating the cumulative sum for each stock from the beginning of 2024 until now. Additionally, we will select only the top 10 stocks with the highest cumulative sum to include in the plot. This can be accomplished using the functions available in the tidyverse package in R.

```r
df_filter <- df %>% 
  group_by(symbol) %>%
  arrange(date) %>%
  mutate(accumulated_volume = cumsum(volume))

df_finished <- df_filter %>% 
  group_by(date) %>% 
  top_n(10,accumulated_volume) %>% 
  arrange(date,accumulated_volume)

df_finished <- df_finished %>% 
  group_by(date) %>% 
  mutate(rank = rank(-accumulated_volume))
```

With the provided code, the data will be transformed as shown in the table below. This table will include the ranking and cumulative sum up until the most recent date, which is essential for creating the plot.

|  date        | symbol  | volume     | accumulated_volume | rank |
| ------------ | ------- | ---------- | ------------------ | ---- |
| 2024-01-02   | MPXL.JK | 213109100  | 213109100          | 10   |
| 2024-01-02   | BAPA.JK | 282855200  | 282855200          | 9    |
| 2024-01-02   | BIPI.JK | 300042500  | 300042500          | 8    |
| 2024-01-02   | STRK.JK | 328691100  | 328691100          | 7    |
| 2024-01-02   | NATO.JK | 454095300  | 454095300          | 6    |
| 2024-01-02   | DOOH.JK | 489452400  | 489452400          | 5    |
| 2024-01-02   | DEWA.JK | 509467700  | 509467700          | 4    |
| 2024-01-02   | CARE.JK | 673579200  | 673579200          | 3    |
| 2024-01-02   | BUMI.JK | 998119800  | 998119800          | 2    |
| 2024-01-02   | NATO.JK | 1033123000 | 1033123000         | 1    |

To finalize the data preparation for the plot, we will format the accumulated volume to enhance readability. This formatting will make the numbers easier for people to interpret.

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

# Apply the formatNumber function using mutate
df_finished <- df_finished %>%
  mutate(accumulated_volume_text = formatNumber(accumulated_volume))
```

|  date        | symbol  | volume     | accumulated_volume | rank | accumulated_volume_text |
| ------------ | ------- | ---------- | ------------------ | ---- | ----------------------- |
| 2024-01-02   | MPXL.JK | 213109100  | 213109100          | 10   | 213.11M                 |
| 2024-01-02   | BAPA.JK | 282855200  | 282855200          | 9    | 282.86M                 |
| 2024-01-02   | BIPI.JK | 300042500  | 300042500          | 8    | 300.04M                 |
| 2024-01-02   | STRK.JK | 328691100  | 328691100          | 7    | 328.69M                 |
| 2024-01-02   | NATO.JK | 454095300  | 454095300          | 6    | 454.10M                 |
| 2024-01-02   | DOOH.JK | 489452400  | 489452400          | 5    | 489.45M                 |
| 2024-01-02   | DEWA.JK | 509467700  | 509467700          | 4    | 509.47M                 |
| 2024-01-02   | CARE.JK | 673579200  | 673579200          | 3    | 673.58M                 |
| 2024-01-02   | BUMI.JK | 998119800  | 998119800          | 2    | 998.12M                 |
| 2024-01-02   | NATO.JK | 1033123000 | 1033123000         | 1    | 1033.12M                |


## Data Visualization

Let's begin by designing the theme for our plot! You don't necessarily have to create a custom theme from scratch; there are pre-built themes available for use in ggplot that are free. However, in this instance, I've created a custom theme to ensure that the final content aligns with other `Sectors` content.

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

After creating the theme, we can directly plot the data into static plot first before we animate it into an animated plot.

```r
# Specify color for each stocks (company)
colors <- c("#f87171", "#fb923c","#fbbf24", "#facc15",
            "#a3e635","#4ade80","#34d399","#2dd4bf",
            "#22d3ee","#38bdf8","#60a5fa","#818cf8",
            "#a78bfa","#c084fc","#e879f9","#f472b6",
            "#fb7185","#b91c1c","#b45309","#4d7c0f",
            "#15803d","#0f766e","#0e7490","#4338ca",
            "#fde68a","#bbf7d0","#bae6fd","#fbcfe8")


staticplot <- ggplot(df_finished, aes(rank,accumulated_volume/2, group = symbol)) +
  # Create the bar chart
  geom_tile(aes(y = accumulated_volume/2,
                height = accumulated_volume,fill = as.factor(symbol),
                width = 0.9), alpha = 0.8, color = NA) +
  #Set the color of each bar according to the 'color' variable
  scale_fill_manual(values = colors)+
  #Set the xticker label
  geom_text(aes(y = 0, label = paste(symbol, " "), color="white"), vjust = 0.2, hjust = 1,size=7) +
  #Set the bar chart (accumulated volume label)
  geom_text(aes(y=accumulated_volume,label = paste0(" ",accumulated_volume_text),color="white", hjust=0),size=4) + 
  coord_flip(clip = "off", expand = FALSE) +
  #Add the logo of the company in each bar
  geom_image(aes(image=image),size=0.05, asp=1.5) + 
  scale_y_continuous(labels = scales::comma) +
  scale_x_reverse() +
  guides(color = FALSE, fill = FALSE) +
  custom_theme
```

After finishing create the static plot, we can directly convert it into a gif by using this code

```r
anim <- staticplot + transition_states(date, transition_length = 4, state_length = 1) +
  view_follow(fixed_x = TRUE)  +
  labs(title = 'Accumulative Transaction Volume: {closest_state}',
       subtitle  =  "Most traded stocks on Indonesia Stock Exchange since 2024",
       caption  = "Source: https://sectors.app")
```

and here is the result of the gif

![Most Traded Stocks in 2024](../image/most_traded_stocks.gif)

In our latest experiment, we dove into the world of animated plots using data from [Sectors](https://sectors.app) with R. Stay tuned for more exciting recipes featuring Sectors data!