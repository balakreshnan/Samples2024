# Azure Open AI Assitant with csv file

## Use Case

- Titanic dataset
- ask questions to get answers from the documents
- Using Azure Open AI gpt 4 turbo
- ask questions on tabular dataset

## Requirements

- Azure Subscription
- Azure Machine learning
- Azure Open AI gpt 4 turbo
- Python
- Titanic dataset

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

- Creating polling function

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

- message processing function

```
from typing import Optional

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

- Retrieve and display the thread

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

- upload the file

```
# Upload a file with an "assistants" purpose
file = client.files.create(
  file=open("Titanic.csv", "rb"),
  purpose='assistants'
)
```

- Create a assistant

```
# Create an assistant using the file ID
assistant = client.beta.assistants.create(
  instructions="You are an AI assistant that can write code to help answer math questions.",
  model="gpt-4-turbo",
  tools=[{"type": "code_interpreter"}],
  file_ids=[file.id]
)
```

- Create a thread

```
thread = client.beta.threads.create(
  messages=[
    {
      "role": "user",
      "content": "can you calculate how many surviours were there in the data? explain how did you derive the calculations",
      #"file_ids": ["file.id"] # file id will look like: "assistant-R9uhPxvRKGH3m0x5zBOhMjd2" 
    }
  ]
)
```

- setup instructions

```
name = "aisearch-assistant"
instructions = """You are an assistant designed to help people answer questions.
"""
```

- run thread

```
run = client.beta.threads.runs.create(thread_id=thread.id, assistant_id=assistant.id, instructions=instructions)
#poll_run_till_completion(
#    client=client, thread_id=thread.id, run_id=run.id, verbose=True
#)
messages = retrieve_and_print_messages(client=client, thread_id=thread.id, verbose=True)
```

- check for status

```
run = client.beta.threads.runs.retrieve(
  thread_id=thread.id,
  run_id=run.id)
#time.sleep(2.5)
print(run.status)
```

- get the response

```
messages = client.beta.threads.messages.list(
  thread_id=thread.id)
print(messages.data[0].content[0].text.value)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AOAI/images/ragtableassistant1.jpg 'RagChat')