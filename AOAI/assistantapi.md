# Azure Open AI Assistant API - Python

## Using Azure Machine Learning compute instance

## Introduction

- Show case how to access the Azure Open AI Assistant API using Python
- Using Azure Machine Learning compute instance
- Run a end to end flow
- Display output

## Prerequisites

- Azure subscription
- Azure machine learning Service
- Compute instance
- Azure open ai service in east us 2 or sweden (at the time of writing this document)
- Simple example to run

## Code

- Install latest open ai python sdk in python kernel 3
- Since i already have older openai i am going to upgrade

```
%pip install --upgrade openai
```

- Setup the Open AI Client
- Azure open ai in east us2
- Make sure we have the latest version of api(older version will error)

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://aoainame.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxxxxxxx",  
  api_version="2024-02-15-preview"
)
```

- Test the client
- Run a simple example

```
message_text = [{"role":"system","content":"You are an AI assistant that helps people find information."},
{"role": "user", "content": "what is the age of michael jordan"}]
```

```
response = client.chat.completions.create(
    model="gpt-4-turbo", # model = "deployment_name".
    messages=message_text
)

print(response.choices[0].message.content)
```

- funtion to how the json output

```
import json

def show_json(obj):
    display(json.loads(obj.model_dump_json()))
```

- now lets start the assistant
- Provide instructions

```
assistant = client.beta.assistants.create(
    name="Math Tutor",
    instructions="You are a personal math tutor. Write and run code to answer math questions.",
    tools=[{"type": "code_interpreter"}],
    model="gpt-4-turbo"
)

show_json(assistant)
```

- Next create thread

```
thread = client.beta.threads.create()
```

- Set the question to ask

```
message = client.beta.threads.messages.create(
    thread_id=thread.id,
    role="user",
    content="I need to solve the equation `3x + 11 = 14`. Can you help me?"
)
show_json(message)
```

- function to wait for run to complete

```
import time

def wait_on_run(run, thread):
    while run.status == "queued" or run.status == "in_progress":
        run = client.beta.threads.runs.retrieve(
            thread_id=thread.id,
            run_id=run.id,
        )
        time.sleep(0.5)
    return run
```

- Create the run using threads

```
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
)
show_json(run)
```

- run the code

```
run = wait_on_run(run, thread)
show_json(run)
```

- list all messages

```
messages = client.beta.threads.messages.list(thread_id=thread.id)
show_json(messages)
```

- ask for model to explain

```
# Create a message to append to our thread
message = client.beta.threads.messages.create(
    thread_id=thread.id, role="user", content="Could you explain this to me?"
)

# Execute our run
run = client.beta.threads.runs.create(
    thread_id=thread.id,
    assistant_id=assistant.id,
)

# Wait for completion
wait_on_run(run, thread)

# Retrieve all the messages added after our last user message
messages = client.beta.threads.messages.list(
    thread_id=thread.id, order="asc", after=message.id
)
show_json(messages)
```

- display the output

```
print(messages.data[0].content[0].text.value)
```

- output

```
Certainly! To solve a linear equation like \(3x + 11 = 14\), our goal is to isolate the variable \(x\) on one side of the equation. We want to find the value of \(x\) that makes the equation true. To do this, we perform a series of operations that will simplify the equation step by step. Here's how we do it:

1. **Isolate the variable term**: We start by moving the constant term that does not contain \(x\) to the other side of the equation. We do this by doing the opposite operation to both sides of the equation. In this case, we subtract 11 from both sides because the constant term is \(+11\) on the left side:

\[3x + 11 - 11 = 14 - 11\]

The \(+11\) and \(-11\) on the left side cancel out, leaving us with:

\[3x = 3\]

2. **Solve for the variable**: We now have a simpler equation, \(3x = 3\), where \(x\) is multiplied by 3. To isolate \(x\), we want to undo this multiplication by doing the opposite operation, which is division. We divide both sides of the equation by 3:

\[\frac{3x}{3} = \frac{3}{3}\]

On the left side, the \(3\) in the numerator and the \(3\) in the denominator cancel each other out, and we are left with just \(x\). On the right side, 3 divided by 3 equals 1. So our equation simplifies to:

\[x = 1\]

And we find that \(x\) must equal 1 to make the original equation true.

In summary, the steps to solve \(3x + 11 = 14\) are:

1. Subtract 11 from both sides to get \(3x = 3\).
2. Divide both sides by 3 to get \(x = 1\).

So the solution to the equation is \(x = 1\).
```