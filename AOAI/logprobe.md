# Azure Open AI - showing probability of token's predicted

## Introduction

- Using Azure Open AI how to show logprobs
- Showing token predicted probability
- display for each word in the sentence

## Requirements

- Azure Open AI
- Azure Subscription
- Python
- Azure Machine Learning compute instance
- Azure open ai deployment in East US or North or South central US only (as of 2024-03-14)

## Steps

- install necessary libraries

```
%pip install openai
```

- now restart the kernel
- import the necessary libraries

```
import os
import openai

openai.api_type = "azure"
openai.api_base = "https://aoairesourcename.openai.azure.com/"
openai.api_version = "2024-02-01"
openai.api_key = "xxxxxx"
```

- make sure change the version as shown above "2024-02-01"
- next invoke the open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://aoairesroucename.openai.azure.com/", 
  api_key="xxxxx",  
  api_version="2024-02-01"
)
```

- Create a sample message to send

```
message_text = [{"role":"system","content":"You are an AI assistant that helps people find information."},
{"role": "user", "content": "what is the age of michael jordan"}]
```

- now call open ai chat completions and send the message

```
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

print(response.choices[0].message.content)
```

- the above will print only message

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/logprobs1.jpg 'RagChat')

- to print the logprobs

```
import numpy as np

for logprob in response.choices[0].logprobs.content:
    #print(logprob)
    print(f"Token or the word: {logprob.token} and the probability: {logprob.logprob} linear probability: {np.round(np.exp(logprob.logprob)*100,2)}%")
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/logprobs2.jpg 'RagChat')