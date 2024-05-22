# GPT 4o - Getting started

## Introduction

GPT 4o is a powerful language model that can generate human-like text. It is the fourth iteration of the GPT series, and it has been trained on a diverse range of internet text to improve its language capabilities. In this guide, we will walk you through the basics of using GPT 4o and help you get started with generating text using this powerful model.

## Pre-requisites

- Azure Subscription
- Azure Open AI GPT 4o in East us or west us 3 region
- Azure Open AI GPT 4o key
- open ai sdk

## Steps

- Get the open ai key and endpoint
- Deploy gpt 4o model and get the deployment name
- Install libraries

```
%pip install --upgrade openai --quiet
%pip install opencv-python --quiet
%pip install moviepy --quiet
%pip install python-dotenv
```

- Import libraries

```
import base64
import requests
from IPython.display import Image, display, Audio, Markdown
import base64
```

- load the config

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_4o"], 
  api_key=config["AZURE_OPENAI_KEY_4o"],  
  #api_version="2024-02-01"
  #api_version="2024-05-01"
  api_version="2024-05-01-preview"
)
```

- Encode image

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- set the image file name

```
image_path = "dmvform1.jpg"
```

