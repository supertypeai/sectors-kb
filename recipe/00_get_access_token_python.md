# 0.0 Intro

## Overview

This section of the repository hosts a collection of recipes that provide working code examples featuring the [Sectors API](https://sectors.app/api). It is a community-led effort to illustrate the capabilities of the Sectors API and demonstrate how to integrate it into various software applications -- commonly Python, R, and JavaScript. 

Each recipe is written to be concise, easy to follow, and provide practical guidance for developers looking to leverage the Sectors API in their projects. If you wish to contribute a recipe or have any questions, please feel free to reach out to us on our [Discord channel](https://discord.gg/TAnZMmNS4X) or follow the [contribution guidelines](https://github.com/supertypeai/sectors-kb/tree/main/recipe#community-contributor).

In this particular recipe, we will demonstrate how to get the access token by logging in using your Sectors account. This is the first step to authenticate your request to the Sectors API.

## Objective

- Install required library
- Configure the sample
- Run the sample
- Get Started

## Prerequisite

To follow this recipe, you need the following prerequisites:

- Python (we recommend Python 3.6 or higher)
- The [pip](https://pypi.org/project/pip/) management tool
- Sectors account (logged in, or sign up [here](https://sectors.app/))

## Install required libraries

- Install the [requests](https://requests.readthedocs.io/en/latest/) library; This library will be used to make HTTP requests to the Sectors API

- Install [pandas](https://pypi.org/project/pandas/) to do some data exploratory

- In this recipe we will be using [altair](https://pypi.org/project/altair/) for the data visualization

```bash
pip install requests
pip install pandas
pip install altair
```

## Configure the sample

In this example we will examine how to get the access token by logging in using your Sectors account.

1. In your working directory, create this file named `get_access_token.py`
2. Include the following code in `get_access_token.py`

   ```python
   import requests

   url = "https://sectors-api.fly.dev/api/token/"

   body = {
       "email": "your_email",
       "password": "your_password"
   }

   response = requests.post(url, json = body)

   if response.status_code == 200:
       data = response.json()
       access_token = data["access"]
   else:
       # Handle error
       print(response.status_code)
   ```

## Run your sample

1. In your working directory, build and run the sample:

   ```
   python get_access_token.py
   ```

2. You should see the access token that looks like this
    ```
    eyJhbGciOiJIUzIxxxx...xxx
    ```
    This access token will be used in the future in this cookbook.

## Get Started

In the next chapter we will examine some of the Sector API endpoints perform some magical :sparkles: visualization using [altair](<(https://pypi.org/project/altair/)>). 

Whenever you're ready, [dive into the next chapter](./01_list_all_subsectors.md) to get started.
