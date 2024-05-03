---
title: "Stocks with Highest Market Capitalization since December 2020 on Indonesia Stock Exchange"
author: "Gerald Bryan"
author_link: "https://github.com/geraldbryan"
date: "2024-05-02"
language: "R"
---

Hello and welcome back to our [Sector's](https://sectors.app) animation plot series! If you haven't already explored our previous recipes and recreated those captivating plots, you're in for a treat. This series is a showcase of creativity, using the dynamic duo of `ggplot` and `gganimate` to craft stunning animated visualizations. These plots aren't just eye candy; they're the content that keeps [Sector's Instagram](https://www.instagram.com/sectorsapp/) account buzzing (make sure to check them out and hit that follow button!). Each plot is based on meticulously curated data from the [Sector's](https://sectors.app) team, providing insightful analysis, particularly focusing on the Indonesia Stock Exchange. So, without further ado, let's dive into the excitement of our third recipe in this animation plot series!

# Financial Ratio Valuation Metrics

In the last two recipes, we already used the market capitalization and transaction volume to do some analysis. In this recipe, we will use financial ratio to do some valuation on each stock price. There are two financial ratio valuation metrics that we will use here which are `Price to Earning Ratio` (PE Value) and `Price to Book Valur Ratio` (PBV).

## PE Value Ratio

The price-to-earnings (P/E) ratio is a metric that evaluates a company's stock price in relation to its earnings per share (EPS). This ratio is crucial for determining the value of a company's stock, as it allows for comparisons of a company's valuation over time, with other companies in the same industry, or with the broader market.

> Example:
> A P/E ratio of 15 means that the company’s current market value equals 15 times its annual earnings.

The P/E ratio serves as a valuable tool for investors to gauge whether a stock is undervalued or overvalues. A high P/E ratio may indicate that the stock is overvalued, suggesting either optimistic growth expectations or inflated market sentiment and vice versa. Additionally, by calculating the median P/E ratio across several years, investors can establish a standardized benchmark to determine a stock's potential worth.

However, a notable limitation of using P/E ratios arises when comparing companies across different sectors. Variations in valuation and growth rates are common among industries due to differences in revenue generation timing and business models, making direct P/E ratio comparisons less reliable.

## PBV Ratio

The price-to-book ratio (P/B ratio) of a company is calculated by dividing its current stock price per share by its book value per share (BVPS). This ratio provides insight into how the market values the company relative to its book value.

> Example A P/B ratio of 1 means that the stock price is trading in line with the book value of the company. In other words, the stock price would be considered fairly valued, strictly from a P/B standpoint.

Typically, a P/B ratio below one suggests that a company is undervalued, whereas a ratio above one suggests that the stock is trading at a premium. However, the significance of these values can vary significantly across industries. Different sectors have their own norms, where lower or higher P/B ratios may be considered typical.

However, the P/B ratio has its limitations. For instance, it does not take into account intangible assets such as intellectual property and brand value, which can be significant in certain industries. As a result, the P/B ratio may not always accurately reflect a company's true value. Furthermore, the P/B ratio may be less relevant for companies with a high proportion of intangible assets, such as technology firms, where the value lies more in ideas and innovations than in physical assets. 

# Plot Creation

After reading about the financial ratios valuation metrics, now we can start to play with the data and create a animated plot to shown the comparisons of the financial valuation metrics (PE and PBV) of some major banks in Indonesia.

## Glimpse of the data

Same like the other two recipes, the [dataset](../idx_company_report.csv) that is used here is provided by [Sectors](https://sectors.app), and here is the data that we will handle today:

| sub_sector   | symbol  | historical_valuation                                                                  | market_cap   |
| ------------ | ------- | ------------------------------------------------------------------------------------- | ------------ |
| Banks        | ARTO.JK | [{"pb":37.5020388158071,"pe":-243.792432227128,"ps":694.972856734688,"year":2020, ... | 5.560000e+13 |
| Banks        | BBCA.JK | [{"pb":4.475852894277,"pe":30.4530861602451,"ps":11.3386964329326,"year":2020, ...    | 8.540300e+13 |
| Banks        | BBNI.JK | [{"pb":1.03458928065172,"pe":34.7530471103703,"ps":2.09814065618114,"year":2020, ...  | 6.300000e+10 |
| Banks        | BBRI.JK | [{"pb":2.24404134373024,"pe":27.296475059198,"ps":3.90812024342799,"year":2020, ...   | 2.810300e+13 |
| Banks        | BMRI.JK | [{"pb":1.54302128039226,"pe":17.3942521554938,"ps":3.05224667300517,"year":2020, ...  | 4.390000e+11 |

## Data Manipulation

```r
# Take the top 10 companies in bank sub-sectors with highest market_cap
df <- df %>% 
  select(c("symbol","sub_sector","historical_valuation",'market_cap')) %>% 
  filter(sub_sector == "Banks") %>% 
  top_n(10,market_cap)
```
First, we will select `Banks` sub-sector as the one that we want to do the analysis because Banks is one of the most popular sectors in Indonesia stock market. We will select top companies with the highest market capitalization in Banks industry, since there are total of 47 companies and we only want to compares the bigger one.

```r
# Create function to expand JSON column
expand_json_column <- function(json_str) {
  # Parse JSON string
  json_data <- fromJSON(json_str)
  # Convert to data frame
  data_frame <- as.data.frame(json_data)
  return(data_frame)
}

# Apply function to expand historical valuation JSON column
expanded_data <- lapply(df$historical_valuation, expand_json_column)

# Bind the expanded data frames
result <- do.call(rbind, expanded_data)

# Add symbol (ticker) columns to the data frame by duplicating thesymbol value from original data frame by 4 each value, since the financial data is from 2020-2023
result$symbol <- rep(df$symbol, each = 4)

df <- merge(result %>% select(c("symbol","year","pe","pb")),df,by="symbol") %>% # Merge the historical valuation data into original data frame
  select(-c(historical_valuation,sub_sector)) %>% 
  mutate_at(vars(pb, pe), ~round(., 2)) %>% # round PBV and PE value by 2 decimal numbers
  filter(symbol %notin% c("ARTO.JK","BNLI.JK")) # remove ARTO.JK and BNLI.JK because these 2 tickers has outliers value that can disturb the plot visualization
```

In the data glimpse, you see that there is one column `historical_valuation` (where the PE ratio and PBV ratio value is stored) that is in the JSON format, so in the code, we try to convert the JSON format column into a table and then merge it with our original data. 

The `historical_valuation` column also has the `Banks` industry PE and PBV value, therefore we will take it and manually merge the data using row binding to our original data.

```r
# Get the subsector pe and pbv value from result dataframe
subsec <- result %>% 
  select(c("year","pb_peer_avg","pe_peer_avg")) %>% 
  distinct(year,pb_peer_avg,pe_peer_avg) %>% 
  mutate("symbol" = "Banks Sub-Sector") %>% 
  rename(pb=pb_peer_avg, pe = pe_peer_avg) %>% 
  select(c(symbol,year,pe,pb))

# merge (row bind) the data for sub-sector financials data into ticker data
df <- rbind(subsec,df %>% select(-market_cap))
```

and here is the glimpse of the final data that we produced using the code above

| symbol             | year | pe       | pb        |
| ------------------ | ---- | -------- | --------- |
| Banks Sub-Sector   | 2020 | 27.29648 | 1.2624248 |
| BBCA.JK            | 2020 | 30.45000 | 4.4800000 |
| BBCA.JK            | 2021 | 28.35000 | 4.3900000 |
| BBNI.JK            | 2022 | 9.30000  | 1.2500000 |
| BBNI.JK            | 2023 | 9.30000  | 1.3000000 |
| BBRI.JK            | 2020 | 27.30000 | 2.2400000 |

## Data Visualization

Same like before, we will create a custom theme for our plot just for the content purpose, you don't need to use this custom theme if you don't want to

```r
# Create custom theme for the plot
custom_theme <- theme(
  axis.line=element_blank(),
  axis.text.x=element_text(color="white",size=10),  # Set x-axis labels to white
  axis.text.y=element_text(color="white",size=10),
  axis.ticks=element_blank(),
  axis.text = element_text(color="white",size=6),
  axis.title.x=element_text(color="white",size=15),
  axis.title.y=element_text(color="white",size=15),
  panel.background=element_rect(fill="black"), # Black background
  panel.border=element_blank(),
  panel.grid.major=element_blank(),
  panel.grid.minor=element_blank(),
  panel.grid.major.x = element_line( size=.1, color="grey" ),
  panel.grid.minor.x = element_line( size=.1, color="grey" ),
  plot.title=element_text(size=25, hjust=0.5, face="bold", colour="white", vjust=-1.5, margin=margin(t=2, unit="line")),
  plot.subtitle=element_text(size=18, hjust=0.5, vjust=0,face="italic", color="grey"),
  plot.caption =element_text(size=12, hjust=0.5, face="italic", color="grey"),
  plot.margin = margin(1,1, 1, 1, "cm"),
  plot.background=element_rect(fill="black"),
  legend.background = element_rect(fill = "black"),
  legend.text = element_text(color='white',size=15),
  legend.title = element_text(color='white',size=18),
  legend.key.size = unit(1.5, "lines"),
  legend.key = element_rect(fill="black")
)
```

Now let's dive to the code that create the main plot or the static plot. In this recipe we will create a scatter plot with respect to PE ratio and PBV ratio as the x and y axis respectively. Also, we will differentiate each company using the color and add the text below each point to make it easier for reader to know which company each point is refer to.

```r
# Create the plot
p <- ggplot(
  df, 
  aes(x = pe, y=pb, colour = symbol)) +
  geom_point(alpha = 0.7,size=12,show.legend = FALSE) +
  geom_text(aes(label = symbol), vjust = 2,size=8)+
  labs(x = "Price to Earning Ratio", y = "Price to Book Value Ratio")+
  ggtitle("Financial Ratio Comparisons of Top 8 Major Companies in Bank Sector") +
  custom_theme +
  theme(legend.position = "none")
p
```

After creating the plot, now we will try to animate that plot, in this animation we will animate the value of each companies PBV and PE ratio value by year. The data we have are PE and PBV ratio value from 2020 to 2023, therefore we will have a plot that animate the PE and PBV value of each companies from 2020 to 2023. Last step, save the animated plot into .gif file.

```r
# Make the plot animation transition
anim <- p + transition_time(year) +
  labs(subtitle = "Year: {frame_time}")

# Save the animation plot to .gif file
animate(anim, 200, fps = 8, width = 1200, height = 1000,
        renderer = gifski_renderer("banks_sub_sector.gif"))
```

and here is the final result of our plot

![Financial Comparisons of Top 8 Major Banks in Indonesia](../image/banks_sub_sector.gif)

Therefore, by examining the above plot, we can assess the evolution of valuation for each company in the banking sector over time, both in comparison to other companies within the industry and against the sector/industry as a whole. It is hoped that this article has provided you with insights and perhaps inspired you to create your own animated plot. 

Also, remember to visit [Sectors's website](https://sectors.app) for further information and analysis of Indonesian stock market data. 

We look forward to see you in the next installment of the animation plot recipe series!

