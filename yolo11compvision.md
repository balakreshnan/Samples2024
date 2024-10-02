# Yolov11, Azure Computer vision latest API Object Detection Inferencing with Streamlit

## Introduction

- Create a streamlit app to show case object detection inferencing using Yolov11 model.
- Also inference with Azure Computer Vision API.
- Using image as input and show the output with bounding boxes.
- using existing model and weights for inferencing.
- There is no training involved in this code.
- Using a open source image available in public
- This is for educational purpose only.

## Pre-requisite

- Install streamlit
- python 3.12
- yolov11 model and weights
- open source image
- Local Laptop
- Python Virtual Environment
- Azure Computer Vision API
- Azure Subscription
- Azure Computer Vision API Key

## Code

- Install libraries

```
import os
from openai import AzureOpenAI
import gradio as gr
from dotenv import dotenv_values
import time
from datetime import timedelta
import json
import streamlit as st
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
import PyPDF2
import docx
import fitz  # PyMuPDF
from PIL import Image, ImageDraw, ImageFont
import matplotlib.pyplot as plt
import cv2
import torch
from pathlib import Path
import numpy as np
from ultralytics import YOLO
```

- Load the configuration

```
config = dotenv_values("env.env")

css = """
.container {
    height: 75vh;
}
"""
```

- Load open ai config and other AI Search

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION"], 
  api_key=config["AZURE_OPENAI_KEY_VISION"],  
  api_version="2024-05-01-preview"
  #api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)

#model_name = "gpt-4-turbo"
#model_name = "gpt-35-turbo-16k"
model_name = "gpt-4o-g"

search_endpoint = config["AZURE_AI_SEARCH_ENDPOINT"]
search_key = config["AZURE_AI_SEARCH_KEY"]
search_index=config["AZURE_AI_SEARCH_INDEX1"]
SPEECH_KEY = config['SPEECH_KEY']
SPEECH_REGION = config['SPEECH_REGION']
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']

citationtxt = ""

#https://docs.ultralytics.com/usage/python/ - training
#https://docs.ultralytics.com/modes/predict/#plotting-results
#https://docs.ultralytics.com/modes/predict/#boxes

# Load a model
# model = YOLO("yolov8n.pt")  # pretrained YOLOv8n model
model = YOLO("yolo11n.pt")

# Set up your Computer Vision subscription key and endpoint
subscription_key = config["COMPUTER_VISION_KEY"]
endpoint = config["COMPUTER_VISION_ENDPOINT"]
```

- Extract the image from image byte array

```
def extractobjectsfromimage_image(imgfile, selected_optionmodel, user_input):
    returntxt = ""

    # Define the API version and URL for object detection
    # analyze_url = endpoint + "vision/v4.0/analyze"
    #analyze_url = endpoint + "/computervision/imageanalysis:analyze?api-version=2024-02-01&features=tags,read,caption,denseCaptions,smartCrops,objects,people"
    analyze_url = endpoint + "/computervision/imageanalysis:analyze?api-version=2023-04-01-preview&features=tags,read,caption,denseCaptions,smartCrops,objects,people"

    # Parameters for the request, specifying that we want to analyze objects
    params = {
        "visualFeatures": "Objects"
    }

    # Path to the local image
    image_path = imgfile
    # Convert the image to bytes
    #image_byte_array = io.BytesIO()
    #imgfile.save(image_byte_array, format='PNG')  # Save as PNG
    #image_bytes = image_byte_array.getvalue()
    # Convert the image to byte stream (JPEG)
    image_byte_arr = io.BytesIO()
    imgfile.save(image_byte_arr, format='JPEG')
    image_bytes = image_byte_arr.getvalue()

    # Headers for the API request, including the content type for binary data
    headers = {
        'Ocp-Apim-Subscription-Key': subscription_key,
        'Content-Type': 'application/octet-stream'
    }

    # Read the image as binary data
    #with open(image_path, "rb") as image_data:
    #    # Send the POST request to the Azure Computer Vision API
    #    response = requests.post(analyze_url, headers=headers, params=params, data=image_data)
    #    #response.raise_for_status()  # Raise exception if the call failed

    response = requests.post(analyze_url, headers=headers, params=params, data=image_bytes)

    # Parse the JSON response
    analysis = response.json()
    returntxt = json.dumps(analysis, indent=5)
    #returntxt = analysis
    #print(returntxt)

    return returntxt
```

- now function to draw boxes

```
# Function to draw bounding boxes  
def draw_text_bounding_boxes(draw, words):  
    for word in words:  
        box = word["boundingBox"]  
        content = word["content"]  
        # Draw lines between each point to form a box  
        draw.polygon([(box[0], box[1]), (box[2], box[3]), (box[4], box[5]), (box[6], box[7])], outline="green", width=2)  
        # Put text  
        draw.text((box[0], box[1] - 10), content, fill="green")
```

- Now the main function for UI

```
def yoloimage():

    col1, col2 = st.columns([1, 1])
    with col1:
        st.write("Upload a Image file")
        # Upload an image
        uploaded_file = st.file_uploader("Choose an image...", type=['jpg', 'jpeg', 'png'])

        if uploaded_file is not None:
            # Display the uploaded image
            image = Image.open(uploaded_file)
            st.image(image, caption='Uploaded Image', use_column_width=True)
    with col2:
        if st.button("Analyze"):
            # Call the function to extract objects from the image
            #returntxt = extractobjectsfromimage_image(uploaded_file, model_name, "Analyze the image")
            #st.write(returntxt)
            results = model(image)  # return a list of Results objects
            img = None
            # Visualize the results
            # Plot results image
            #im_bgr = image.plot()  # BGR-order numpy array
            #im_rgb = Image.fromarray(im_bgr[..., ::-1])  # RGB-order PIL image
            rgb_image = np.array(image)
            im_rgb = rgb_image[:, :, ::-1]
            img = im_rgb
                            
            # Display the image with detected objects
            st.write("### Objects detected in the image: YOLOv11")
            st.image(img, caption='Detected Objects', use_column_width=True)

            st.write("### Objects detected in the image: Azure Computer Vision API")
            imageinfo = extractobjectsfromimage_image(image, "gpt-4o", "")
            # st.json(imageinfo)
            outputjson = json.loads(imageinfo)
            # print('Output JSON:', outputjson)
            draw = ImageDraw.Draw(image)
            # Draw bounding boxes  
            for item in outputjson['denseCaptionsResult']['values']:  
                box = item['boundingBox']  
                text = item['text']  
                draw.rectangle([box['x'], box['y'], box['x'] + box['w'], box['y'] + box['h']], outline="red", width=2)  
                draw.text((box['x'], box['y']), text, fill="black")  

            # Parsing the bounding boxes and tags in 'objectsResult'
            for obj in outputjson['objectsResult']['values']:
                bounding_box = obj['boundingBox']
                x, y, w, h = bounding_box['x'], bounding_box['y'], bounding_box['w'], bounding_box['h']
                tag_info = obj['tags'][0]
                tag_name = tag_info['name']
                confidence = tag_info['confidence']

                # Draw the bounding box (outline)
                draw.rectangle([x, y, x + w, y + h], outline="red", width=3)

                # Label with the tag name and confidence score
                label = f"{tag_name} ({confidence:.2f})"
                
                # Optional: Load a font (system font or custom TTF file)
                try:
                    font = ImageFont.truetype("arial.ttf", 15)
                except IOError:
                    font = ImageFont.load_default()

                # Use textbbox to calculate text size (replacing textsize)
                text_bbox = draw.textbbox((x, y), label, font=font)
                text_width = text_bbox[2] - text_bbox[0]
                text_height = text_bbox[3] - text_bbox[1]

                # Draw the background rectangle for the label
                draw.rectangle([x, y - text_height, x + text_width, y], fill="red")

                # Draw the label text on the image
                draw.text((x, y - text_height), label, fill="white", font=font)
            words = outputjson["readResult"]["pages"][0]["words"] 
            draw_text_bounding_boxes(draw, words)
            st.image(image, caption="Image with bounding boxes", use_column_width=True)
```

- Create a main function to run the streamlit app

```
if __name__ == "__main__":
    st.set_page_config(page_title="Yolov11 Object Detection", page_icon=":camera:", layout="wide")
    st.markdown(f'<style>{css}</style>', unsafe_allow_html=True)
    st.title("Yolov11 Object Detection")
    yoloimage()
```

- Run the streamlit app

```
streamlit run yolo11compvision.py
```

- output 

![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/yolov11-1.jpg 'RagChat')

- Done