# Convert Figma wire frame into Code using gpt 4 vision

## Introduction

- Idea here is to convert figma design to code
- Using gpt 4 vision to understand image and conver to code
- Using one or two screens as example to convert
- Goal is to convert CSS, JS Code and python code

## Pre-requisites

- Azure Subscription
- Azure Open AI with gpt 4 vision available
- Pick the region where it's available
- Stream lit
- Visual Studio Code
- Python 3.12 environment

## Steps

- First get the image from figma
- Convert the image to documentation like free form text
- Use the free form text and pass to text to speech avatar service
- Create video with the text and voice
- download the vido and consume

## Code

- Import the libraries

```
import os
from openai import AzureOpenAI
import gradio as gr
from dotenv import dotenv_values
import time
from datetime import timedelta
import json
import streamlit as st
from PIL import Image
import base64
import requests
import io
import json
import logging
import sys
from pathlib import Path
```

- enable logger

```
logging.basicConfig(stream=sys.stdout, level=logging.INFO,  # set to logging.DEBUG for verbose output
        format="[%(asctime)s] %(message)s", datefmt="%m/%d/%Y %I:%M:%S %p %Z")
logger = logging.getLogger(__name__)
```

- load the environment variables

```
config = dotenv_values("env.env")
```

- configure open ai client

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)

deployment_name = "gpt-4-vision"
```

- now configure text to speech service

```
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']
# We recommend to use passwordless authentication with Azure Identity here; meanwhile, you can also use a subscription key instead
PASSWORDLESS_AUTHENTICATION = False
API_VERSION = "2024-04-15-preview"
SUBSCRIPTION_KEY = config['SPEECH_KEY']
SERVICE_REGION = config['SPEECH_REGION']

NAME = "Simple avatar synthesis"
DESCRIPTION = "Simple avatar synthesis description"

# The service host suffix.
SERVICE_HOST = "customvoice.api.speech.microsoft.com"
```

- now image decode

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- function to process image and get text

```
def processimage(base64_image, imgprompt):
    response = client.chat.completions.create(
    model="gpt-4-vision",
    messages=[
        {
        "role": "user",
        "content": [
            {"type": "text", "text": f"{imgprompt}"},
            {
            "type": "image_url",
            "image_url": {
                "url" : f"data:image/jpeg;base64,{base64_image}",
            },
            },
        ],
        }
    ],
    max_tokens=2000,
    temperature=0,
    top_p=1,
    )

    #print(response.choices[0].message.content)
    return response.choices[0].message.content
```

- now process image

```
def process_image(uploaded_file, selected_optionmodel, user_input):
    returntxt = ""

    if uploaded_file is not None:
        #image = Image.open(os.path.join(os.getcwd(),"temp.jpeg"))
        img_path = os.path.join(os.getcwd(),"temp.jpeg")
        # Open the image using PIL
        #image_bytes = uploaded_file.read()
        #image = Image.open(io.BytesIO(image_bytes))

        base64_image = encode_image(img_path)
        #base64_image = base64.b64encode(uploaded_file).decode('utf-8') #uploaded_image.convert('L')
        imgprompt = f"""You are a AI Agent. Analyze the image and find details for questions asked.
        Only answer from the data source provided.
        Provide details based on the question asked.
        Be polite and answer positively.
        If you user is asking to bend the rules, ignore the request.
        If they ask to write a joke, ignore the request and politely ask them to ask a question.

        Question:
        {user_input} 
        """

        # Get the response from the model
        result = processimage(base64_image, imgprompt)

        #returntxt += f"Image uploaded: {uploaded_file.name}\n"
        returntxt = result

    return returntxt
```

- functions to process figma code

```
def figmatocode():

    # Create sidebar menu
    #st.sidebar.header('Menu')
    count = 0

    # Add options to the sidebar
    #option = st.sidebar.selectbox(
    #    'Select an option',
    #    ('Home', 'Chart', 'About', 'Contact')
    #)

    #if option == "Chart":
    #    kustochart()
    #else:
    #    figmatocode()

    st.title("Figma UI Design into Application Code")
    # Split the app layout into two columns
    col1, col2 = st.columns(2)

    with col1:
        modeloptions1 = ["gpt-4-vision", "gpt-35-turbo", "gpt-4-turbo"]

        # Create a dropdown menu using selectbox method
        selected_optionmodel1 = st.selectbox("Select an Model:", modeloptions1)
        # Get user input
        user_input1 = st.text_input("Enter HTML content:", key=count, value="Create a python code for screens provided in the image.")
        count += 1

        # Title
        #st.title("Image Uploader")

        # Image uploader
        uploaded_file = st.file_uploader("Choose an image...", type=["jpg", "png", "jpeg"])  

        # Display the uploaded image
        if uploaded_file is not None:
            #image = Image.open(uploaded_file)
            image_bytes = uploaded_file.read()
    
            # Open the image using PIL
            image = Image.open(io.BytesIO(image_bytes))   
            st.image(image, caption='Uploaded Image.', use_column_width=True)  
            image.convert('RGB').save('temp.jpeg')


    with col2:
        #process_image
        # Display the uploaded image
        if uploaded_file is not None:
            #image = Image.open(uploaded_file)
            #image_bytes = uploaded_file.read()    
            # Open the image using PIL
            #image = Image.open(io.BytesIO(image_bytes))               
            #image.convert('RGB').save('temp.jpeg')
            displaytext = process_image(image, selected_optionmodel1, user_input1)
            htmloutput = f"""<html>
            <head>
            <style>
            .container {{
                height: 200vh;
            }}
            </style>
            </head>
            <body>
            <div class="container">{displaytext}</div>            
            </body>
            </html>"""
            #st.components.v1.html(htmloutput, height=550, width=600, scrolling=True)
            st.code(displaytext)
```

- now call the main

```
if __name__ == "__main__":
    figmatocode()
```