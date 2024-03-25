# Azure Open AI GPT 4V create documentation for Screens

## Use Case

- Ability to create documentation for screens using GPT 4 Vision model
- Provide application UI as image and get the documentation for the same
- Use the GPT 4 Vision model to generate the documentation for the screens
- Help to build new documentation for new applications built to run business process

## Prerequisites

- Azure Subscription
- Azure machine learning service
- Azure Open AI Service
- Python 3.10 or greater
- Need GPT 4 Vision model
- Check the region where it is available
- I downloaded wisconsin DMV driver license form mv3001.pdf - https://wisconsindot.gov/documents/formdocs/mv3001.pdf
- Open the pdf and take the screenshot of the form and save as jpg image.

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/dmvform1.jpg 'RagChat')

- Now our goal to try to create documentation for the above image

## Steps/Code

- import libraries

```
import base64
import requests
```

- Load Environment variables

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Setup open ai configuration

```
import openai 

openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_KEY"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = config["AZURE_OPENAI_ENDPOINT"]
openai.base_url = config["AZURE_OPENAI_ENDPOINT"]
```

- Setup the open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2024-02-01"
)
```

- Function to open the image file and get base 64

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- Setup the image to load

```
image_path = "dmvform1.jpg"
```

- Convert to base64 format

```
image_base64 = encode_image(image_path)
```

- now create the open ai request

```
response = client.chat.completions.create(
  model="gpt-4-vision",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "Create details documentation with step by step instruction for the image provided. Only respond from information in the image."},
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

- here is the output

```
Step 1: Fill out the applicant name section with your first, middle, and last name.
Step 2: Provide your social security number.
Step 3: Enter your residence address, including street, apartment number (if applicable), city, state, and zip code. If your mailing address is different from your residence address, provide it in the designated section.
Step 4: Fill in your sex, race, eye color, hair color, weight, and height.
Step 5: If you have changed your name since your last license or ID card, provide your former name and the reason for the name change (marriage, divorce, or other).
Step 6: Answer the questions in the "ALL APPLICANTS" section by marking "YES" or "NO" as appropriate. These questions cover topics such as organ donation, address withholding, driving status with the Wisconsin Department of Motor Vehicles (WDMV), out-of-state convictions, possession of out-of-state driver's license/identification card, driving experience, and legal residence status.
Step 7: Read the certification statement and sign in the space provided.
Step 8: The "OFFICE USE ONLY" section is to be left blank, as it will be completed by the appropriate personnel at the Wisconsin Department of Transportation.
Step 9: The "VISION" section is to be completed by a DMV examiner, including details such as visual acuity with and without prescription, temporal field of vision, color perception
```

- Continue this for other forms and screens
- Amplify your documentation with GPT 4 Vision model
- Increase productivity and reduce time to create documentation