# Azure Open AI Assitant with Rag AI Search

## Use Case

- Access internal documents using embeddings to ground with the user's knowledge
- ask questions to get answers from the documents
- Using Azure Open AI gpt 4 turbo
- Idea here is to learn to show how rag can be implemented

## Requirements

- Azure Subscription
- Azure Machine learning
- Azure Open AI gpt 4 turbo
- Python

## Code

- install libraries

```
%pip install azure-search-documents
```

```
%pip install openai
```

```
%pip install python-dotenv
```

- Now load the environment variables

```
import openai
import json
```

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure search

```
search_endpoint = config["AZURE_SEARCH_SERVICE_ENDPOINT"]
index_name = config["AZURE_SEARCH_INDEX_NAME"]
key = config["AZURE_SEARCH_API_KEY"]
```

- Configure open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://xxxxx.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxxxxxxxxx",  
  api_version="2024-02-15-preview"
)
```

- search client

```
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.models import VectorizedQuery


search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
```

- search

```
from typing import Optional

def searchai(searchtext):
    searchcontent= ""
    search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
    vector_query = VectorizedQuery(vector=get_embeddings(searchtext), k_nearest_neighbors=5, fields="contentVector")#
    #results = search_client.search(search_text=searchtext)
    results = search_client.search(vector_queries=[vector_query])
    for result in results:
        print("{}: {})".format(result["url"], result["content"]))
        searchcontent += f"{result['content']} : {result['url']}"
    return json.dumps(searchcontent)
```

```
import json

def show_json(obj):
    display(json.loads(obj.model_dump_json()))
```

- model name

```
model_name = f'gpt-4-turbo'
```

- define poll and pull

```
def poll_run_till_completion(
    client: AzureOpenAI,
    thread_id: str,
    run_id: str,
    available_functions: dict,
    verbose: bool,
    max_steps: int = 10,
    wait: int = 3,
) -> None:
    """
    Poll a run until it is completed or failed or exceeds a certain number of iterations (MAX_STEPS)
    with a preset wait in between polls

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param run_id: Run ID
    @param assistant_id: Assistant ID
    @param verbose: Print verbose output
    @param max_steps: Maximum number of steps to poll
    @param wait: Wait time in seconds between polls

    """

    if (client is None and thread_id is None) or run_id is None:
        print("Client, Thread ID and Run ID are required.")
        return
    try:
        cnt = 0
        while cnt < max_steps:
            run = client.beta.threads.runs.retrieve(thread_id=thread_id, run_id=run_id)
            if verbose:
                print("Poll {}: {}".format(cnt, run.status))
            cnt += 1
            if run.status == "requires_action":
                tool_responses = []
                if (
                    run.required_action.type == "submit_tool_outputs"
                    and run.required_action.submit_tool_outputs.tool_calls is not None
                ):
                    tool_calls = run.required_action.submit_tool_outputs.tool_calls

                    for call in tool_calls:
                        if call.type == "function":
                            if call.function.name not in available_functions:
                                raise Exception("Function requested by the model does not exist")
                            function_to_call = available_functions[call.function.name]
                            tool_response = function_to_call(**json.loads(call.function.arguments))
                            tool_responses.append({"tool_call_id": call.id, "output": tool_response})

                run = client.beta.threads.runs.submit_tool_outputs(
                    thread_id=thread_id, run_id=run.id, tool_outputs=tool_responses
                )
            if run.status == "failed":
                print("Run failed.")
                break
            if run.status == "completed":
                break
            time.sleep(wait)

    except Exception as e:
        print(e)
```

- Create message

```
def create_message(
    client: AzureOpenAI,
    thread_id: str,
    role: str = "",
    content: str = "",
    file_ids: Optional[list] = None,
    metadata: Optional[dict] = None,
    message_id: Optional[str] = None,
) -> any:
    """
    Create a message in a thread using the client.

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param role: Message role (user or assistant)
    @param content: Message content
    @param file_ids: Message file IDs
    @param metadata: Message metadata
    @param message_id: Message ID
    @return: Message object

    """
    if metadata is None:
        metadata = {}
    if file_ids is None:
        file_ids = []

    if client is None:
        print("Client parameter is required.")
        return None

    if thread_id is None:
        print("Thread ID is required.")
        return None

    try:
        if message_id is not None:
            return client.beta.threads.messages.retrieve(thread_id=thread_id, message_id=message_id)

        if file_ids is not None and len(file_ids) > 0 and metadata is not None and len(metadata) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, file_ids=file_ids, metadata=metadata
            )

        if file_ids is not None and len(file_ids) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, file_ids=file_ids
            )

        if metadata is not None and len(metadata) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, metadata=metadata
            )

        return client.beta.threads.messages.create(thread_id=thread_id, role=role, content=content)

    except Exception as e:
        print(e)
        return None
```

- retrieve message

```
def retrieve_and_print_messages(
    client: AzureOpenAI, thread_id: str, verbose: bool, out_dir: Optional[str] = None
) -> any:
    """
    Retrieve a list of messages in a thread and print it out with the query and response

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param verbose: Print verbose output
    @param out_dir: Output directory to save images
    @return: Messages object

    """

    if client is None and thread_id is None:
        print("Client and Thread ID are required.")
        return None
    try:
        messages = client.beta.threads.messages.list(thread_id=thread_id)
        display_role = {"user": "User query", "assistant": "Assistant response"}

        prev_role = None

        if verbose:
            print("\n\nCONVERSATION:")
        for md in reversed(messages.data):
            if prev_role == "assistant" and md.role == "user" and verbose:
                print("------ \n")

            for mc in md.content:
                # Check if valid text field is present in the mc object
                if mc.type == "text":
                    txt_val = mc.text.value
                # Check if valid image field is present in the mc object
                elif mc.type == "image_file":
                    image_data = client.files.content(mc.image_file.file_id)
                    if out_dir is not None:
                        out_dir_path = Path(out_dir)
                        if out_dir_path.exists():
                            image_path = out_dir_path / (mc.image_file.file_id + ".png")
                            with image_path.open("wb") as f:
                                f.write(image_data.read())

                if verbose:
                    if prev_role == md.role:
                        print(txt_val)
                    else:
                        print("{}:\n{}".format(display_role[md.role], txt_val))
            prev_role = md.role
        return messages
    except Exception as e:
        print(e)
        return None
```

- now ask questions

```
query = "List top 5 candidates for Strategic leadership role with details?"
```

```
import time

name = "aisearch-assistant"
instructions = """You are an assistant designed to help people answer questions.

You have access to query the web using azure cognitive ai Search. You should call ai search whenever a question requires up to date information or could benefit from profiles data.
"""

message = {"role": "user", "content": query}


tools = [
    {
        "type": "function",
        "function": {
            "name": "searchai",
            "description": "Searches AI Search to get up-to-date information from the web.",
            "parameters": {
                "type": "object",
                "properties": {
                    "searchtext": {
                        "type": "string",
                        "description": "The search query",
                    }
                },
                "required": ["searchtext"],
            },
        },
    }
]

available_functions = {"searchai": searchai}
verbose_output = True

#client = AzureOpenAI(api_key=aoai_api_key, api_version=api_version, azure_endpoint=azure_endpoint)

assistant = client.beta.assistants.create(
    name=name, description="", instructions=instructions, tools=tools, model=model_name
)

thread = client.beta.threads.create()

create_message(client, thread.id, message["role"], message["content"])


run = client.beta.threads.runs.create(thread_id=thread.id, assistant_id=assistant.id, instructions=instructions)
poll_run_till_completion(
    client=client, thread_id=thread.id, run_id=run.id, available_functions=available_functions, verbose=verbose_output
)
messages = retrieve_and_print_messages(client=client, thread_id=thread.id, verbose=verbose_output)
```

- now check for status

```
run = client.beta.threads.runs.retrieve(
  thread_id=thread.id,
  run_id=run.id)
#time.sleep(2.5)
print(run.status)
```

- print message

```  
messages = client.beta.threads.messages.list(
  thread_id=thread.id)
print(messages.data[0].content[0].text.value)
```

- output

```
Based on the profiles found, here are the top candidates that hold significant strategic leadership roles:

1. Senior Managing Director at Accenture with experience in leading global teams to drive innovation in CEO and leadership mindsets, talent, and cultures. Previously held roles like Vice President of Inclusion & Diversity at Apple and various other leadership positions at Deloitte, GRAIL, and more. Education includes a Ph.D. from New York University.
   - Profile: [Senior Managing Director at Accenture](https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(44).pdf)

2. Former Chief Technology Officer at Accenture Technology with a record of leading massive talent and leadership development programs across 300,000 employees. Instrumental in Accenture's digital transformation and has partnered with top academic institutions for learning programs.
   - Profile: [CTO at Accenture Technology](https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(49).pdf)

3. Arjun Bedi – Strategic Clients Portfolio Lead and member of Accenture’s Global Management Committee. Known for operational and P&L leadership, digital/technology innovation, and CEO/C-Suite advisory. Over 25 years of leadership at Accenture, he has been influential in the Life Sciences industry.
   - LinkedIn Profile: [Arjun Bedi LinkedIn](https://www.linkedin.com/in/arjun-bedi-ba6251)

4. Senior Managing Director at Accenture, holding the position of Market Unit Lead – US Northeast on the Global Management Committee since June 2022. Other roles include North America Client Account Lead and Global Industry Sector Lead - Life Sciences at Accenture.
   - Profile: [Senior Managing Director - Market Unit Lead – US Northeast](https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(4).pdf)

5. A strategic leadership profile with experience as Vice President of Strategy & Innovation at AstraZeneca, as well as leadership roles in IBM Global Business Services and Coopers & Lybrand. Education from Harvard Business School Executive Education and the Wharton School.
   - Profile: [Vice President of Strategy & Innovation at AstraZeneca](https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(26).pdf)

These candidates have demonstrated extensive experience in leadership roles, particularly in areas related to strategy, transformation, and innovation, critical competencies for strategic leadership positions.
```