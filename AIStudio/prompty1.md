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

def get_response(question, chat_history):
    print("inputs:", question)
    embedding = get_embedding(question)
    context = get_context(question, embedding)
    #print("context:", context)
    print("getting result...")
    rs = []

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
    # Override model config with dict
    model_config = {
        "api": "chat",
        "configuration": {
            "type": "azure_openai",
            "azure_deployment": "gpt-4o-g",
            "api_key": f"{api_key}",
            "api_version": f"{api_version}",
            "azure_endpoint": f"{azure_endpoint}",
        },
        "parameters": {
            "max_tokens": 512
        }
    }

    #print(model_config)

    data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "chat1.prompty")
    #print(data_path)

    #prompty_obj = Prompty.load(data_path, model=override_model)
    #prompty_obj  = Prompty.load(source="chat1.prompty", model=model_config)
    prompty_obj  = Prompty.load(source="chat.prompty", model=model_config)

    result = prompty_obj(question = question, documents = context)
    #result = prompty_obj(first_name="John", last_name="Doh", question="What is the capital of France?")

    print("result: ", result)

    # Initialize Azure OpenAI Connection with your environment variables
    eval_model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=azure_endpoint,
        api_key=api_key,
        azure_deployment=azure_chatdeployment,
        api_version=api_version,
    )

    relevance_evaluator = RelevanceEvaluator(model_config=eval_model_config)
    coherence_evaluator = CoherenceEvaluator(model_config=eval_model_config)
    #path = "profiles.jsonl"

    evalutionname = "chat_eval_gpt4og_" + str(uuid.uuid4())[:8]
    print(evalutionname)
    json_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "profiles.jsonl")
    print(json_path)

    # Initialzing Relevance Evaluator
    relevance_eval = RelevanceEvaluator(eval_model_config)
    # Running Relevance Evaluator on single input row
    relevance_score = relevance_eval(
        answer="The Alpine Explorer Tent is the most waterproof.",
        context="From the our product list,"
        " the alpine explorer tent is the most waterproof."
        " The Adventure Dining Table has higher weight.",
        question="Which tent is the most waterproof?",
    )
    print(relevance_score)

    # Initialzing Violence Evaluator with project information
    violence_eval = ViolenceEvaluator(azure_ai_project)
    # Running Violence Evaluator on single input row
    violence_score = violence_eval(question="What is the capital of France?", answer="Paris.")
    print(violence_score)

    #print(rs)
    # Convert the Python list (array) to a JSON string
    data = [{
        "chat_input": question,
        "answer": result,
        "context": str(context)
    },
    {
        "chat_input": question,
        "answer": result,
        "context": str(context)
    }]
    json_data = json.dumps(data, indent=4)

    # Write the JSON string to a file
    #with open('chat_data.jsonl', 'w') as file:
    #    file.write(json_data)
    # Write the JSONL file
    with open('chat_data.jsonl', 'w') as file:
        for entry in data:
            # Convert each dictionary to a JSON string and write it as a new line
            file.write(json.dumps(entry) + '\n')

    print("JSON file created successfully!")


    return {"answer": result, "context": context}

# Helper methods
def load_jsonl(path):
    with open(path, "r") as f:
        return [json.loads(line) for line in f.readlines()]


def copilot_wrapper(*, chat_input, **kwargs):
    #from copilot_flow.copilot import get_chat_response#

    result = get_response(chat_input)

    parsedResult = {"answer": str(result["answer"]), "context": str(result["context"])}
    return parsedResult


def run_evaluation(eval_name, dataset_path):

    # Initialize Azure OpenAI Connection with your environment variables
    model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=azure_endpoint,
        api_key=api_key,
        azure_deployment=azure_chatdeployment,
        api_version=api_version,
    )


    # Initializing Evaluators
    relevance_eval = RelevanceEvaluator(model_config)
    groundedness_eval = GroundednessEvaluator(model_config)
    coherence_eval = CoherenceEvaluator(model_config)

    output_path = "./eval_results.jsonl"
    print(" Data Path: ", dataset_path)

    result = evaluate(
        target=copilot_wrapper,
        evaluation_name=eval_name,
        data=dataset_path,
        evaluators={
            "relevance": relevance_eval,
            "groundedness": groundedness_eval,
            "coherence": coherence_eval,
        },
        evaluator_config={
            "relevance": {"question": "${data.chat_input}", "answer": "${data.answer}", "context": "${data.context}"},
            "coherence": {"question": "${data.chat_input}", "answer": "${data.answer}", "context": "${data.context}"},
        },
        # to log evaluation to the cloud AI Studio project
        azure_ai_project=azure_ai_project,
    )

    tabular_result = pd.DataFrame(result.get("rows"))
    tabular_result.to_json(output_path, orient="records", lines=True)

    return result, tabular_result

if __name__ == "__main__":
    print("running chat request")
    answer = get_response("show me top 5 technology leadership candidates", [])
    eval_name = "chat-eval"
    #dataset_path = "./eval_dataset.jsonl"
    dataset_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "chat_data.jsonl")
    #https://learn.microsoft.com/en-us/azure/ai-studio/tutorials/copilot-sdk-build-rag?tabs=azure-portal
    #https://learn.microsoft.com/en-us/azure/ai-studio/tutorials/copilot-sdk-evaluate-deploy

    df = pd.read_json(dataset_path, lines=True)
    print(df)

    result, tabular_result = run_evaluation(
        eval_name=eval_name, dataset_path=dataset_path
    )
```