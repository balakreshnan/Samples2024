# Identifying Bath, Rest and Kitchen in Architectural document

## Identify pages to look for details analysis for Tiles

## Introduction

- Process large PDF file to identify pages with Bath, Rest and Kitchen
- These files are Architectural documents of buildings, apartments, etc.
- The goal is to identify the pages with details of Bath, Rest and Kitchen
- Then find square footage for tiles section

## Requirements

- Python
- Azure Machine Learning Service
- Azure Open AI GPT 4 vision
- PyPDF2, PyMuPDF

## Steps - Code

- First install libraries needed

```
%pip install PyPDF2
%pip install PyMuPDF
```

```
%pip install openai
%pip install python-dotenv
```

- Load environment variables for keys

```
from dotenv import load_dotenv
config = dotenv_values("env.env")
```

- Configure open ai

```
import openai 

openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_KEY"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = config["AZURE_OPENAI_ENDPOINT"]
openai.base_url = config["AZURE_OPENAI_ENDPOINT"]
```

- Setup open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2024-02-01"
)
```

- import image libraries

```
import base64
import requests

import fitz
from PIL import Image
```

- Convert image to base64

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- function to process image and extract information
- System prompt
- now here is the prompt

```
imgprompt = """You are a Construction Architect Agent. Analyze the diagram and find details for questions asked.
Only answer from the data source provided.
Extract the squre footage of where to Bath room, rest room, Shower, toilet, Kitchen 
from the architectural diagram provided?. 
"""
```

```
def processimage(base64_image):
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
    max_tokens=300,
    )

    #print(response.choices[0].message.content)
    return response.choices[0].message.content
```

- Setup image folder

```
image_folder = "constimages"
```

- Now read the 100's pages pdf and only process with bath room, rest room and kitchen
- We wanted to filter only pages to process instead all pages to save cost.
- The above process will also get rid of unwanted pages


```
import fitz

text = ''

def extract_images_from_pdf(pdf_path, image_folder):
    # Open the PDF file
    pdf_document = fitz.open(pdf_path)

    # Iterate through each page
    for page_number in range(len(pdf_document)):
        # Get the page
        page = pdf_document.load_page(page_number)

        text1 = page.get_text().lower()
        if "bath" in text1 or "kitchen" in text1 or "restroom" in text1:

            # Convert the page to an image
            pix = page.get_pixmap()

            # Save the image
            image_path = f"{image_folder}/page_{page_number + 1}.png"
            pix.save(image_path)
            base64_image = encode_image(image_path)
            print(f"Page {page_number + 1} saved as {image_path}")
            rttext = processimage(base64_image)
            print(f"Information: {rttext} \n")

    # Close the PDF file
    pdf_document.close()
```

- Now process the pdf file

```
pdf_path = pdf_file_path
image_folder = image_folder

extract_images_from_pdf(pdf_path, image_folder)
```

- Wait for the process to complete.
- Output will be only pages that has information about Bath, Rest and Kitchen and it's square foot.
- But will also says which pages has no information about these so that human can review and add to the process.