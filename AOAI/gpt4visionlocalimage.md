# Azure Machine Learning - GPT 4 Vision

## Requirements

- Azure Machine Learning workspace
- Azure Storage account
- Azure Machine learning compute instance
- Install open ai v1.0 and above
- gpt 4 vision was only available in west us
- i deployed the mode as gpt-4-vision (deployment name)

## Code

- Initiate open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://aoairesource.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxxxxxxxxxx",  
  api_version="2023-09-01-preview"
)
```

- now load the image

```
import base64
import requests
```

- now set the image path

```
image_path = "visualize-error-bars-1.png"
```

- Read image from local file system

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- convert the image to base64

```
base64_image = encode_image(image_path)
```

- call open wi gpt 4 vision with image and ask question

```
response = client.chat.completions.create(
  model="gpt-4-vision",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "what's in the image?"},
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

print(response.choices[0].message.content)
```

- output

```
A plot showing the median household income for counties in Maine, based on estimates from the 2016-2020 American Community Survey (ACS).
```

- now try different question

```
question = "can you create insights?"
```

- call open wi gpt 4 vision with image and ask question

```
response = client.chat.completions.create(
  model="gpt-4-vision",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": question},
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

print(response.choices[0].message.content)
```

- output

```
1. Cumberland County has the highest median household income among the counties in Maine, followed by York and Sagadahoc.
2. Piscataquis County has the lowest median household income, followed by Aroostook and Washington.
3. The median household income in Maine ranges from around $40,000 to over $70,000.
4. The majority of the counties have a median household income between $50,000 and $60,000.
5. There is a significant gap between the highest and lowest-income counties in Maine, with a difference of about $30,000 between Cumberland and Piscataquis counties.
```