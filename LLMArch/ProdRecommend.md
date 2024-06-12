# Product Recommendation/Supply chain recommendation of vendors to fulfill orders using LLM - Azure Open AI GPT 4o models

## Pre-requisites

- Azure Subscription
- Azure AI Studio
- Azure Open AI
- Visual studio code
- Python
- Open ai sdk
- Stream lit app

## Steps

- import the libraries

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
```

- Load environment variables

```
config = dotenv_values("env.env")
```

- Set the open ai client
- Assign deployment variables

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION"], 
  api_key=config["AZURE_OPENAI_KEY_VISION"],  
  api_version="2024-05-01-preview"
  #api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)


model_name = "gpt-4o-g"
```

- Function to process the prompt and get the response

```
def processpdfwithprompt(user_input1, selected_optionmodel1):
    returntxt = ""
    message_text = [
    {"role":"system", "content":"you are provided with instruction on what to do. Be politely, and provide positive tone answers."}, 
    {"role": "user", "content": f"""{user_input1}"""}]

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

- now create the main streamlit app

```
def processtext():
    returntxt = ""

    count = 0
    col1, col2 = st.columns(2)
    with col1:
        modeloptions1 = ["gpt-4o-g", "gpt-4o", "gpt-4-turbo", "gpt-35-turbo"]

        # Create a dropdown menu using selectbox method
        selected_optionmodel1 = st.selectbox("Select an Model:", modeloptions1)
        # Get user input
        user_input1 = st.text_area("Enter your text here:", height=20*20, max_chars=5000, key=count)
        count += 1

    with col2:
        if st.button("Process Text"):
            returntxt = processpdfwithprompt(user_input1, selected_optionmodel1)
            st.markdown(returntxt, unsafe_allow_html=True)
```

- Create the main function

```
def main():
    st.title("Product Recommendation using LLM - Azure Open AI GPT 4o models")
    processtext()
```

- Call the main function

```
if __name__ == "__main__":
    main()
```

- now run the app and we are going to build product recommendation using LLM

```
streamlit run ProdRecommend.py
```

- in the UI on the left text box enter the below prompt which has details about the product and on the right side select the model and click on process text

```
Here is the customer purchase history of person B
Purchase history:
tomatoes - 01/01/2024 - 2 Lbs
apples - 01/01/2024 - 1 lbs
mangoes - 01/01/2024 - 1 lbs
Pepsi   - 02/03/2024 - 6 packs
Gatorade- 02/03/2024 - 12 packs
banana  - 02/03/2024 - 1 lbs
Coriander - 02/03/2024 - 4 bunch
Strawberry - 02/03/2024 - 2 boxes 
Blackberry - 02/03/2024 - 2 boxes
Raspberry - 02/03/2024 - 2 boxes

Personal Profile of B:
Age: 60
Height: 5 feet and 9 inches
BMI is 25%
Diabetic: boarder line
Heart condition: no
Goal: Looking to reduce weight

Here are the available items:
Banana, Strawberry, Blackberry, blue berries, Apples, raspberries, beans,
watermelon, Apples, mangoes, cherry

Can you provide recommendations on products for person B, provide only 4 items based on healthy diet. Only provide recommendations that fit the profile of B based on the profile information provided.
```

- Click Process text and wait for the output

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/prodrecommend1.jpg 'RagChat')

- now lets work on another use case for supply chain management
- Idea here is we have a customer delivery date
- Now we need to find vendors who can fulfill the order on time
- Items can be in multiple warehouse as well
- Items can be multiple vendors as well
- Here is the prompt below

```
Customer A - delivery time is 09/03/2024

Bill of Materials: Required for customer delivery on time
Apple - 1000 lbs - 09/03/2024
Mangoes - 5000 lbs - 09/03/2024
water bottles - 100,000 - 09/03/2024

Vendor availability
Company A - warehouse A - Apples - 200 Lbs - 08/24/2024
Company A - warehouse B - Apples - 300 Lbs - 08/28/2024
Company B - warehouse A - Apples - 2000 Lbs - 08/26/2024
Company B - warehouse B - Apples - 2000 Lbs - 08/21/2024
Company C - warehouse A - Mangoes - 200 Lbs - 08/20/2024
Company C - warehouse B - Mangoes - 2000 Lbs - 08/21/2024
Company C - warehouse C - Mangoes - 10000 Lbs - 08/26/2024
Company A - warehouse A - water bottles - 50,000 Lbs - 08/20/2024
Company A - warehouse B - water bottles - 20,000 Lbs - 08/22/2024
Company B - warehouse A - water bottles - 150,000 Lbs - 08/23/2024
Company C - warehouse A - water bottles - 50,000 Lbs - 08/20/2024
Company C - warehouse B - water bottles - 250,000 Lbs - 09/20/2024
Company C - warehouse C - water bottles - 150,000 Lbs - 08/28/2024

Given the requirement above provided and also vendor product availability, can you provide recommendations on which vendor is likely to deliver the product on time to meet our customer delivery time.
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/supplychain1.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/supplychain2.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/supplychain3.jpg 'RagChat')