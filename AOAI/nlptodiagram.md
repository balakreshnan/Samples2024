# Natural Language Processing to Diagram using mermaid

## Introduction

- Convert NLP to Diagram
- Use natural language to create flow or architecture diagram
- Using mermaid JS library
- Right now only block diagram and we can build more later
- Using Visual Studio Code
- Using Python 3.11
- Azure open ai to generate the diagram
- Streamlit as front end

## Pre-requisite

- Azure account
- Azure open ai
- Visual Studio Code
- Python 3.11
- Streamlit
- Mermaid JS

## Steps

### Code

- First import the required libraries

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
import PyPDF2
```

- Load the environment variables

```
config = dotenv_values(".env")
```

- Configure open ai client and LLM configuration

```

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_ASSITANT"], 
  api_key=config["AZURE_OPENAI_KEY_ASSITANT"],  
  api_version="2024-02-15-preview"
  #api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)

#model_name = "gpt-4-turbo"
model_name = "gpt-35-turbo-16k"

llm_config={"config_list": [
    {"model": "gpt-35-turbo", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"},
    {"model": "gpt-35-turbo-16k", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"},
    {"model": "gpt-4-turbo", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"}
    ],
    "timeout": 600,
    "cache_seed": 44,
    "temperature": 0.0}
```

- function to create the mermain code based on the input

```

def processdiagramprompt(selected_optionmodel, user_input1):
    returntxt = ""

    start_time = time.time()
    system_prompt = f"""You are Mermaid JS Chart Expert in javascript. Provide only the mermaid javascript code without any explanation.
    Adjust the height and width to show the chart properly.
    Show axis labels, values and title for the chart. Also show series name in the legend.
    Do Not provide any explanation or comments in the code. Use the dataset provide below.
    Create code for D3 version 6.0.0
    Only use the data provided. if can't find data then respond with no data found.
    Use Camel case for javascript functions.
    Create D3 code for version 6.0.0 and above
    Use Javascript for parsing date using Date function and format the date to yyyy-mm-dd format.

    Sources:
    {user_input1}
    """

    #message = f"Create a {selected_optioncharttype} chart plot for stocks for 6 months of tesla?"
    #message1 = f"Create a {selected_optioncharttype} chart for {message}"
    #message1 = f"{message}"
    message1 = f"Create a {user_input1} mermaid code."

    print('Chart selected: ' , message1)

    messages = [
        { "role": "system", "content": system_prompt },
        { "role": "user", "content": message1.lower() },
    ]

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=selected_optionmodel,
        #messages=history_openai_format,
        messages=messages,
        seed=42,
        temperature=0.0,
        max_tokens=2000,
        top_p=1.0,
        #stream=True
    )

    #print("KQL query created: ", response.choices[0].message.content)
    
    #htmloutput = """<script> 
    #response.choices[0].message.content
    #</script>"""
    returntxt = response.choices[0].message.content.replace("```","").replace("Javascript","").replace("javascript","").replace("mermaid","").strip()

    return returntxt
```

- format the output to show in the streamlit

```
def mermaid_chart(diagcode):
    html_code = f"""
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.1/css/all.min.css">
    <div class="mermaid">{diagcode}</div>
    <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
    <script>mermaid.initialize({{startOnLoad:true}});</script>
    """
    return html_code
```

- Main streamlit UI code to show the input and output

```
def processdiagrams():
    
    count = 0
    col1, col2 = st.columns(2)
    rtext = ""

    with col1:
        modeloptions = ["gpt-35-turbo", "gpt-4-turbo", "llama2", "mixstral"]

        # Create a dropdown menu using selectbox method
        selected_optionmodel = st.selectbox("Select an Model:", modeloptions)
        
        # Get user input
        user_input = st.text_input("Enter scenario to create diagrams:")

        if st.button("Submit"):
            count += 1
            
            rtext = processdiagramprompt(selected_optionmodel, user_input)

            chart_js = mermaid_chart(rtext)
            print("Javascript Text: ", chart_js)
            # Embed HTML content with JavaScript code for the chart
            st.components.v1.html(chart_js, height=350, width=600, scrolling=True)

    with col2:
        # Create a button for the user to submit the form
        if rtext:
            #chart_js = f"""
            #<div class="mermaid">
            #{rtext}
            #</div>
            #<script type="module">
            #import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10.9.0/dist/mermaid.min.js';
            #</script>
            #"""

            chart_js = mermaid_chart(rtext)
            print("Javascript Text: ", chart_js)
            # Embed HTML content with JavaScript code for the chart
            # st.components.v1.html(chart_js, height=350, width=600, scrolling=True)
            #st.code(chart_js, language='Javascript')

if __name__ == "__main__":
    processdiagrams()
```

- Run the streamlit app
- try few prompts

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/diag1.png 'RagChat')