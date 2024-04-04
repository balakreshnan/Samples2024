# Based on API reference create documentation

## Introduction

- From API reference in csv or json format
- Here is the sample api list i got - listallapissample.csv
- Only for development and design ideation

## Pre-requisites

- Azure subscription
- Azure Open AI GPT 4 turbo
- Azure machine learning

## Code - Steps

- first load environment

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure open ai

```
import os
import openai
import pandas as pd

openai.api_type = "azure"
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = "2024-02-01"
openai.api_key = config["AZURE_OPENAI_KEY"]
```

- Setup open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  #api_version="2024-02-01"
  api_version="2024-02-15-preview"
)
```

- load the csv file with api information

```
df = pd.read_csv('listallapissample.csv')
```

- reduce the data set

```
df20 = df[:20]
```

- convert to json string

```
jsonstr = df20.to_json(orient='records')
```

- Create the system prompt

```
msgtxt = f"""You are API documentation agent, your job is to create documentation on how to consume the apis.
Only create documentation on the data sources provides.
Source: {jsonstr}

Create detail documentation as Mark down format:
"""
```

- Setup the message

```
message_text = [{"role":"system","content":"You are an AI assistant that helps people find information."},
{"role": "user", "content": msgtxt}]
```

- now create the documentation

```
response = client.chat.completions.create(
    model="gpt-4-turbo", # model = "deployment_name".
    messages=message_text,
    max_tokens=500,
    temperature=0,
    stop=None,
    seed=123,
    tools=None,
    logprobs=True,  # whether to return log probabilities of the output tokens or not. If true, returns the log probabilities of each output token returned in the content of message..
    top_logprobs=2,
)

print(response.choices[0].message.content)
```

- Response

```
API Documentation

Below is the API documentation for various data sources provided in Markdown format.

API Directories

1. APIs.guru

- **API Name:** APIs.guru
- **Category:** API Directories
- **Description:** OpenAPI API directory
- **Sample URL:** `https://api.apis.guru/v2/list.json`

Usage

To retrieve a list of APIs, send a GET request to the sample URL:

http
GET https://api.apis.guru/v2/list.json


2. Public APIs

- **API Name:** Public APIs
- **Category:** API Directories
- **Description:** List of public APIs
- **Sample URL:** `https://api.publicapis.org/entries`

Usage

To get a list of public APIs, send a GET request to the sample URL:

http
GET https://api.publicapis.org/entries


Art & Images

3. Art Institute of Chicago

- **API Name:** Art Institute of Chicago
- **Category:** Art & Images
- **Description:** Artwork from the museum
- **Sample URL:** `https://api.artic.edu/api/v1/artworks/search?q=cats`

Usage

To search for artworks related to "cats", send a GET request to the sample URL:

http
GET https://api.artic.edu/api/v1/artworks/search?q=cats


4. COLOURlovers

- **API Name:** COLOURlovers
- **Category:** Art & Images
- **Description:** Color trends
- **Sample URL:** `https://www.colourlovers.com/api/colors/new?format=json`

Usage

To retrieve the latest color trends in JSON format, send a GET request to the sample URL:

http
GET https://www.colourlovers.com/api/colors/new?format=json


1. DiceBear

- **API Name:** DiceBear
- **Category:** Art & Images
- **Description:** Generate random SVG avatars
- **Sample URL:** `https://api.dicebear.com/6.x/pixel-art/svg`

Usage

To generate a random SVG avatar in the pixel-art style, send a GET request to the sample URL:

http
GET https://api.dicebear.com/6.x/pixel-art/svg


Done with output
```

- we can change the prompt and be creative.