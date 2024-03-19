# Microsoft Auto-Gen implementing RAG (Retrieval Augmented Generation) for your own data

## Introduction

- Build a agent framework for your own data
- Use the RAG model to generate responses
- Talk to Azure AI Search to retrieve documents
- Summarize the documents

## Pre-requisites

- Python 3.6 or later
- Azure Subscription
- Azure machine learning service
- Azure Cognitive Search
- microsoft autogen sdk

## Steps

- install the microsoft autogen sdk

```
%pip install pyautogen
```

- Now import autogen

```
import autogen
```

- load environment variables

```
from dotenv import dotenv_values
# specify the name of the .env file name 
env_name = "env.env" # change to use your own .env file
config = dotenv_values(env_name)
```

- Load open ai and it's configuration

```
import openai 

openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_API_KEY"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = config["AZURE_OPENAI_API_VERSION"]
openai.base_url = config["AZURE_OPENAI_ENDPOINT"]
```

- Load the Azure AI Search keys

```
search_endpoint = config["AZURE_SEARCH_ENDPOINT"]
index_name = config["AZURE_SEARCH_INDEX_NAME"]
key = config["AZURE_SEARCH_KEY"]
```

- Set the LLM config to use Azure open ai service

```
llm_config={"config_list": [
    {"model": "gpt-4-turbo", "api_key": config["AZURE_OPENAI_API_KEY"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT"],
    "api_type" : "azure", "api_version" : "2024-02-01"}
    ]}
```

- create the function to conver to embeddings for vector search

```
def get_embeddings(text: str):
    # There are a few ways to get embeddings. This is just one example.
    import openai

    open_ai_endpoint = "https://aoairesourcename.openai.azure.com/"
    open_ai_key = "xxxxxxxxx"

    client = openai.AzureOpenAI(
        azure_endpoint=open_ai_endpoint,
        api_key=open_ai_key,
        api_version="2024-02-01,
    )
    embedding = client.embeddings.create(input=[text], model="text-embedding-ada-002")
    return embedding.data[0].embedding
```

- Create Search client

```
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.models import VectorizedQuery
from typing import Optional
import json


search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
```

- now setup the Assistant agent

```
from autogen import AssistantAgent, UserProxyAgent

# create an AssistantAgent instance named "assistant"
assistant = AssistantAgent(name="assistant")

# create a UserProxyAgent instance named "user_proxy"
user_proxy = UserProxyAgent(name="user_proxy")
```

- configure the chatbot and user agent

```
chatbot = autogen.AssistantAgent(
    name="chatbot",
    system_message="""You are HR AI Assitant to provide profiles, only use the functions you have been provided with.
    Provide Citations which are provided as url in the content.""",
    llm_config=llm_config,
)

# create a UserProxyAgent instance named "user_proxy"
user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    #is_termination_msg=lambda x: x.get("content", "") and x.get("content", "").rstrip().endswith("TERMINATE"),
    human_input_mode="NEVER",
    max_consecutive_auto_reply=1,
)
```

- Now create and register the Azure AI search function

```
@user_proxy.register_for_execution()
@chatbot.register_for_llm(name="python", description="Run Python function to retrieve data from ai search.")
def searchai(searchtext="List top 5 Strategy leadership candidates with details?"):
    searchcontent= ""
    search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
    vector_query = VectorizedQuery(vector=get_embeddings(searchtext), k_nearest_neighbors=5, fields="contentVector")#
    #results = search_client.search(search_text=searchtext)
    results = search_client.search(vector_queries=[vector_query])
    for result in results:
        print("{}: {})".format(result["url"], result["content"]))
        searchcontent += f"{result['content']} : {result['url']}"
    # print('Search Content:' , searchcontent)
    return json.dumps(searchcontent)
```

- Register the function
- We need this to use inside an agent

```
# Register the function with the chatbot's llm_config.
searchai = chatbot.register_for_llm(description="Search Profiles.")(searchai)

# Register the function with the user_proxy's function_map.
user_proxy.register_for_execution()(searchai)
```

- now set the query

```
query = "List top 5 CTO leadership position role with details?"
```

- Initiate the chat

```
result = user_proxy.initiate_chat(
    chatbot,
    message=query,
)
```

- print the last output
- Output has chat_history object which has all the history of the chat

```
print(result.chat_history[-1]['content'])
```