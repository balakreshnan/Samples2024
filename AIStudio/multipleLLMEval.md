# Evalulate Use Case against multiple LLM Models - Pick the best model for your use case

## Private Leaderboard for Use Case specific models

## Introduction

- How can we find which LLM model is best for our use case?
- evaluate our own use case with multiple LLM models
- Evaluate against groundtruth data

## Pre-requisites

- Azure Subscription
- Azure AI Studio
- Azure Open AI
- Azure AI Search
- Visual studio Code
- Azure AI SDK

## Steps

- Create a new project in Azure AI Studio
- Install AI Sdk in Visual Studio Code
- Create a local environment
- Create a new notebook
- Install Azure AI SDK

```
%pip install --upgrade azure-ai-generative[index,evaluate,promptflow]
%pip install promptflow-tracing
```

- Now import the libraries

```
from azure.ai.generative.evaluate import evaluate
import os
from azure.ai.resources.client import AIClient
from azure.ai.resources.entities import Data
from azure.ai.resources.constants import AssetTypes
from azure.identity import DefaultAzureCredential
from azure.ai.resources.entities import AzureOpenAIModelConfiguration
```

- create the results list to store output

```
results = []
```

- Now create a new client

```
from azure.ai.resources.client import AIClient 
from azure.identity import DefaultAzureCredential

subscription = "xxxxxxxxxxxxxxxxxxxxxxxxxxx" # Subscription of your AI Studio project
resource_group = "rg-name" # Resource Group of your AI Studio project
project = "aiprojectname" #Name of your Ai Studio Project

client = AIClient(
    subscription_id=subscription, 
    resource_group_name=resource_group, 
    project_name=project, 
    credential=DefaultAzureCredential())
```

- Now configure the data from local file
- Set the output path
- Set the model deployment as gpt 4 turbo to use.

```
tracking_uri = client.tracking_uri
connection = client.connections.get("connectionname")
config = AzureOpenAIModelConfiguration.from_connection(connection, model_name="gpt-4", deployment_name="gpt-4-turbo")
data_path = os.getcwd() + "\\chat_history2.jsonl"
output_path = os.getcwd() + "\\downloaded_artifacts\\remote"

print(data_path)
print(output_path)
```

- now run the first evaluation

```
result = evaluate(
    evaluation_name="profileseval-vscode",
    #target="LinkedInProfile-sample-flow",
    data=data_path,
    task_type="chat",
    metrics_list=[ "gpt_coherence", "gpt_fluency", "gpt_groundedness", "gpt_retrieval_score","hate_unfairness", "violence", "self_harm", "sexual"], #"hate_unfairness", "violence", "self_harm", "sexual",
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

- Save the results

```
results.append("{ 'model' : 'gpt-4-turbo', 'metrics' : " + str(result.metrics_summary) + "," + "'studio_url' : '" + result.studio_url + "' }")
```

- Set the model deployment as gpt 4o global endpoint to use.

```
connection = client.connections.get("aoaitestbb1")
config = AzureOpenAIModelConfiguration.from_connection(connection, model_name="gpt-4o", deployment_name="gpt-4o-g")
```

- now run the first evaluation

```
result = evaluate(
    evaluation_name="profileseval-vscode",
    #target="LinkedInProfile-sample-flow",
    data=data_path,
    task_type="chat",
    metrics_list=[ "gpt_coherence", "gpt_fluency", "gpt_groundedness", "gpt_retrieval_score","hate_unfairness", "violence", "self_harm", "sexual"], #"hate_unfairness", "violence", "self_harm", "sexual",
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

- Save the results

```
results.append("{ 'model' : 'gpt-4o-g', 'metrics' : " + str(result.metrics_summary) + "," + "'studio_url' : '" + result.studio_url + "' }")
```

- Set the model deployment as gpt 4o endpoint to use.

```
connection = client.connections.get("aoaitestbb1")
config = AzureOpenAIModelConfiguration.from_connection(connection, model_name="gpt-4o", deployment_name="gpt-4o")
```

- now run the first evaluation

```
result = evaluate(
    evaluation_name="profileseval-vscode",
    #target="LinkedInProfile-sample-flow",
    data=data_path,
    task_type="chat",
    metrics_list=[ "gpt_coherence", "gpt_fluency", "gpt_groundedness", "gpt_retrieval_score","hate_unfairness", "violence", "self_harm", "sexual"], #"hate_unfairness", "violence", "self_harm", "sexual",
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

- Save the results

```
results.append("{ 'model' : 'gpt-4o', 'metrics' : " + str(result.metrics_summary) + "," + "'studio_url' : '" + result.studio_url + "' }")
```

- Set the model deployment as gpt 35 turbo16k endpoint to use.

```
connection = client.connections.get("aoaitestbb1")
config = AzureOpenAIModelConfiguration.from_connection(connection, model_name="gpt-35-turbo-16k", deployment_name="gpt-35-turbo-16k")
```

- now run the first evaluation

```
result = evaluate(
    evaluation_name="profileseval-vscode",
    #target="LinkedInProfile-sample-flow",
    data=data_path,
    task_type="chat",
    metrics_list=[ "gpt_coherence", "gpt_fluency", "gpt_groundedness", "gpt_retrieval_score","hate_unfairness", "violence", "self_harm", "sexual"], #"hate_unfairness", "violence", "self_harm", "sexual",
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

- Save the results

```
results.append("{ 'model' : 'gpt-35-turbo-16k', 'metrics' : " + str(result.metrics_summary) + "," + "'studio_url' : '" + result.studio_url + "' }")
```

- now print all the results to pick the best model

```
for row in results:
    print(row)
    print("\n")
```

- Output

```
{ 'model' : 'gpt-4-turbo', 'metrics' : {'gpt_retrieval_score': nan, 'gpt_groundedness': nan, 'gpt_fluency': 4.0, 'gpt_coherence': 4.0, 'sexual_defect_rate': 0.0, 'violence_defect_rate': 0.0, 'self_harm_defect_rate': 0.0, 'hate_unfairness_defect_rate': 0.0},'studio_url' : 'https://ai.azure.com/build/evaluation/xxxxxx?wsid=/subscriptions/xxxxxxx/resourceGroups/rgname/providers/Microsoft.MachineLearningServices/workspaces/wkspace' }


{ 'model' : 'gpt-4o-g', 'metrics' : {'gpt_groundedness': nan, 'gpt_retrieval_score': nan, 'gpt_coherence': nan, 'gpt_fluency': nan, 'hate_unfairness_defect_rate': 0.0, 'self_harm_defect_rate': 0.0, 'violence_defect_rate': 0.0, 'sexual_defect_rate': 0.0},'studio_url' : 'https://ai.azure.com/build/evaluation/xxxxxx?wsid=/subscriptions/xxxxxxx/resourceGroups/rgname/providers/Microsoft.MachineLearningServices/workspaces/wkspace' }


{ 'model' : 'gpt-4o', 'metrics' : {'gpt_retrieval_score': nan, 'gpt_groundedness': nan, 'gpt_fluency': nan, 'gpt_coherence': nan, 'hate_unfairness_defect_rate': 0.0, 'sexual_defect_rate': 0.0, 'violence_defect_rate': 0.0, 'self_harm_defect_rate': 0.0},'studio_url' : 'https://ai.azure.com/build/evaluation/xxxxxx?wsid=/subscriptions/xxxxxxx/resourceGroups/rgname/providers/Microsoft.MachineLearningServices/workspaces/wkspace' }


{ 'model' : 'gpt-35-turbo-16k', 'metrics' : {'gpt_retrieval_score': nan, 'gpt_groundedness': nan, 'gpt_fluency': 2.0, 'gpt_coherence': 3.0, 'violence_defect_rate': 0.0, 'hate_unfairness_defect_rate': 0.0, 'self_harm_defect_rate': 0.0, 'sexual_defect_rate': 0.0},'studio_url' : 'https://ai.azure.com/build/evaluation/xxxxxx?wsid=/subscriptions/xxxxxxx/resourceGroups/rgname/providers/Microsoft.MachineLearningServices/workspaces/wkspace' }

```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/multiplellmeval1.jpg 'RagChat')