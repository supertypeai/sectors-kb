---
title: "SectorScan - Part 2: Bring the app to life"
author: "Aurellia Christie"
author_link: "https://github.com/AurelliaChristie"
date: "2024-05-30"
language: "Python"
---

# SectorScan - Part 2: Bring the app to life

Hey there, welcome to the second part of SectorScan development recipe! If you haven't checked out the first part of this recipe, please check it [here](01_sectorscan_part1.md). As I mentioned before, in this part, we'll elevate everything we've accomplished by seamlessly integrating it into Streamlit. This will transform our work into a fully functional, interactive app. Get ready to see SectorScan come to life as a powerful tool for financial analysis and visualization! To give you an overview of what SectorScan will look like, you can visit [this link](https://sectorscan.streamlit.app/).

Note: to follow along this recipe, you can't use Google Colab anymore. Please use Visual Studio Code or any other local IDE of your choice.

# Getting familiar with Streamlit

Why do we choose Streamlit to build SectorScan? Because it has these key features:
- Simple: Streamlit's intuitive API allows you to create powerful web apps with minimal effort.
- Fast: Quickly iterate on your app and see changes in real-time with Streamlit's automatic hot reloading.
- Flexible: Streamlit supports a wide range of data visualization libraries, making it easy to incorporate charts, tables, and other visualizations into your app.
- Sharing: Share your apps with others by deploying them to Streamlit's cloud platform or hosting them on your own server.

The first thing we wanna do here is to get familiar with how to create your first Streamlit app:
1. Let's create a folder called `sectorscan` in your chosen working directory and create a file called `sectorscan.py` inside it. 
2. Please make sure you've already installed `streamlit` in your local. If you've done that, now import it to your Python script (`sectorscan.py`) using this code:
   
    ```python
    import streamlit as st

    # Import some other libraries that we're going to need in our code later
    import pandas as pd
    import requests
    import altair as alt
    ```

3. Using [this documentation](https://docs.streamlit.io/develop/api-reference) as your reference, write your app using Streamlit's simple API:

    ```python
    # Title
    st.title("SectorScan")

    # Content
    st.write("Welcome to SectorScan!")
    ```

4. Run your app from the command line:
   
   ```
   streamlit run sectorscan.py
   ```

5. That's it! You've created your first Streamlit app. The previous command line you run should redirect you to the browser showing your app as follows:

    ![streamlit_start](/recipe/python_app/sectorscan/image/streamlit_start.png)

    If you want to stop the app, just hit `CTRL + C` in your command line. 

In the next section, we'll start to move everything we've created in the previous part to this app.

# Build the SectorScan

To make your life easier, keep [this documentation](https://docs.streamlit.io/develop/api-reference) open while you follow along with this tutorial. Feel free to also modify my code if you find a better way to display the app's contents!

## Data retrieval function

First thing that we want to move to our app is the function to retrieve [Sector](https://sectors.app/api)'s data. But certainly, there are some modifications we need to do. One thing we need to modify is the way we call the API key. There's no way we're going to display the API key right away in our code for our safety. Thus, we're going to utilize [streamlit secrets](https://docs.streamlit.io/develop/api-reference/connections/st.secrets) in our function:

- Create a folder called `.streamlit` in your `sectorscan` folder, and fill it with a file named `secrets.toml`.
- In the TOML file, create a new variable called `SECTORS_KEY` and paste your API key there:
    
    ```
    SECTORS_KEY="Your API Key"
    ```
- Access the secrets from your Python script using `st.secrets`:

    ```python
    api_key = st.secrets["SECTORS_KEY"]
    ```

Next, to handle error from API request, previously we use `raise Exception`. Since we're creating an app, we need to let the user knows if something went wrong, but in the same time, not being too specific. Thus, we'll use [`st.error()`](https://docs.streamlit.io/develop/api-reference/status/st.error) to notice the user about the error:

```python
st.error("Error: Something went wrong. Please reload the app.")
```

We'll also add a [`st.cache_data`](https://docs.streamlit.io/develop/api-reference/caching-and-state/st.cache_data) decorator to our function to make use of cache on our function calls. Cache is a mechanism for temporarily storing frequently accessed data to improve performance and reduce redundant computations. 

So the final function looks like this:
```python
@st.cache_data
def fetch_data(url):
  api_key = st.secrets["SECTORS_KEY"]

  headers = {
      "Authorization": api_key
  }

  response = requests.get(url, headers = headers)

  if response.status_code == 200:
      return response.json()
  else:
      # Handle error
      st.error("Error: Something went wrong. Please reload the app.")
```

You can test the function by retrieving the sectors data and display it as a text first using [`st.write()`](https://docs.streamlit.io/develop/api-reference/write-magic/st.write):
```python
url = "https://api.sectors.app/api/data/subsectors/"
sectors = fetch_data(url)
sectors.sort() # sort the list based on the sector's name to make it tidier
st.write(sectors)
```

Notice that your app won't be changed when you modify your code:

![modified](/recipe/python_app/sectorscan/image/modified.png)

To automatically reload your app whenever the source code change, click the `Always rerun` button on the top right of the page. But if you only want to reload the app when you ask the app to do so, click the `Rerun` button instead. If the function works, your app should look like this after you click the button (I've removed the welcome text to make the app tidier):

![print_sectors](/recipe/python_app/sectorscan/image/print_sectors.png)

Ok since the function works already, remove the `st.write()` and let's continue to bring more content to our app!

## Sectors filter component

Since we want to compare data from different sectors in this app, we want to make sure that we provide a component for the user to choose which sectors they want to compare. Thus, we're going to provide a multiselect filter that displays all the available sectors. We'll use [`st.multiselect()`](https://docs.streamlit.io/develop/api-reference/widgets/st.multiselect) for this:
```python
options = st.multiselect(
    label="Filter Sectors",
    options=sectors,
    default=sectors[0:3], 
    max_selections=5
)
```

We use `options` variable to contain the selected sectors to be used across our code later. And to ensure the visualizations remain clear and easy to read, we limit the maximum selections to 5 sectors. When your app reloads, it should show the multiselect filter as follows:

![filter](/recipe/python_app/sectorscan/image/filter.png)

Notice that the options are in `..-..` format, so we'll utilize `format_func` parameter of the `st.multiselect()` to display the options in a prettier way:

```python
def format_option(option):
    return option.replace("-", " ").title() # change - to space and use title case

options = st.multiselect(
    label="Filter Sectors",
    options=sectors,
    default=sectors[0:3], 
    format_func=format_option,
    max_selections=5
)
```

Ok, now we have a better filter to be displayed:

![better_filter](/recipe/python_app/sectorscan/image/better_filter.png)

If you read the documentation of [`st.multiselect()`](https://docs.streamlit.io/develop/api-reference/widgets/st.multiselect), there's no option to disable the `clear options` button, thus, we have to make sure that we handle the case when user clears the options (no sector is selected). We'll use [`st.warning()`](https://docs.streamlit.io/develop/api-reference/status/st.warning) to warn the user about this:
```python
if len(options) == 0:
    st.warning("Please select at least one sector!")
else:
    st.write("Visualization") # later on change it to the visualization codes
```

Now if you clear the options you'll see this warning:

![warning](/recipe/python_app/sectorscan/image/warning.png)

## Market cap section

In the market cap section, we're going to display two rows:
- The first row will be divided into two columns to display the total market cap and historical market cap visualization 
- The second row will show the historical market cap change visualization.

But first, of course we need to move all the code to retrieve the market cap data to the `sectorscan.py`. Copy paste the last chunk of code of [this section](01_sectorscan_part1.md#data-processing). 

Since now we're retrieving the data based on the selected sectors, we need to modify the code a bit. Instead of using `range(3)`, we'll change it to our `options` variable:
```python
df_mc_curr = pd.DataFrame()
df_mc_hist = pd.DataFrame()
df_mc_change = pd.DataFrame()

for i in options:
    url = f"https://api.sectors.app/api/data/sector/report/{i}/?sections=market_cap"
    market_cap = fetch_data(url)
    ... # the rest of the code
```

### First row of the market cap section

To process the first row, don't forget to also move the following codes to `sectorscan.py`:
- Code to produce the final data frame of `df_mc_curr` and the `mc_curr_chart` code in [this section](01_sectorscan_part1.md#total-market-cap-visualization)
- Code to produce the final data frame of `df_mc_hist` and the `mc_hist_chart` code in [this section](01_sectorscan_part1.md#historical-market-cap-visualization).

To display the first row as two columns, we'll use [`st.columns()`](https://docs.streamlit.io/develop/api-reference/layout/st.columns). And to display the `altair` charts, we'll use [`st.altair_chart()`](https://docs.streamlit.io/develop/api-reference/charts/st.altair_chart):
```python
# Market cap section title
st.subheader("Market Cap")

# First row visualization
col1, col2 = st.columns(
    spec=[0.6, 0.4], # relative width of each column
    gap="large" # size of gap between column
)

# First column
with col1:
    st.altair_chart(mc_curr_chart)

# Second column
with col2:
    st.altair_chart(mc_hist_chart)
```

The result will be as follow:

![mc_first_row](/recipe/python_app/sectorscan/image/mc_first_row.png)

Feel free to modify the width of the columns and the charts.

### Second row of the market cap section

For the second row, let's first move the code to produce the final data frame of `df_mc_change` and the code of `mc_change_chart` from [this section](01_sectorscan_part1.md#historical-market-cap-change-visualization) to `sectorscan.py`.

Since the second row is not divided into columns, we can directly use `st.altair_chart()` to display the chart:
```python
# Second row visualization
st.altair_chart(mc_change_chart)
```

The result will be as follow:

![mc_second_row](/recipe/python_app/sectorscan/image/mc_second_row.png)

## Valuation section

First let's move the code to retrieve valuation data from the last chunk of code of [this section](01_sectorscan_part1.md#data-processing-1). Don't forget to change the `range(3)` to `options`:
```python
df_valuation = pd.DataFrame()

for i in options:
    url = f"https://api.sectors.app/api/data/sector/report/{i}/?sections=valuation"
    valuation = fetch_data(url)
    ... # the rest of the code
```

In this section, we're going to only have one visualization, but we'll let the user choose what valuation metric they want to show. We'll use [`st.selectbox()`](https://docs.streamlit.io/develop/api-reference/widgets/st.selectbox) to do so:
```python
# Valuation section title
st.subheader("Valuation")

# Valuation metric filter
option = st.selectbox(
    label="Select Valuation Metric",
    options=("Price/Book Ratio", "Price/Earning Ratio", "Price/Sales Ratio", "Price/Cash Flow Ratio")
)
```

The `option` variable will contain the selected valuation metric and we'll use it to make our `valuation_chart` to be dynamic. Move the `valuation_chart` code from [this section](01_sectorscan_part1.md#valuation-visualization) to `sectorscan.py`, but change all the `Price/Book Ratio` to `option`. Then, display it using `st.altair_chart()`:
```python
# valuation_chart
valuation_chart = alt.Chart(df_valuation).mark_line(
    point=True # add individual data points to the line chart
).encode(
    x=alt.X("Year:N", axis=alt.Axis(labelAngle=0)), # 0 degree of x-axis label angle
    y=alt.Y(f"{option}:Q"),
    color=alt.Color(
        "Sector:N",
        scale=alt.Scale(scheme="lightgreyred"),
        sort=alt.SortField(field="Sector", order="ascending") # sort color based on the sector's name
    ),
    tooltip=[
        alt.Tooltip("Sector:N"),
        alt.Tooltip("Year:N"),
        alt.Tooltip(f"{option}:Q", format=".2f")
    ]
).properties(
    title=f"{option} Across Sectors",
    width=900
)

st.altair_chart(valuation_chart)
```

Your valuation section should look like this:

![valuation](/recipe/python_app/sectorscan/image/valuation.png)

If you change the selection, the chart should be updated based on it.

## Top companies section

In this section, we'll use tabs to display the visualizations. There are four tabs we want to create here:

- `Market Cap` to display Top companies based on Market Cap
- `Growth` to display Top companies based on Revenue Growth
- `Profit` to display  Top companies based on Profit
- `Revenue` to display Top companies based on Revenue

Let's first move the code to retrieve top companies data from the last chunk of code in [this section](01_sectorscan_part1.md#data-processing-2) to `sectorscan.py`. Also change the `range(3)` to `options`:
```python
df_top_mc = pd.DataFrame()
df_top_growth = pd.DataFrame()
df_top_profit = pd.DataFrame()
df_top_revenue = pd.DataFrame()

for i in options:
    url = f"https://api.sectors.app/api/data/sector/report/{i}/?sections=companies"
    company = fetch_data(url)
    ... # the rest of the code
```

Also move the following codes to `sectorscan.py`:

- Code to produce the final data frame of `df_top_mc` and the `mc_chart` code in [this section](01_sectorscan_part1.md#top-companies-based-on-market-cap-visualization)
- Code to produce the final data frame of `df_top_growth` and the `growth_chart` code in [this section](01_sectorscan_part1.md#top-companies-based-on-revenue-growth-visualization)
- Code to produce the final data frame of `df_top_profit` and the `profit_chart` code in [this section](01_sectorscan_part1.md#top-companies-based-on-profit-visualization)
- Code to produce the final data frame of `df_top_revenue` and the `revenue_chart` code in [this section](01_sectorscan_part1.md#top-companies-based-on-revenue-visualization).

Then, to display all the visualizations on each of its tab, we'll use [`st.tabs()`](https://docs.streamlit.io/develop/api-reference/layout/st.tabs):
```python
# Top companies section title
st.subheader("Top Companies")

# Top companies visualization
tab1, tab2, tab3, tab4 = st.tabs(
    tabs=["Market Cap", "Growth", "Profit", "Revenue"]
)

# First tab
with tab1:
    st.altair_chart(mc_chart)

# Second tab
with tab2:
    st.altair_chart(growth_chart)

# Third tab
with tab3:
    st.altair_chart(profit_chart)

# Fourth tab
with tab4:
    st.altair_chart(revenue_chart)
```

Your top companies section should look like this:

![top_companies](/recipe/python_app/sectorscan/image/top_companies.png)

When you click on another tab's title, it should display the chart based on the title.

# (Optional) Deploy the SectorScan

Actually if you only need your app locally, you can stop right here. But if you want your app to be accessible anywhere, you need to continue to this section. 

## Prerequisite

Some additional prerequisites are required to be able to deploy the app:

- [Github account](https://github.com/signup)
- [Streamlit account](https://authkit.streamlit.io/sign-up)

## Step by step

We're going to follow [this documentation](https://docs.streamlit.io/deploy/streamlit-community-cloud/deploy-your-app) to deploy our app:

1. Login and go to [Github](https://github.com/), then click the `New` button:
   
   ![new_repo](/recipe/python_app/sectorscan/image/new_repo.png) 

2. Fill in the repository details, more or less as follows:

    ![github](/recipe/python_app/sectorscan/image/github.png)

3. Click the `Create repository` button.
4. Now, prepare your files:

    - `sectorscan.py` that we've created throughout this recipe.
    - `requirements.txt` with the content as follows:
  
        ```txt
        streamlit==1.35.0
        ```
5. Upload both of the files to the repository by clicking the `Add file` - `Upload files` button on the repository page:

    ![upload_files](/recipe/python_app/sectorscan/image/upload_files.png)

    Then, click on the `Commit changes` button. Your repository is ready to be deployed! Your repository page now should look like this:

    ![final_repo](/recipe/python_app/sectorscan/image/final_repo.png)

6. Login to [Streamlit](https://authkit.streamlit.io/) to deploy the app. 
7. Click the `Create app` button on the top right:

    ![create_streamlit](/recipe/python_app/sectorscan/image/create_streamlit.png)

8. Click on the `Yup, I have an app` option.
9. Fill in the app detail as follows:
    
    ![streamlit_detail](/recipe/python_app/sectorscan/image/streamlit_detail.png)

    You should choose different app URL since I've used `sectorscan.streamlit.app`.

10. Click the `Advance settings...` link below the `App URL` and paste the content of `secrets.toml` in there:
    
    ![secrets](/recipe/python_app/sectorscan/image/secrets.png)

    Then, click the `Save` button.

11. Last step, click the `Deploy!` button and congrats, your app is ready!

# Go beyond

Congratulations on completing this recipe! You've taken the first step in creating a powerful financial analytics tool. Now, the possibilities are endless. You can always expand and improve your app by adding more visualizations and analyses. Dive deeper into the data available from the [Sectors API](https://sectors.app/api) to uncover new insights and make your app even more comprehensive and insightful. Happy coding!

Note: The full version of the code can be accessed [here](https://github.com/AurelliaChristie/sectorscan).

