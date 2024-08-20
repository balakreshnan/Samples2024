# Building Shopping cart using Natural Language - Gen AI

## Introduction

- Idea is to build a shopping cart using natural language.
- We are using Natural lanugage and using Azure Open AI to process
- Build a shopping cart by identifying the items and their quantity
- Streamlit based application
- Only to show the concept and is not a production ready application
- Please use this for educational purposes only

## Pre-requisites

- Azzure Subscription
- Azure Open AI
- Visual Studio COode
- Streamlit

## Code

- Install necessary libraries
- First lets import the libraries

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
import autogen
from typing import Optional
from typing_extensions import Annotated
from streamlit import session_state as state
```

- initialize shopping cart

```
# Initialize session state for shopping cart
if 'cart' not in st.session_state:
    st.session_state['cart'] = {}

if 'messages' not in st.session_state:
    st.session_state.messages = []

config = dotenv_values("env.env")

css = """
.container {
    height: 75vh;
}
```

- invoke the Azure Open AI

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION"], 
  api_key=config["AZURE_OPENAI_KEY_VISION"],  
  api_version="2024-05-01-preview"
)

#model_name = "gpt-4-turbo"
#model_name = "gpt-35-turbo-16k"
model_name = "gpt-4o-g"
```

```
search_endpoint = config["AZURE_AI_SEARCH_ENDPOINT"]
search_key = config["AZURE_AI_SEARCH_KEY"]
search_index=config["AZURE_AI_SEARCH_INDEX1"]
SPEECH_KEY = config['SPEECH_KEY']
SPEECH_REGION = config['SPEECH_REGION']
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']

citationtxt = ""
```

- Functions to manage the cart

```
def add_to_cart(items):
    txt = ""
    for item in items:
        product = item['product']
        quantity = item['quantity']
        txt += f"Added {quantity} {product}(s) to cart!\n"
        if product in st.session_state['cart']:
            st.session_state['cart'][product] += str(quantity)
        else:
            st.session_state['cart'][product] = str(quantity)
    print(st.session_state['cart'])
    st.success(txt)
    #st.success(f"Added {', '.join([f'{item['quantity']} : {item['product']}(s)' for item in items])} to cart!")

# Function to add a new message to the chat history
def add_message(user_message):
    st.session_state.messages.append({"role": "user", "content": user_message})
    # Simulate a response from the assistant
    st.session_state.messages.append({"role": "assistant", "content": f"Assistant's response to: '{user_message}'"})
```

- Show cart

```
def show_cart():
    st.write("### Your Shopping Cart")
    if not st.session_state.cart:
        st.write("Your cart is empty.")
    else:
        cart_items = [{"item": item, "quantity": quantity} for item, quantity in st.session_state.cart.items()]
        cart_json = json.dumps(cart_items, indent=2)
        st.code(cart_json, language="json")
```

- Prcess input and extract the product and quantity

```
def processinput(user_input1, selected_optionmodel1):
    returntxt = ""

    message_text = [
    {"role":"system", "content":"""You are Shopping cart AI Agent. Be politely, and provide positive tone answers.
     Try to pick the product information and also quantity of the product to add to the cart.
     If not sure, ask the user to provide more information."""}, 
    {"role": "user", "content": f"""{user_input1}. Respond only product name and quantity to add to the cart as JSON array."""}]

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

- now the final function

```
def digiassit():
    #main()
    st.title("Shopping Cart Application")
    count = 0
    
    col1, col2 = st.columns([1,2])
    with col1:
        modeloptions1 = ["gpt-4o-g", "gpt-4o", "gpt-4-turbo", "gpt-35-turbo"]

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
        st.write("### Chat Interface")
        #prompt = st.chat_input("You: ", key="user_input")
        #with st.sidebar:
        #    messages = st.container(height=300)
        #messages = st.container(height=300)
        messages = st.container(height=300)
        if prompt := st.chat_input("i would like to add 7 apples and 5 oranges", key="user_input"):
            messages.chat_message("user").write(prompt)
            #messages.chat_message("assistant").write(f"Echo: {prompt}")
            itemtoadd = processinput(prompt, selected_optionmodel1)
            add_message(prompt)
            #print("Item to add:", itemtoadd)
            item = json.loads(itemtoadd.replace("```", "").replace("json", "").replace("`",""))
            # add_to_cart(item["product"], item["quantity"])
            add_to_cart(item)
            cart_json = json.dumps(item, indent=2)
            messages.chat_message("assistant").write(cart_json)
```

- run the application

```
streamlit run genaishoppingcart.py
```

- Type input and wait

```
i would like to have 4 apples, 6 bananas, 2 coconuts, 1 gallon milk, 1 orange juice
```

- Output

```
[ { "product": "apples", "quantity": 4 }, { "product": "bananas", "quantity": 6 }, { "product": "coconuts", "quantity": 2 }, { "product": "gallon milk", "quantity": 1 }, { "product": "orange juice", "quantity": 1 } ]
```

- my goal was to show the NLP to JSON array and add to cart

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/shopping-01.jpg 'Phi3 128K instruct Model')

- Cart management is not implemented