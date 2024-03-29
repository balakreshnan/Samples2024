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


search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
```

- search

```
from typing import Optional

def searchai(searchtext):
    searchcontent= ""
    search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
    results = search_client.search(search_text="CEO Role")
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
import time

name = "aisearch-assistant"
instructions = """You are an assistant designed to help people answer questions.

You have access to query the web using azure cognitive ai Search. You should call ai search whenever a question requires up to date information or could benefit from profiles data.
"""

message = {"role": "user", "content": "List top 5 candidates for CEO role with details?"}


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

  
messages = client.beta.threads.messages.list(
  thread_id=thread.id)
print(messages.data[0].content[0].text.value)
```

- output

```
Certainly! Analyzing candidates' profiles and their skills requires a systematic approach to understanding where each candidate excels and where they may need improvement. Here's a step-by-step explanation of how this kind of analysis can be conducted:

1. **Data Collection**: The first step is to gather comprehensive data on each candidate. This data might include:

   - **Educational Background**: Degrees, certifications, and relevant coursework.
   - **Professional Experience**: Previous positions held, duration, roles, and responsibilities.
   - **Skill Set**: A list of technical and soft skills, languages known, software proficiency, etc.
   - **Skill Assessments**: Scores or evaluations of the candidate's skills, possibly from standardized tests or assessments.
   - **Progression Over Time**: Information on how the candidate's skills have improved over different time periods.
   - **Feedback**: Comments or reviews from peers, supervisors, or mentors.

2. **Data Structuring**: The collected data must be structured in a format that's easy for analysis, such as a spreadsheet or a database. Each candidate's information should be consistently recorded to allow for comparative analysis.

3. **Data Cleaning**: Before analysis, the data should be cleaned to remove any inaccuracies, inconsistencies, or irrelevant information.

4. **Analysis**: The analysis involves several sub-steps:
   
   - Identifying key skills that are important for the roles you're hiring for.
   - Comparing each candidate's skill set against these key skills.
   - Evaluating progression by looking at how skills have developed over time.
   - Assessing any gaps where a candidate might lack certain skills or experience.

5. **Insight Generation**: From the analysis, insights can be drawn about each candidate’s strengths and weaknesses. Some possible insights might include:

   - **Improvements**: Areas where candidates have shown notable skill growth or increased competencies.
   - **Skill Gaps**: Skills that are lacking or underdeveloped in comparison to the desired profiles.
   - **Potential for Growth**: Candidates with the potential to develop certain skills, based on their learning trajectory or adaptability.
   - **Comparative Evaluation**: How candidates stack up against one another in terms of skills and experiences relevant to the job.

6. **Recommendations**: Finally, based on the insights, recommendations can be provided. These might include:

   - Suggesting specific training or learning modules to candidates to improve their skills.
   - Recommending candidates for roles that match their current skill set.
   - Advising on the development of personalized career growth plans for candidates to help them close the skill gaps.
   - Guiding the hiring process by identifying which candidates are well-suited for certain positions.

To do all of this, I would need the relevant data which could be analyzed to provide you with the insights and recommendations. If you have such data, you can provide it in an anonymized and non-sensitive format, and I can assist you with the analysis. If you don't have the data but are looking for advice on how to proceed, I can guide you on structuring your data collection and analysis process to fit your needs.
```