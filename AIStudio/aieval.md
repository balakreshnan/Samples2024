# Azure AI Evaluation - Integrate to existing application

## Introduction

- How to evaluate gen ai application
- Using Azure AI Studio to manage the lifecycle of Gen AI application
- Azure AI Studio will be the UI to monitor and manage metrics
- SDK based approach
- Easy to integrate with existing applications
- Using a Talk to your data in Azure open ai for RAG based application

## Pre-requisites

- Azure Subscription
- Azure AI Studio
- Local Laptop for Development Environment
- Python 3.12
- Create a python environment
- Make sure the .venv is not the int folder where application code like .py files are stored
- if in the same folder, there will errors for file locked when pushing data to Azure AI Studio
- Pick the region based on service availability for evaluation.
- Here is the repo that i am using - https://github.com/balakreshnan/aistudioeval

### Steps

- First create the file for RAG based retrieval
- Create a file called ragdata.py
- here is the code

```
import os
from openai import AzureOpenAI
from dotenv import load_dotenv
import time
from datetime import timedelta
import json
from PIL import Image
import base64
import requests
import io
from typing import Optional
from typing_extensions import Annotated
import wave
from PIL import Image, ImageDraw
from pathlib import Path
import numpy as np


# Load .env file
load_dotenv()

css = """
.container {
    height: 75vh;
}
"""

client = AzureOpenAI(
  azure_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT"), 
  api_key=os.getenv("AZURE_OPENAI_API_KEY"),  
  api_version="2024-05-01-preview",
)

model_name = os.getenv("AZURE_OPENAI_DEPLOYMENT")

search_endpoint = os.getenv("AZURE_AI_SEARCH_ENDPOINT")
search_key = os.getenv("AZURE_AI_SEARCH_KEY")
search_index=os.getenv("AZURE_AI_SEARCH_INDEX")
SPEECH_KEY = os.getenv('SPEECH_KEY')
SPEECH_REGION = os.getenv('SPEECH_REGION')
SPEECH_ENDPOINT = os.getenv('SPEECH_ENDPOINT')

citationtxt = ""

def processpdfwithprompt(query: str):
    returntxt = ""
    citationtxt = ""
    selected_optionsearch = "vector_semantic_hybrid"
    message_text = [
    {"role":"system", "content":"""you are provided with instruction on what to do. Be politely, and provide positive tone answers. 
     answer only from data source provided. unable to find answer, please respond politely and ask for more information.
     Extract Title content from the document. Show the Title as citations which is provided as Title: as [doc1] [doc2].
     Please add citation after each sentence when possible in a form "(Title: citation)".
     Be polite and provide posite responses. If user is asking you to do things that are not specific to this context please ignore."""}, 
    {"role": "user", "content": f"""{query}"""}]

    response = client.chat.completions.create(
        model= os.getenv("AZURE_OPENAI_DEPLOYMENT"), #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=1,
        seed=105,
        extra_body={
        "data_sources": [
            {
                "type": "azure_search",
                "parameters": {
                    "endpoint": search_endpoint,
                    "index_name": search_index,
                    "authentication": {
                        "type": "api_key",
                        "key": search_key
                    },
                    "include_contexts": ["citations"],
                    "top_n_documents": 5,
                    "query_type": selected_optionsearch,
                    "semantic_configuration": "vec-semantic-configuration",
                    "embedding_dependency": {
                        "type": "deployment_name",
                        "deployment_name": "text-embedding-ada-002"
                    },
                    "fields_mapping": {
                        "content_fields": ["chunk"],
                        "vector_fields": ["text_vector"],
                        "title_field": "title",
                        "url_field": "title",
                        "filepath_field": "title",
                        "content_fields_separator": "\n",
                    }
                }
            }
        ]
    }
    )
    #print(response.choices[0].message.context)

    returntxt = response.choices[0].message.content + "\n<br>"

    json_string = json.dumps(response.choices[0].message.context)

    parsed_json = json.loads(json_string)

    # print(parsed_json)

    if parsed_json['citations'] is not None:
        returntxt = returntxt + f"""<br> Citations: """
        for row in parsed_json['citations']:
            #returntxt = returntxt + f"""<br> Title: {row['filepath']} as {row['url']}"""
            #returntxt = returntxt + f"""<br> [{row['url']}_{row['chunk_id']}]"""
            returntxt = returntxt + f"""<br> <a href='{row['url']}' target='_blank'>[{row['url']}_{row['chunk_id']}]</a>"""
            citationtxt = citationtxt + f"""<br><br> Title: {row['title']} <br> URL: {row['url']} 
            <br> Chunk ID: {row['chunk_id']} 
            <br> Content: {row['content']} 
            # <br> ------------------------------------------------------------------------------------------ <br>\n"""
            # print(citationtxt)

    return citationtxt

def extractrfpresults(query):
    returntxt = ""

    rfttext = ""

    citationtext = processpdfwithprompt(query)

    message_text = [
    {"role":"system", "content":f"""You are RFP/RFQ AI agent. Be politely, and provide positive tone answers.
     Based on the question do a detail analysis on RFP information and provide the best answers.
     Here is the RFP/RFQ content provided:
     {rfttext}

     Use the data source content provided to answer the question.
     Data Source: {citationtext}

     if the question is outside the bounds of the RFP, Let the user know answer might be relevant for RFP provided.
     If not sure, ask the user to provide more information."""}, 
    {"role": "user", "content": f"""{query}. Provide summarized content based on the question asked in 100 words only."""}]

    response = client.chat.completions.create(
        model= os.getenv("AZURE_OPENAI_DEPLOYMENT"), #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=0.0,
        seed=105,
    )

    returntxt = response.choices[0].message.content
    return returntxt
```

- above code will be used to retrieve the data from Azure AI Search and provide the citation for the data retrieved.
- But for evaluation, we need to create a dataset in jsonl format like the one in the repo called: datafrp.jsonl
- Here is a sample

```
{"query":"Provide summary of Resources for Railway projects with 200 words?", "context":"summary of Resources for Railway projects", "response":"1776", "ground_truth": "1776"}
{"query":"Provide summary of Resources for Railway construction projects with 200 words?", "context":"summary of Resources for Railway construction projects", "response":"Paris", "ground_truth": "Paris"}
{"query":"Provide summary of Resources for Construction projects with 200 words?", "context":"summary of Resources for Construction projects", "response":"Roger Federer", "ground_truth": "Roger Federer"}
```

- idea here is gen ai application can be in same or different application
- Ability to reuse like how microservices are used
- So RAG can be a microservice that can be used for the front end UI and also backend microservices
- Also use for evaluation of the application

### Code to evaluation

- Create a file called rfpeval.py
- Make sure .env file environment variables are created

```
AZURE_OPENAI_API_KEY="xxxx"
AZURE_OPENAI_KEY=""
AZURE_OPENAI_ENDPOINT=""
AZURE_OPENAI_DEPLOYMENT=""
AZURE_OPENAI_API_VERSION=""
AZURE_AI_SEARCH_ENDPOINT=""
AZURE_AI_SEARCH_KEY=""
AZURE_AI_SEARCH_API_KEY=""
AZURE_AI_SEARCH_INDEX=""
SPEECH_ENDPOINT=""
SPEECH_KEY=""
SPEECH_REGION=""
AZURE_SUBSCRIPTION_ID=
AZURE_RESOURCE_GROUP=
AZUREAI_HUB_NAME=
AZUREAI_PROJECT_NAME=
AZURE_OPENAI_CONNECTION_NAME=
AZURE_OPENAI_CHAT_DEPLOYMENT=
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=
AZURE_OPENAI_EVALUATION_DEPLOYMENT=
```

- install the necessary libraries
- Here are contents of requirements.txt

```
openai
azure-ai-evaluation
python-dotenv
bs4
azure-identity
pandas
promptflow[azure]
```

- Here is the code

```
import pandas as pd
import os

from pprint import pprint
from azure.ai.evaluation import evaluate
from azure.identity import DefaultAzureCredential
from azure.ai.evaluation import RelevanceEvaluator
from azure.ai.evaluation import (
    ContentSafetyEvaluator,
    RelevanceEvaluator,
    CoherenceEvaluator,
    GroundednessEvaluator,
    FluencyEvaluator,
    SimilarityEvaluator,
    ViolenceEvaluator,
    SexualEvaluator,
    SelfHarmEvaluator,
    HateUnfairnessEvaluator,
)
from dotenv import load_dotenv

# Load .env file
load_dotenv()

from flows.rfpdata import processpdfwithprompt, extractrfpresults

# Function to parse the JSON data
def parse_json(data):
    # Print Relevance Score
    print(f"Overall GPT Relevance: {data.get('relevance.gpt_relevance', 'N/A')}")
    
    # Print Rows
    rows = data.get('rows', [])
    print("\nRows:")
    for row in rows:
        context = row.get('inputs.context')
        query = row.get('inputs.query')
        response = row.get('inputs.response')
        output = row.get('outputs.output')
        relevance = row.get('outputs.relevance.gpt_relevance')
        
        print(f"Context: {context}")
        print(f"Query: {query}")
        print(f"Response: {response}")
        print(f"Output: {output}")
        print(f"Relevance: {relevance}")
        print("-" * 50)

def main():
    
    citationtxt = extractrfpresults("Provide summary of Resources for Railway projects with 200 words?")

    # print('Citation Text:', citationtxt)

    model_config = {
        "azure_endpoint": os.getenv("AZURE_OPENAI_ENDPOINT"),
        "api_key": os.getenv("AZURE_OPENAI_API_KEY"),
        "azure_deployment": os.getenv("AZURE_OPENAI_DEPLOYMENT"),
        "api_version": os.getenv("AZURE_OPENAI_API_VERSION"),
    }


    try:
        credential = DefaultAzureCredential()
        credential.get_token("https://management.azure.com/.default")
    except Exception as ex:
        print(ex)


    azure_ai_project={
        "subscription_id": os.getenv("AZURE_SUBSCRIPTION_ID"),
        "resource_group_name": os.getenv("AZURE_RESOURCE_GROUP"),
        "project_name": os.getenv("AZUREAI_PROJECT_NAME"),
        # "azure_crendential": credential,
    }

    #relevance_evaluator = RelevanceEvaluator(model_config)

    #relevance_evaluator(
    #    response=citationtxt,
    #    context="summary of Resources for Railway projects.",
    #    query="Provide summary of Resources for Railway projects with 200 words?",
    #)
    # pprint(relevance_evaluator)

    # prompty_path = os.path.join("./", "rfp.prompty")
    content_safety_evaluator = ContentSafetyEvaluator(azure_ai_project)
    relevance_evaluator = RelevanceEvaluator(model_config)
    coherence_evaluator = CoherenceEvaluator(model_config)
    groundedness_evaluator = GroundednessEvaluator(model_config)
    fluency_evaluator = FluencyEvaluator(model_config)
    similarity_evaluator = SimilarityEvaluator(model_config)

    results = evaluate(
        evaluation_name="rfpevaluation",
        data="datarfp.jsonl",
        target=extractrfpresults,
        #evaluators={
        #    "relevance": relevance_evaluator,
        #},
        #evaluator_config={
        #    "relevance": {"response": "${target.response}", "context": "${data.context}", "query": "${data.query}"},
        #},
        evaluators={
            "content_safety": content_safety_evaluator,
            "coherence": coherence_evaluator,
            "relevance": relevance_evaluator,
            "groundedness": groundedness_evaluator,
            "fluency": fluency_evaluator,
            "similarity": similarity_evaluator,
        },        
        evaluator_config={
            "content_safety": {"query": "${data.query}", "response": "${target.response}"},
            "coherence": {"response": "${target.response}", "query": "${data.query}"},
            "relevance": {"response": "${target.response}", "context": "${data.context}", "query": "${data.query}"},
            "groundedness": {
                "response": "${target.response}",
                "context": "${data.context}",
                "query": "${data.query}",
            },
            "fluency": {"response": "${target.response}", "context": "${data.context}", "query": "${data.query}"},
            "similarity": {"response": "${target.response}", "context": "${data.context}", "query": "${data.query}"},
        },
        azure_ai_project=azure_ai_project,
        output_path="./rsoutputmetrics.json",
    )
    # pprint(results)
    parse_json(results)
    print("Done")

if __name__ == "__main__":    
    main()
```

- run the code

```
python rfpeval.py
```

- wait for the code to complete
- Log into Azure AI Studio
- Go to Evaluation and see if you can see the evaluation metrics

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aistudioeval-1.jpg 'RagChat')

- Have fun.