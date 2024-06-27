# Consume Azure Custom vision Prediction API

## Introduction

- customvision.ai
- Azure Custom Vision Prediction API
- Train in Azure Custom vision Training
- Use the prediction API to predict the image
- Loop images

## Pre-requisites

- Azure Subscription
- Azure Custom Vision
- Azure Custom Vision Prediction API
- images
- Do bounding box
- Train the model
- Publish the model

## Steps

- Create a new project in Azure Custom Vision
- Upload the images
- Train the model
- Publish the model
- Get the prediction key
- Get the endpoint
- Use the prediction API
- Loop through the images

- configure the endpoint and the key
- Make sure use the image endpoint

```
import os
import requests

# URL of the custom vision prediction endpoint
endpoint_url = 'https://airesource.cognitiveservices.azure.com/customvision/v3.0/Prediction/xxxxxxxxxxx/detect/iterations/Iteration1/image'
      
# Headers for the request
headers = {
    "Prediction-Key": "xxxxxxx",
    "Content-Type": "application/octet-stream"
}
```

- loop the images and process

```
folder_path = "images"
for filename in os.listdir(folder_path):
    file_path = os.path.join(folder_path, filename)
    if os.path.isfile(file_path):  
         # Open the image file in binary mode
        with open(file_path, "rb") as image_file:
            # Body of the request is the image file in binary
            response = requests.post(endpoint_url, headers=headers, data=image_file)
            #print(response.json())
            result = response.json()
            #for pred in result['predictions']:
            #    #print(pred['tagId'], pred['tagName'], pred['boundingBox'], pred['probability'], '\n')
            #    rs = f"Tag ID: {pred['tagId']} - Tag ID: {pred['tagName']} - probability: {pred['probability']} - boundingBox: {pred['boundingBox']}"
            #    print(rs)
```

- done