# 0.0 Your First Request!

## Overview

The "Sectors API Cookbook" provides a comprehensive overview and practical guidance for utilizing the Sectors API. This resource serves as a valuable reference for developers, offering insights into the various sectors covered by the API, along with detailed instructions on how to integrate and leverage its functionalities within software applications. This cookbook equips developers with the necessary knowledge and tools to effectively harness the power of the Sectors API in their projects.

## Objective

- Install required library
- Configure the sample
- Run the sample
- Get Started

## Prerequisite

To follow this cookbook, you need the following prerequisites:

- Python
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

In this example we will examine how to make your first request to the sectors API

1. Go to [Sectors API Page](https://www.sectors.app/api) and Retrieve your API Key
2. In your working directory, create this file named `first_request.py`
3. Include the following code in `first_request.py`

   ```python
   import requests
   API_KEY = "Your API KEY"
   url = "https://sectors-api.fly.dev/api/data/subsectors"

   headers = {
    Authorization: API_KEYS
   }

   response = requests.get(url, headers = headers)

   if response.status_code == 200:
       data = response.json()
       print(data)
   else:
       # Handle error
       print(response.status_code)
   ```

## Run your sample

1. In your working directory, build and run the sample:

   ```
   python first_request.py
   ```

2. You should see the list of all subsectors!

## Get Started

On the next chapter we will examine every API that Sectors provide and do some magical :sparkles: visualization using [altair](<(https://pypi.org/project/altair/)>), heads up to the next section.
