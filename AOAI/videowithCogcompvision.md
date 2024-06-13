# Summarize a video using Azure Open AI GPT 4 Vision and Computer vision models

## Pre-requisites

- Azure Subscription
- Azure Open AI
- Azure Machine learning
- Azure Cognitive Computer vision or AI Vision latest
- GPT 4 turbo vision preview was available in west us in us region

## Introduction

- Summarize a video using Azure Open AI GPT 4 Vision and Computer vision models
- First we need video in Blob storage
- Create a SAS URL and store the SAS URL in a variable
- Only works with GPT-4-turbo-vision-preview model
- Model was available in west us in us region at the time of writing the article
- Need to first create a video index using florence model or computer vision model 4.0
- Video used - https://youtu.be/yjtxPmHdl54
- [![Watch the video](https://img.youtube.com/vi/yjtxPmHdl54/default.jpg)](https://youtu.be/yjtxPmHdl54)

## Steps

- install libraries

```
%pip install --upgrade openai
```

- import libraries

```
import openai 
import time
import requests
import json
```

- load config variables

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure open ai configuration

```
openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_KEY_VISION_PREVIEW"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT_VISION_PREVIEW"]
openai.api_version = "2024-02-01"
openai.base_url = config["AZURE_OPENAI_ENDPOINT_VISION_PREVIEW"]
```

- Now the client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION_PREVIEW"], 
  api_key=config["AZURE_OPENAI_KEY_VISION_PREVIEW"],  
  api_version="2024-02-01"
)
```

- set the deployment name

```
deployment_name = "gpt-4-vision"
```

- set the computer vision variables

```
computervisionendpoint = config["AZURE_COMPUTER_VISION_ENDPOINT"]
computervisionkey = config["AZURE_COMPUTER_VISION_KEY"]
```

- configure open ai response client

```
api_base = config["AZURE_OPENAI_ENDPOINT_VISION_PREVIEW"] # your endpoint should look like the following https://YOUR_RESOURCE_NAME.openai.azure.com/
api_key = config["AZURE_OPENAI_KEY_VISION_PREVIEW"]
deployment_name = "gpt-4-vision"
api_version = '2023-12-01-preview' # this might change in the future
videourl = ["https://blobname.blob.core.windows.net/aivideos/csifactory-business.mp4?sp=r&st=xxxxxxxxxxxxxxxxxxx"]
videourlstr = "https://blobname.blob.core.windows.net/aivideos/csifactory-business.mp4?sp=r&st=xxxxxxxxxxxxxxxxxxxxxxxxx"
```

- function to create video index

```
def create_video_index(vision_api_endpoint: str, vision_api_key: str, index_name: str) -> object:
    url = f"{vision_api_endpoint}/computervision/retrieval/indexes/{index_name}?api-version=2023-05-01-preview"
    headers = {"Ocp-Apim-Subscription-Key": vision_api_key, "Content-Type": "application/json"}
    data = {"features": [{"name": "vision", "domain": "surveillance"}, {"name": "speech"}]}
    return requests.put(url, headers=headers, data=json.dumps(data))
```

- function to add video to index

```
def add_video_to_index(
    vision_api_endpoint: str, vision_api_key: str, index_name: str, video_url: str, video_id: str
) -> object:
    url = (
        f"{vision_api_endpoint}/computervision/retrieval/indexes/{index_name}"
        f"/ingestions/my-ingestion?api-version=2023-05-01-preview"
    )
    headers = {"Ocp-Apim-Subscription-Key": vision_api_key, "Content-Type": "application/json"}
    data = {
        "videos": [{"mode": "add", "documentId": video_id, "documentUrl": video_url}],
        "generateInsightIntervals": False,
        "moderation": False,
        "filterDefectedFrames": False,
        "includeSpeechTranscrpt": True,
    }
    return requests.put(url, headers=headers, data=json.dumps(data))
```

- function to wait for the video index to be ready

```
def wait_for_ingestion_completion(
    vision_api_endpoint: str, vision_api_key: str, index_name: str, max_retries: int = 30
) -> bool:
    url = (
        f"{vision_api_endpoint}/computervision/retrieval/indexes/{index_name}/ingestions?api-version=2023-05-01-preview"
    )
    headers = {"Ocp-Apim-Subscription-Key": vision_api_key}
    retries = 0
    while retries < max_retries:
        time.sleep(10)
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            state_data = response.json()
            if state_data["value"][0]["state"] == "Completed":
                print(state_data)
                print("Ingestion completed.")
                return True
            if state_data["value"][0]["state"] == "Failed":
                print(state_data)
                print("Ingestion failed.")
                return False
        retries += 1
    return False
```

- Process the Video index

```
def process_video_indexing(
    vision_api_endpoint: str, vision_api_key: str, video_index_name: str, video_SAS_url: str, video_id: str
) -> None:
    # Step 1: Create an Index
    response = create_video_index(vision_api_endpoint, vision_api_key, video_index_name)
    print(response.status_code, response.text)

    # Step 2: Add a video file to the index
    response = add_video_to_index(vision_api_endpoint, vision_api_key, video_index_name, video_SAS_url, video_id)
    print(response.status_code, response.text)

    # Step 3: Wait for ingestion to complete
    if not wait_for_ingestion_completion(vision_api_endpoint, vision_api_key, video_index_name):
        print("Ingestion did not complete within the expected time.")
```

- process the video index first
- Below will create a new index and add the video to the index

```
process_video_indexing(computervisionendpoint,computervisionkey, "csivideoindex",videourlstr,"csifactory-business")
```

- Now set the message to summarize the video

```
messages=[
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": [  
        {
            "type": "acv_document_id",
            "acv_document_id": "csifactory-business"
        },
        { 
            "type": "text", 
            "text": "Describe this video:" 
        }
    ] } 
]
```

- now call the open ai client

```
# Construct the API request URL
api_url = (
    f"{api_base}/openai/deployments/{deployment_name}"
    f"/extensions/chat/completions?api-version={api_version}"
)

# Including the api-key in HTTP headers
headers = {
    "Content-Type": "application/json",
    "api-key": api_key,
    "x-ms-useragent": "Azure-GPT-4V-video/1.0.0",
}

# Payload for the request
payload = {
    "model": "gpt-4-vision",
    "dataSources": [
        {
            "type": "AzureComputerVisionVideoIndex",
            "parameters": {
                "computerVisionBaseUrl": f"{computervisionendpoint}/computervision",
                "computerVisionApiKey": computervisionkey,
                "indexName": "csivideoindex",
                "videoUrls": videourl,
            },
        }
    ],
    "enhancements": {"video": {"enabled": True}},
    "messages": messages,
    "temperature": 0.7,
    "top_p": 0.95,
    "max_tokens": 800,
}

# Send the request and handle the response
try:
    response = requests.post(api_url, headers=headers, json=payload)
    response.raise_for_status()  # Raise an error for bad HTTP status codes
    print(response.json())
    #print(response.choices[0].message.content)
except requests.RequestException as e:
    print(f"Failed to make the request. Error: {e}")
```

- Output summarize content

```
obj = response.json()
print(obj['choices'][0]['message']['content'])
```

- Output

```
The video appears to be a virtual tour of an automated manufacturing facility, showcasing various stages of the production process. It begins with an introduction to the conveyor system, describing it as the lifeline of operations that ensures smooth flow of production. The tour then highlights collaborative robots working alongside human operators, assisting in tasks to perfect the product assembly.

Quality assurance is emphasized at inspection stations, where products are thoroughly checked for any imperfections, with faulty items being sent for repair. The video features capping robots that secure the final seals on products, and filling stations for both liquids and dry goods, where products are carefully measured and prepared.

The tour concludes at the packaging station, where finished products are picked, placed, and packaged with care, ready for distribution. Throughout the facility, the integration of technology and human skill is noted, contributing to efficiency, consistency, and quality in the production process. The video closes with a thank you to the viewers for joining the tour and a statement about the dedication and innovation that go into every step of manufacturing.
```