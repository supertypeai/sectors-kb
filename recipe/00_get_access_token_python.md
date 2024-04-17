# 0.0 Intro

## Overview

The "Sectors API Cookbook" provides a comprehensive overview and practical guidance for utilizing the Sectors API. This resource serves as a valuable reference for developers, offering insights into the various sectors covered by the API, along with detailed instructions on how to integrate and leverage its functionalities within software applications. This cookbook equips developers with the necessary knowledge and tools to effectively harness the power of the Sectors API in their projects. We will use

## Objective

- Install required library
- Configure the sample
- Run the sample
- Get Started

## Prerequisite

To follow this cookbook, you need the following prerequisites:

- Python 3.10 or greater
- The [pip](https://pypi.org/project/pip/) management tool
- Sectors account

## Install required libraries

- Install the [requests](https://requests.readthedocs.io/en/latest/) library to make HTTP Requests

- Install [pandas](https://pypi.org/project/pandas/) to do some data exploratory
- In this cookbook we will use [altair](https://pypi.org/project/altair/) to do the data visualization

  ```
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

On the next chapter we will examine every API that Sectors provide and do some magical :sparkles: visualization using [altair](<(https://pypi.org/project/altair/)>), heads up to the next section.
