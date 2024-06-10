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
%pip install promptflow
%pip install promptflow-evals
%pip install promptflow-tracing
```

- Import the libraries

```
import os
from typing import Any, Dict, List
from dotenv import load_dotenv
from dotenv import dotenv_values
#load_dotenv()
import uuid

from promptflow.core import Prompty, AzureOpenAIModelConfiguration
from promptflow.evals.evaluators import ChatEvaluator
from promptflow.evals.evaluators import RelevanceEvaluator, CoherenceEvaluator
from promptflow.evals.evaluate import evaluate
from askwiki import ask_wiki
from promptflow.evals.evaluators import ViolenceEvaluator
from promptflow.evals.evaluators import RelevanceEvaluator
from promptflow.evals.synthetic import AdversarialSimulator
from promptflow.evals.synthetic import AdversarialScenario
from azure.identity import DefaultAzureCredential
```

- Enable tracing

```
from promptflow.tracing import start_trace
start_trace()
```

- Load the environment

```
config = dotenv_values("env.env")
```

- Configure the model

```
model_config = AzureOpenAIModelConfiguration(
    azure_deployment=config["AZURE_OPENAI_DEPLOYMENT_NAME"],
    api_version=config["AZURE_OPENAI_DEPLOYMENT_VERSION"],
    azure_endpoint=config["AZURE_OPENAI_ENDPOINT_VISION"],
    api_key=config["AZURE_OPENAI_KEY_VISION"]
)
```

- COnfigure ai project

```
azure_ai_project={
        "subscription_id": "80ef7369-572a-4abd-b09a-033367f44858",
        "resource_group_name": "rg-babalai",
        "project_name": "babal-sweden",
    }
```

- Create the main function
- we are comparing 3 models
- gpt-4o-g, gpt-4o, gpt-4-turbo
- Accumulate the results

```
def main():

    results = []

    model_config = AzureOpenAIModelConfiguration(
        azure_deployment=config["AZURE_OPENAI_DEPLOYMENT_NAME"],
        api_version=config["AZURE_OPENAI_DEPLOYMENT_VERSION"],
        azure_endpoint=config["AZURE_OPENAI_ENDPOINT_VISION"],
        api_key=config["AZURE_OPENAI_KEY_VISION"]
    )
    chat_eval = ChatEvaluator(model_config=model_config)

    chat_history=[
        {"role": "user", "content": "Does Azure OpenAI support customer managed keys?"},
        {"role": "assistant", "content": "Yes, customer managed keys are supported by Azure OpenAI."}
    ]
    chat_input="Do other Azure AI services support this too?"

    prompty = Prompty.load("chat.prompty", model={'configuration': model_config})
    response = prompty(chat_history=chat_history, chat_input=chat_input)

    print(response)

    relevance_evaluator = RelevanceEvaluator(model_config=model_config)
    coherence_evaluator = CoherenceEvaluator(model_config=model_config)
    path = "profiles.jsonl"
    result = evaluate(
        data=path,
        evaluators={
            "coherence": coherence_evaluator,
            "relevance": relevance_evaluator,
        },
        evaluator_config={
            "coherence": {
                "answer": "${data.answer}",
                "question": "${data.question}"
            },
            "relevance": {
                "answer": "${data.answer}",
                "context": "${data.context}",
                "question": "${data.question}"
            }
        },
        azure_ai_project=azure_ai_project,
        #output_path="./evalresults/evaluation_results.json",
        evaluation_name="chat_eval_gpt4og_" + str(uuid.uuid4())[:8],
    )
    # print(result)
    results.append("{ 'model' : 'gpt-4o-g', 'metrics' : " + str(result['metrics']) + "," + "'studio_url' : '" + str(result['studio_url']) + "' }")

    ## Now gpt 4o model

    model_config = AzureOpenAIModelConfiguration(
        azure_deployment="gpt-4o",
        api_version=config["AZURE_OPENAI_DEPLOYMENT_VERSION"],
        azure_endpoint=config["AZURE_OPENAI_ENDPOINT_VISION"],
        api_key=config["AZURE_OPENAI_KEY_VISION"]
    )

    relevance_evaluator = RelevanceEvaluator(model_config=model_config)
    coherence_evaluator = CoherenceEvaluator(model_config=model_config)
    path = "profiles.jsonl"
    result = evaluate(
        data=path,
        evaluators={
            "coherence": coherence_evaluator,
            "relevance": relevance_evaluator,
        },
        evaluator_config={
            "coherence": {
                "answer": "${data.answer}",
                "question": "${data.question}"
            },
            "relevance": {
                "answer": "${data.answer}",
                "context": "${data.context}",
                "question": "${data.question}"
            }
        },
        azure_ai_project=azure_ai_project,
        #output_path="./evalresults/evaluation_results.json",
        evaluation_name="chat_eval_gpt4o_" + str(uuid.uuid4())[:8],
    )
    results.append("{ 'model' : 'gpt-4o', 'metrics' : " + str(result['metrics']) + "," + "'studio_url' : '" + str(result['studio_url']) + "' }")

    ## Now gpt 4 turbo

    model_config = AzureOpenAIModelConfiguration(
        azure_deployment="gpt-4-turbo",
        api_version=config["AZURE_OPENAI_DEPLOYMENT_VERSION"],
        azure_endpoint=config["AZURE_OPENAI_ENDPOINT_VISION"],
        api_key=config["AZURE_OPENAI_KEY_VISION"]
    )

    relevance_evaluator = RelevanceEvaluator(model_config=model_config)
    coherence_evaluator = CoherenceEvaluator(model_config=model_config)
    path = "profiles.jsonl"
    result = evaluate(
        data=path,
        evaluators={
            "coherence": coherence_evaluator,
            "relevance": relevance_evaluator,
        },
        evaluator_config={
            "coherence": {
                "answer": "${data.answer}",
                "question": "${data.question}"
            },
            "relevance": {
                "answer": "${data.answer}",
                "context": "${data.context}",
                "question": "${data.question}"
            }
        },
        azure_ai_project=azure_ai_project,
        #output_path="./evalresults/evaluation_results.json",
        evaluation_name="chat_eval_gpt-4-turbo_" + str(uuid.uuid4())[:8],
    )
    results.append("{ 'model' : 'gpt-4-turbo', 'metrics' : " + str(result['metrics']) + "," + "'studio_url' : '" + str(result['studio_url']) + "' }")

    print(results)
```

- Run the main function

```
if __name__ == "__main__":
    main()
```
