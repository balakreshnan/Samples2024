# Yolov8 Object Detection Inferencing with Streamlit

## Introduction

- Create a streamlit app to show case object detection inferencing using Yolov8 model.
- Using image as input and show the output with bounding boxes.
- using existing model and weights for inferencing.
- There is no training involved in this code.
- Using a open source image available in public
- This is for educational purpose only.

## Pre-requisite

- Install streamlit
- python 3.12
- yolov8 model and weights
- open source image
- Local Laptop
- Python Virtual Environment

## Code

- Install libraries

```python
opencv-python
ultralytics
PyMuPDF
python-docx
matplotlib
Pillow
pyreadline3
huggingface_hub
onnxruntime
onnxruntime-genai
torch
torchvision
python-dotenv
PyPDF2
streamlit
```

- Load the libraries

```
from PIL import Image, ImageDraw
import matplotlib.pyplot as plt
import cv2
import torch
from pathlib import Path
import numpy as np
from datetime import timedelta
import json
import streamlit as st
from PIL import Image
from typing import Optional
from typing_extensions import Annotated
from streamlit import session_state as state
```

- Load the yolov8 model and weights

```
# Load a model
model = YOLO("yolov8n.pt")  # pretrained YOLOv8n model
```

- now define the function to run the object detection

```
def yoloinf():
    # global variables
    global model, confidence, cfg_model_path
    imgfile = "bus.jpg"
    image = Image.open(imgfile)
    #img = image

    st.write("Running YOLOv5 inference...")
    #detected_img, results = detect_objects1(image)

    #image = convert_rgb_to_rgba(image)
    print(f"Image mode: {image.mode}")

    # Run inference
    results = model(imgfile)  # return a list of Results objects
    img = None
    # Visualize the results
    for i, r in enumerate(results):
        # Plot results image
        im_bgr = r.plot()  # BGR-order numpy array
        im_rgb = Image.fromarray(im_bgr[..., ::-1])  # RGB-order PIL image
        img = im_rgb
            
    # Display the image with detected objects
    st.image(img, caption='Detected Objects', use_column_width=True)
```

- now the main function to run the streamlit app

```
def __main__():
    st.title("YOLOv8 Object Detection")
    st.write("This is a simple Streamlit app to run YOLOv8 object detection.")
    yoloinf()
```

- Display the output

![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/yolov8-1.jpg 'RagChat')

- Display Image with bounding boxes

![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/yolov8-2.jpg 'RagChat')

- Done