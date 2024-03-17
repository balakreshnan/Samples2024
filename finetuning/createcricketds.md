# Create New data set for Cricket tabular data

## Introduction

- Create a new dataset for Cricket tabular data
- From tabular open source data
- Data from tabular data

## Requirements

- Azure Account
- Azure Machine Learning Service
- Compute instance with GPU compute
- using Standard_NC48ads_A100_v4 (48 cores, 440 GB RAM, 128 GB disk)
- Using Hugging Face LLaMa 2 model

## Steps

- load dataset

```
import pandas as pd

df = pd.read_csv('IPL_data.csv')
pd.set_option('display.max_columns', None)
```

- load keys

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- invoke Azure Open AI

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://aoainamers.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxx",  
  api_version="2024-02-01"
)
```

- Create message function

```
def createmessage(row):
    # json_string = row.iloc[0].to_json()
    json_string = row.to_json()
    msg = f"""
    Cricket sport match information is provided on how the player performed.
    Information is provided as the header. Based on the information create 10 sentences with not more than 
    300 words.
    Be positive and polite. Do not provide negative content.
    Stick the data provided and do not create content outside the content provided
    Source data is provided in JSON format.

    Sources:
    {json_string}
    
    Create 10 questions and answer pairs based on the cricket match information provided.
    Create output as JSON format with each question as array.
    """
    return msg
```

- Generate data function

```
def generatecricket(row):
    # print(str(row))
    # json_string = row.iloc[0].to_json()
    # print('json' , json_string)
    msg = createmessage(row)

    message_text = [{"role":"system","content":"You are an AI assistant helps create content based on the data row sent as json."},
    {"role": "user", "content": msg}]
    response = client.chat.completions.create(
        model="gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        max_tokens=500,
        temperature=0,
        stop=None,
        seed=123,
        tools=None,
        logprobs=True,  # whether to return log probabilities of the output tokens or not. If true, returns the log probabilities of each output token returned in the content of message..
        top_logprobs=2,
    )
    return response
```

- function to write json

```
import json

def json_array_to_jsonl(json_array, output_file):
    with open(output_file, 'a') as f:
        for item in json_array:
            f.write(json.dumps(item) + '\n')
```

- ouput file name

```
# Output file path
output_file = "output.jsonl"
```

- Generate data

```
i = 0
for index, row in df20.iterrows():
    #print(str(row))
    response = generatecricket(row)
    print(str(index))
    try:
        json_array_to_jsonl(json.loads(response.choices[0].message.content.replace("```","").replace("json","")), output_file)
    except Exception as e:
        print(e)
```