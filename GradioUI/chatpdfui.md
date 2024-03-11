# Chat UI using Gradio and upload pdf and ask questions

## Introduction

- Validate your Gen AI solution using Gradio
- Using your own pdf and ask questions
- Options to provide system message/ prompt to validate the output
- Ability to choose models

## Pre-requisite

- Azure Subscription
- Azure Open AI Services
- Deploy Gpt 4 turbo 128K and gpt 3.5 4K token models
- python 3.11 or later
- Local laptop with visual studio code or any other IDE

## Steps

- Install Gradio

```
pip install gradio
```

- Install Open AI

```
pip install openai
```

- Install PyPDF2

```
pip install PyPDF2
```

- install python-dotenv

```
pip install python-dotenv
```

- Import all libraries

```
import os
from openai import AzureOpenAI
import gradio as gr
from dotenv import dotenv_values
import time
from PyPDF2 import PdfReader
import base64
from io import BytesIO
import tempfile
import PyPDF2
```

- Load the environment variables

```
config = dotenv_values("env.env")
```

- Now configure the Azure Open AI Client

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2023-09-01-preview"
)
```

- Upload function to process pdf and read content

```

def abstract_extract(uploaded_file):

    abstract = ""
    return_text = []
    all_text = ""

    try:
        #print('hello')
        with open(uploaded_file, "rb") as f:
            # Create a PDF reader object
            pdf_reader = PyPDF2.PdfReader(f)

            # Iterate over all the pages in the PDF file
            for page_num in range(len(pdf_reader.pages)):
                # Get the page object
                page = pdf_reader.pages[page_num]

                # Extract the text from the page
                text = page.extract_text()

                # Print the extracted text
                # print(text)
                return_text.append(text)
        #pdf_bytes = BytesIO(uploaded_file)
        #pdf_reader = PdfReader(pdf_bytes)
        

        all_text = " ".join(return_text)
    except Exception as e:
        print('Error:', e)



    return all_text
```

- Funtion to process the UI

```
def predict(message, history, firstllm, system_prompt, file):
    history_openai_format = []
    #file_paths = [file.name for file in upload_button]
    pdftext = ""
    #print('upload_button:', file)
    try:
        #file_paths = upload_file(files)
        print('upload_button:', file)
        pdftext = abstract_extract(file)

    except Exception as e:
        print('Error:', e)
        file_paths = []

    sysmsg = f"""
    {system_prompt}
    Sources:
    {pdftext}
    
    Answer:"""    

    # print('system_prompt:', sysmsg)
    
    history_openai_format.append({"role": "system", "content": sysmsg })
    for human, assistant in history:
        history_openai_format.append({"role": "user", "content": human })
        history_openai_format.append({"role": "assistant", "content":assistant})
    history_openai_format.append({"role": "user", "content": message})

    start_time = time.time()

    print('model:', firstllm)
    # print('Abstract Text:', pdftext)    

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=firstllm,
        messages=history_openai_format,
        #stream=True
    )

    partial_message = ""
    # calculate the time it took to receive the response
    response_time = time.time() - start_time

    #clsprompt_1 = response.choices[0].message.content
    #gr.Textbox.update(label=f"Question {response.choices[0].message.content}", value=f"Question {response.choices[0].message.content}")
    #text1.update(value=f"Question {response.choices[0].message.content}")

    # print the time delay and text received
    print(f"Full response from model {firstllm} received {response_time:.2f} seconds after request")
    #print(f"Full response received:\n{response}")

    returntext = response.choices[0].message.content + f" \nTime Taken: ({response_time:.2f} seconds)"

    #return response.choices[0].message.content + f" \nTime Taken: ({response_time:.2f} seconds)"
    return returntext
```

- Now create the UI
- We want a text box for system prompt
- we need a drop down with model to choose from, in the below example we have gpt-4-turbo, gpt-35-turbo and llama27b
- i haven't deployed llama27b is not deployed and tested

```
with gr.Blocks() as demo:
    #chatbot=gr.Chatbot(height=500)
    task_history = gr.State([])

    with gr.Row():
        with gr.Column(scale=1, min_width=600):
            text1 = gr.Textbox(label="prompt 1")
            inbtw = gr.Button("Submit")
        with gr.Column(scale=2, min_width=600):
            #file_output = gr.File()
            #addfile_btn = gr.UploadButton("Click to Upload a File", file_types=[".pdf",".csv",".docx"], file_count="multiple", render=False)
            #upload_button.upload(upload_file, upload_button, file_output)
            #addfile_btn.upload(upload_file, addfile_btn, file_output, show_progress=True) 
            upload_button = gr.UploadButton(render=False,file_types=[".pdf",".csv",".docx"])
            chatbot = gr.ChatInterface(predict, title="LLM Model", additional_inputs=[gr.Dropdown(
                ["gpt-4-turbo", "gpt-35-turbo", "llama27b"], label="firstllm"
            ),gr.Textbox("You are helpful AI.", label="System Prompt"),
            upload_button,], 
            )
            

if __name__ == "__main__":
    demo.launch()
```

- to run the code go to terminal
- Create a virtual environment
- Activate the virtual environment
- Run the code

```
python chatpdfui.py
```


![info](https://github.com/balakreshnan/Samples2024/blob/main/GradioUI/images/chatpdfui1.jpg 'RagChat')