# Tiles recommendation based on natural language processing

## Introduction

- Provide a interface for users to input their own text and get the recommended tiles based on the text.
- All the tiles information is provided in system prompt for now
- In future this could be from a database or a file
- System prompt will have detail instruction on how the model should respond
- There are some details for coefficients and thresholds in the system prompt

## Pre-requisites

- Azure subscription
- Azure open ai
- Python 3.10 or later
- install open ai library
- install other necessary libraries
- Gather Tiles product information like product name, size available, price, etc.
- Follow the code below

## Code

- import libraries

```
import os
from openai import AzureOpenAI
from dotenv import dotenv_values
import time
from datetime import timedelta
import json
import streamlit as st
from PIL import Image
import base64
import requests
import io
import autogen
from typing import Optional
from typing_extensions import Annotated
from streamlit import session_state as state
import azure.cognitiveservices.speech as speechsdk
from audiorecorder import audiorecorder
import pyaudio
import wave
```

- Load environment variables
- Configure Azure Open AI
- Load other necessary environment variables

```
config = dotenv_values("env.env")

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION_4o_LATEST"], 
  api_key=config["AZURE_OPENAI_KEY_VISION_4o_LATEST"],  
  api_version="2024-05-01-preview"
)

#model_name = "gpt-4-turbo"
#model_name = "gpt-35-turbo-16k"
#model_name = "gpt-4o-g"
model_name = "gpt-4o-2"

search_endpoint = config["AZURE_AI_SEARCH_ENDPOINT"]
search_key = config["AZURE_AI_SEARCH_KEY"]
search_index=config["AZURE_AI_SEARCH_INDEX1"]
SPEECH_KEY = config['SPEECH_KEY']
SPEECH_REGION = config['SPEECH_REGION']
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']

citationtxt = ""
```

- Function to add messages to chat history

```
# Function to add a new message to the chat history
def add_message(user_message):
    st.session_state.messages.append({"role": "user", "content": user_message})
    # Simulate a response from the assistant
    st.session_state.messages.append({"role": "assistant", "content": f"Assistant's response to: '{user_message}'"})
```

- Now here is the actual function that will recommend the tiles based on the text

```
def processinput(user_input1, selected_optionmodel1):
    returntxt = ""

    message_text = [
    {"role":"system", "content":"""Tiles Sizes and Collections Information:\n\n
     
     Here are the tiles sizes available to use:
     24 x 24 inches
     24 x 48 inches
     32 x 64 inches
     48 x 72 inches
     49 x 96 inches
     8 x 48 inches

     Here are the Tile collections available
     Core Mushroom in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Graphite in 1 size which are 24 x 24 inches, in 15mm
     Dolphin in 1 size which are 24 x 24 inches, in 15mm
     Mushroom in 1 Size which are 24 x 48, 24 x 24 inches, in 15mm
     Cookie in 1 size 24 x 24 inches, in 15mm
     Olive in 1 size 24 x 24 inches, in 15mm
     Slate Grey 1 size which are 24 x 24 inches, in 15mm
     Indus Graphite in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Indus Terrain in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Godavari Tan in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Godavari Terrain in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Core cookie in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Core Graphite in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Indus Dolphin in 2 sizes which are 24 x 48 inches, 24 x 24 inches, in 15mm
     Indus Apricot in 1 size which are 2 x 2, 2 x 4 in 15mm Digital full
     Glossy Series
      Ocean Pearl in size 600x1200mm, finish: Glossy
      Ocean Sky in size 600x1200mm, finish: Glossy
      Onxy Grey in size 600x1200mm, finish: Glossy
      Swan Beige in size 600x1200mm, finish: Glossy
     White Series
      Carra Grey in size 600x1200mm, finish: Glossy
      Creta Statuario in size 600x1200mm, finish: Glossy
      Nil Blue in size 600x1200mm, finish: Glossy
      Nil Grey in size 600x1200mm, finish: Glossy
      Smoke White in size 600x1200mm, finish: Glossy
      Spice Gold in size 600x1200mm, finish: Glossy
      Statuario Gold in size 600x1200mm, finish: Glossy
      Statuario Grey in size 600x1200mm, finish: Glossy
      Superb Stuario in size 600x1200mm, finish: Glossy
      Swiss Bianco in size 600x1200mm, finish: Glossy
      Unico White in size 600x1200mm, finish: Glossy
      Zibra Grey in size 600x1200mm, finish: Glossy
     Luxuria Collection
      Altissimo Mint in size 600x1200mm, finish: Glossy
      Astrus Blue in size 600x1200mm, finish: Glossy
      Aura Mint in size 600x1200mm, fisnish: Glossy
      Averaly Aqua in size 600x1200mm, finish: Glossy
      Julian Sky in size 600x1200mm, finish: Glossy
      Nabraska Azul in size 600x1200mm, finish: Glossy
     Light Dark Series
      Amelia Grey in size 600x1200mm
      Armano Grey in size 600x1200mm
      Floris Azul in size 600x1200mm
      Floris Bianco in size 600x1200mm

     Here are the thickness information:
     Thickness are available in 3 sizes
     15mm
     6mm
     9mm

     Here is the options for Finish, can only select one
     Glossy
     Linea Punch
     Matt finish
     Polished
     Polished Matt
     Rustic

     Now you are Tiles expert agent who will provide information to users based on the questions asked and use the above information available above. Customer's can be individual or commercial projects that they would do. Depending on the design and look they are interested in provide them the right tiles to use. At least provide 3 to 5 options to the user. 
     Ask the user to provide square foot information based on which rooms like kitchen, hall, 
     dinning room or rest room or toilet etc to analyze and provide how much tiles are needed based on the sizing information given above.
     Be politely, and provide positive tone answers. Don't get into augments with user.
     Try to provide reasoning on how the recommendations were selected.
     There is a aspect of co effiecient of friction that needs to be considered based on the area where the tiles are used.
     r value is what co effiecient of friction is based on. The higher the r value the more friction the tile has.
     Water absorption is another aspect to consider. The lower the water absorption the better the tile is.
     General area of use is also important to consider. 
     If the tile is used in a high traffic area then the tile should be durable and have a higher r value.
     Only answer from the information provided to best match based on the question asked."""}, 
    {"role": "user", "content": f"""{user_input1}. Provide Recommendations based on the information provided above. 
     If there is square foot provided please calculate the square footage needed."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=0.0,
        seed=105,
    )


    returntxt = response.choices[0].message.content
    return returntxt
```

- Extract only product information for dynamically populate the product list
- Recommendations are coming from model with data provided in system prompt above.

```
def extractproductinfo(user_input1, selected_optionmodel1):
    returntxt = ""

    message_text = [
    {"role":"system", "content":"""You are Tiles expert AI Agent. Be politely, and provide positive tone answers.
     extract recommendation provided and also specificiations to add to the cart.
     If not sure, ask the user to provide more information."""}, 
    {"role": "user", "content": f"""{user_input1}. Respond only recomemndation and details to add to the cart as JSON array."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=0.0,
        seed=105,
    )


    returntxt = response.choices[0].message.content
    return returntxt
```

- Extract tiles calculation from the recommendation provided by the model

```
def extracttilecalcinfo(user_input1, selected_optionmodel1):
    returntxt = ""

    message_text = [
    {"role":"system", "content":"""You are Tiles expert AI Agent. Be politely, and provide positive tone answers.
     extract Tiles calculation and it's details to add to the cart.
     If not sure, ask the user to provide more information."""}, 
    {"role": "user", "content": f"""{user_input1}. Respond only Tile calculation and details to add to the cart as JSON array."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=0.0,
        seed=105,
    )


    returntxt = response.choices[0].message.content
    return returntxt
```

- We can use the information to add to the cart
- main function to provide menu options
- allow users to type in their own text
- Code has sample questions to ask the model

```
def TilesRecom():
    st.title("Tiles Recommendation Assistant")
    count = 0
    col1, col2 = st.columns([1,2])
    with col1:
        modeloptions1 = ["gpt-4o-2", "gpt-4o-g", "gpt-4o", "gpt-4-turbo", "gpt-35-turbo"]

        # Create a dropdown menu using selectbox method
        selected_optionmodel1 = st.selectbox("Select an Model:", modeloptions1)
        count += 1
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
        #now display chat message to store history
        if st.session_state.messages:
            st.write(st.session_state.messages)
            #print('Chat history:' , st.session_state.messages)
    with col2:
        messages = st.container(height=300)
        if prompt := st.chat_input("I have a 2000 square foot bathroom walls and i am looking for contemporary tiles to go with my ivory painted walls", key="user_input"):
            messages.chat_message("user").write(prompt)
            #messages.chat_message("assistant").write(f"Echo: {prompt}")
            itemtoadd = processinput(prompt, selected_optionmodel1)
            add_message(prompt)
            #print("Item to add:", itemtoadd)
            #item = json.loads(itemtoadd.replace("```", "").replace("json", "").replace("`",""))
            # add_to_cart(item["product"], item["quantity"])
            # add_to_cart(item)
            #cart_json = json.dumps(item, indent=2)
            #messages.chat_message("assistant").write(cart_json)
            messages.chat_message("assistant").write(itemtoadd)
            if itemtoadd:
                st.write("### Product Information")
                productinfo = extractproductinfo(itemtoadd, selected_optionmodel1)
                st.write(productinfo)
                tilecalc = extracttilecalcinfo(itemtoadd, selected_optionmodel1)
                st.write("### Tile Calculation Information")
                st.write(tilecalc)
```

- Run the main function

```
if __name__ == "__main__":
    TilesRecom()
```

- done