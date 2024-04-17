# Create dynamics agents for various use case

## Introduction

- Ability to create multiple agents with different dynamics properties
- Provide system prompt for each agent
- Each agent can do different tasks
- Use for Software design on various business use cases
- Using Microsoft autogen

## pre-requisite

- Azure account
- Microsoft autogen
- Azure open ai
- visual studio code
- python 3.11

## Steps

- install libraries
- Create a requirements.txt file and add the below packages
- Then install all of them

```
openai
python-dotenv
gradio
PyPDF2
streamlit
azure-kusto-data
pyautogen
pyautogen[autobuild]
pyautogen[graph]>=0.2.11
```

### Code Section

- Create a new python file and add the below code
- import libraries

```
import os
from openai import AzureOpenAI
import gradio as gr
from dotenv import dotenv_values
import time
from datetime import timedelta
import json
import streamlit as st
from PIL import Image
import base64
import requests
import io
import autogen
from typing import Optional
from typing_extensions import Annotated
from autogen import AssistantAgent
from autogen.agentchat.conversable_agent import ConversableAgent  # noqa E402
from autogen.agentchat.assistant_agent import AssistantAgent  # noqa E402
from autogen.agentchat.groupchat import GroupChat  # noqa E402
from autogen.graph_utils import visualize_speaker_transitions_dict  # noqa E402
```

- load configuration and open ai client configuration

```
config = dotenv_values("env.env")

css = """
.container {
    height: 75vh;
}
"""

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_ASSITANT"], 
  api_key=config["AZURE_OPENAI_KEY_ASSITANT"],  
  api_version="2024-02-15-preview"
  #api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)
```

- now configure llm config for autogen agents

```
model_name = "gpt-35-turbo-16k"

llm_config={"config_list": [
    {"model": "gpt-35-turbo", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"},
    {"model": "gpt-35-turbo-16k", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"},
    {"model": "gpt-4-turbo", "api_key": config["AZURE_OPENAI_KEY_ASSITANT"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT_ASSITANT"],
    "api_type" : "azure", "api_version" : "2024-02-01"}
    ],
    "timeout": 600,
    "cache_seed": 44,
    "temperature": 0.0}

import json
```

- termination functions

```
def termination_msg(x):
    return isinstance(x, dict) and "TERMINATE" == str(x.get("content", ""))[-9:].upper()
```

- function to dynamically create text boxes to add system prompts
- Each system prompt will be a agent
- for example if we say 5 then 5 agents will be created

```
def generate_input_boxes(count):
    input_values = []
    for i in range(count):
        value = st.text_input(f'Enter value {i+1}:', key=i)
        #description = st.text_input(f'Enter description {i+1}:', key=i+1)
        input_values.append(value)
    return input_values
```

- now create process agents function

```
def processagent(query, input_values, mainassistant, maindesc):
    returntxt = ""
    agents = []
    agentsdict = {}


    initializer = autogen.UserProxyAgent(
        name="Init",
        is_termination_msg=termination_msg,
        human_input_mode="NEVER",
        code_execution_config=False,  # we don't want to execute code in this case.
        default_auto_reply="Reply `TERMINATE` if the task is done.",
        description="The boss who ask creates features and prioritize tasks. Come up with new features and method to solve the issue.",
        )

    Architect = AssistantAgent(
        name="Architect",
        is_termination_msg=termination_msg,
        human_input_mode="NEVER",
        system_message=mainassistant,
        llm_config=llm_config,
        description=maindesc,
    )

    agents.append(Architect)

    #print(input_values)

    cnt = 1
    for val in input_values:
        #print('input value: ', str(val))
        node_id = f"useragent{cnt}"
        agentsdict["agent" + str(cnt)] = val
        useragent = autogen.AssistantAgent(
            #name="agent" + str(cnt),
            name=node_id,
            system_message=f"""Your name is {node_id}.
            you are provided instruction as {val}
            Based on the instruction you have to provide or ask questions to understand the business process.
            Ask follow up questions if it's not clear.
            """,
            is_termination_msg=termination_msg,
            description="I am Data Scientist who want to solve complex customer problem",
            llm_config=llm_config,
        )
        print('useragent: ', useragent)
        agents.append(useragent)
        cnt += 1

        #if "TERMINATE" in returntxt:
        #    break
        #commisioner.register_reply([autogen.Agent, None], reply_func=send_messages, config={"callback": None})

    for ag in agents:
            print('Agent name: ', ag.name)

    print('agents: ', agents)
    groupchat = autogen.GroupChat(agents=agents,
        messages=[], max_round=20, speaker_selection_method="round_robin"
    )
    manager = autogen.GroupChatManager(groupchat=groupchat, llm_config=llm_config)

    # Start chatting with boss_aid as this is the user proxy agent.
    result = initializer.initiate_chat(
            manager,
            message=query,
            n_results=5,
            #clear_history=False
    )

    print('output: ' , groupchat.messages)

    #returntxt = str(result.chat_history)

    #returntxt = str(groupchat.messages)

    for row in groupchat.messages:
        returntxt += f"""{row["name"]}: {row["content"]}\n <br><br>"""

    #initializer.reset()

    return returntxt
```

- here is the function to create UI and process the response

```
def dynagents():
    st.title("Dynamic Agents creation based on user input")

    count = 1000
    rttxt = ""

    col1, col2 = st.columns(2)

    with col1:
        query = st.text_input("Problem:", key=count, value="Create a Supply chain co pilot agent to answer question on supplier contracts and their delivery times.")
        count += 1

        mainassistant = st.text_input("Assistant System Message:", key=count, value="You are a Lead Architect, Ask followup questions to understand the business process and persona who will use the system and provide simple high level architecture. Reply `TERMINATE` in the end when everything is done.")
        count += 1

        maindesc = st.text_input("Assistant Description:", key=count, value="I am Lead Architect and my job is to understand the use case from business owner and then design the technology pieces needed.")
        count += 1
        # Get the count of input boxes from the user
        count = st.number_input("Enter the count of input boxes:", min_value=1, step=1)

        # Generate dynamic input boxes based on count
        input_values = generate_input_boxes(count)

        if st.button("Generate Response"):
            if query:
                rttxt = processagent(query, input_values, mainassistant, maindesc)
                print('rttxt: ', rttxt)
                #st.write(rttxt)

        # Display the values entered by the user
        st.write("Values entered:")
        st.write(input_values)
    
    with col2:
        if rttxt:
            #st.write(rttxt) 
            htmloutput = f"""<html>
                <head>
                </head>
                <body>
                <div class="container">{rttxt}</div>            
                </body>
                </html>"""
            st.components.v1.html(htmloutput, height=550, width=600, scrolling=True)

    

#if __name__ == "__main__":
#    dynagents()
```

- run the app

```
streamlit run autogenapp.py
```

- in the UI provide prompts on what the agent should do for user proxy and assistant agents
- you can play with multiple number of agents.