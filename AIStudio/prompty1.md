# Prompty Example 1 for Gen AI Application

## aisearch code

```
from typing import List
import os
from azure.identity import DefaultAzureCredential
from azure.search.documents import SearchClient
from azure.search.documents.models import (
    VectorizedQuery,
    QueryType,
    QueryCaptionType,
    QueryAnswerType,
)
from azure.core.credentials import AzureKeyCredential

from dotenv import dotenv_values

config = dotenv_values("env.env")

azuresearchendpoint = config["AZURE_SEARCH_ENDPOINT"]
azuresearchkey = config["AZURE_SEARCH_KEY"]

def retrieve_documentation(
    question: str,
    index_name: str,
    embedding: List[float],
) -> str:

    
    search_client = SearchClient(
        endpoint=azuresearchendpoint,
        index_name=index_name,
        credential=AzureKeyCredential(azuresearchkey),
    )

    vector_query = VectorizedQuery(
        vector=embedding, k_nearest_neighbors=3, fields="contentVector"
    )

    results = search_client.search(
        search_text=question,
        vector_queries=[vector_query],
        query_type=QueryType.SEMANTIC,
        semantic_configuration_name="default",
        query_caption=QueryCaptionType.EXTRACTIVE,
        query_answer=QueryAnswerType.EXTRACTIVE,
        top=3,
    )

    docs = [
        {
            "id": doc["id"],
            "title": doc["title"],
            "content": doc["content"],
            "url": doc["url"],
        }
        for doc in results
    ]

    return docs
```

- AI Search function above is created as new aisearch.py file
- It's needed to retrieve context data to ground the model
- Setting Environment variables in the env.env file
- Setting Environment variables in the windows machine is been challenging to recognize the environment variables

## Code

```
from dotenv import load_dotenv
#load_dotenv()
from dotenv import dotenv_values

config = dotenv_values("env.env")

#config = load_dotenv("env.env")
azure_deployment = config["AZURE_OPENAI_EMBEDDING_DEPLOYMENT"]
api_key = config["AZURE_OPENAI_API_KEY"]
api_version = config["AZURE_OPENAI_API_VERSION"]
api_base = config["AZURE_OPENAI_ENDPOINT"]
embeddingmodel = config["AZURE_OPENAI_EMBEDDING_MODEL"]
azuresearchendpoint = config["AZURE_SEARCH_ENDPOINT"]
azure_chatdeployment = config["AZURE_OPENAI_CHAT_DEPLOYMENT"]
azure_endpoint = config["AZURE_OPENAI_ENDPOINT"]

from sys import argv
import os
import pathlib
from ai_search import retrieve_documentation
from promptflow.tools.common import init_azure_openai_client
from promptflow.connections import AzureOpenAIConnection
from promptflow.core import (AzureOpenAIModelConfiguration, Prompty, tool)
from promptflow.tracing import start_trace
start_trace()

def get_context(question, embedding):
    return retrieve_documentation(question=question, index_name="aistudioprofile", embedding=embedding)

def get_embedding(question: str):
    #config = load_dotenv("env.env")
    
    #connection = AzureOpenAIConnection(        
    #                azure_deployment=os.getenv("AZURE_OPENAI_EMBEDDING_DEPLOYMENT", ""),
    #                api_version=os.getenv("AZURE_OPENAI_API_VERSION", ""),
    #                api_base=os.getenv("AZURE_OPENAI_ENDPOINT", "")
    #                )
    connection = AzureOpenAIConnection(        
                    #azure_deployment=azure_deployment,
                    api_key=api_key,
                    #api_version=api_version,
                    api_base=api_base,
                    #endpoint=api_base
                    )
                
    client = init_azure_openai_client(connection)

    return client.embeddings.create(
            input=question,
            model=embeddingmodel,
        ).data[0].embedding

@tool
def get_response(question, chat_history):
    print("inputs:", question)
    embedding = get_embedding(question)
    context = get_context(question, embedding)
    print("context:", context)
    print("getting result...")

    ##configuration = AzureOpenAIModelConfiguration(
    #    azure_deployment=azure_chatdeployment,
    #    api_version=api_version,
    #    azure_endpoint=api_base,
    #    api_key=api_key,
    #    #api_base=api_base
    #)
    #override_model = {
    #    "configuration": configuration,
    #    "parameters": {"max_tokens": 512}
    #}

    # Load prompty with AzureOpenAIModelConfiguration override
    configuration = AzureOpenAIModelConfiguration(
        azure_deployment="gpt-4o-g",
        api_key=api_key,
        api_version=api_version,
        azure_endpoint=azure_endpoint,
        #connection=AzureOpenAIConnection(api_base=api_base, api_key=api_key, api_type="azure_openai"),
    )
    override_model = {
        "configuration": configuration,
        "parameters": {"max_tokens": 512}
    }
    #prompty = Prompty.load(source="path/to/prompty.prompty", model=override_model)
    
    #data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "./chat.prompty")
    #print(data_path)

    data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "chat1.prompty")
    print(data_path)

    #prompty_obj = Prompty.load(data_path, model=override_model)
    prompty_obj  = Prompty.load(source="chat1.prompty", model=override_model)

    #result = prompty_obj(question = question, documents = context)
    result = prompty_obj(first_name="John", last_name="Doh", question="What is the capital of France?")

    print("result: ", result)


    return {"answer": result, "context": context}

if __name__ == "__main__":
    print("running chat request")
    get_response("show me top 5 technology leadership candidates", [])
```