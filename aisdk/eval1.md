# Evaluating chat messages for gorundedness, coherence, and Fluency, hate_unfairness, violence, self_harm, sexual

## Introduction

- Evaluate Gen AI applications chat style
- Evaluate the chat messages for groundedness, coherence, and Fluency, hate_unfairness, violence, self_harm, sexual
- Using Azure AI SDK
- Using Azure AI Studio
- Using Microsoft Azure

## Pre-requisite

- Azure account
- Azure AI Studio
- Azure AI SDK
- Visual Studio Code
- Python 3.11

## Steps

### Code

### Creating data set

- Create a chat message data as jsonl file
- It has have user, assistant type array of messages
- Here are few things i struggled with the data set
- Given for me i created the data from prompt flow chat history.
- Better way is to use the format used in azure open ai input prompt.
- 1) get rid of " in front of message value
- 2) convert all the single quote to double quotes
- 3) get rid of \\n, \n, \\\n, \n1, \n2, \n3
- 4) there were double quotes in the content which manually changed to single quote
- 5) used a online validator and it passed the JSONL format
- 6) Converted "User" to "user"
- 7) remove system messages
- 8) replace " inside the content with single quote
- 9) didn't add context to the messages
- Make sure the jsonl file is in right format, other wise there will be lots of errors
- Now once you have the data set then move to next
- Sample data set

```
{"messages":[{"role": "user", "content": "Top 5 candidate on hacking"}, {"role": "assistant", "content": "Based on the provided documents, the top candidates with experience related to security, which could include hacking, are:. Paolo Dal Cin - Accenture Security Global Lead with over 20 years of experience in cyber security strategy, business resilience, cyber defense, cloud protection, incident response, and managed security services Source: Profile 12.. Candidate from Profile 66 - SVP, Security & Government Group at Cisco, and previous experience as SVP & GM End user Computing at VMware and SVP, RSA Products at RSA, The Security Division of EMC Source: Profile 66."}]}
```

### Code Section

- Create a new python file and add the below code
- i am using visual studio code jupyter notebook
- import libraries

```
from azure.ai.generative.evaluate import evaluate
from azure.ai.resources.client import AIClient
from azure.ai.resources.entities import Data
from azure.ai.resources.constants import AssetTypes
from azure.identity import DefaultAzureCredential
from azure.ai.resources.client import AIClient 
from azure.identity import DefaultAzureCredential
from azure.ai.resources.entities import AzureOpenAIModelConfiguration
import os
```

- Log into Azure AI Studio

```
subscription = "xxxxxxxxxxxxxxxxxxxxxxxxx" # Subscription of your AI Studio project
resource_group = "rg-name" # Resource Group of your AI Studio project
project = "aihub name" #Name of your Ai Studio Project

client = AIClient(
    subscription_id=subscription, 
    resource_group_name=resource_group, 
    project_name=project, 
    credential=DefaultAzureCredential())
```

- setup the data set path and also open ai connection

```
tracking_uri = client.tracking_uri
connection = client.connections.get("connectionname")
config = AzureOpenAIModelConfiguration.from_connection(connection, model_name="gpt-4", deployment_name="gpt-4-turbo")
data_path = os.getcwd() + "\\chat_history2.jsonl"
output_path = os.getcwd() + "\\downloaded_artifacts\\remote"

print(data_path)
print(output_path)
```

- now run the evaluation

```
result = evaluate(
    evaluation_name="profiles-chat-eval-with-data",
    data=data_path,
    task_type="chat",
    metrics_list=[ "gpt_coherence", "gpt_fluency", "gpt_groundedness","hate_unfairness", "violence", "self_harm", "sexual"], #"hate_unfairness", "violence", "self_harm", "sexual",
    model_config=config,
    data_mapping={"messages": "messages"},
    tracking_uri=tracking_uri,
    output_path=output_path,
)
print(result)
print(result.metrics_summary)  # will print the defect rate for each content harm
print("Studio URL")
print(result.studio_url)
```

- Now run the code and check the output

```
{'gpt_groundedness': nan, 'gpt_coherence': 3.67, 'gpt_fluency': 3.67, 'sexual_defect_rate': 0.0, 'hate_unfairness_defect_rate': 0.0, 'violence_defect_rate': 0.0, 'self_harm_defect_rate': 0.0}
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/aisdk/images/eval1.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/aisdk/images/eval2.jpg 'RagChat')