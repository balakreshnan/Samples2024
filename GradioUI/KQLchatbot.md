# Create a UI to chat with KQL Kusto Query Language

## Introduction

- Natural Language to KQL capability
- Provide schema definition with prompt
- Also provide intent based routing for other services
- Using Gradio UI for the chat
- using azure open ai models gpt 4 turbo (had the best performance)

## Pre-requisite

- Azure Subscription
- Azure Open AI Services
- Deploy Gpt 4 turbo 128K and gpt 3.5 4K token models
- python 3.11 or later
- Local laptop with visual studio code or any other IDE
- Gradio
- Open AI
- PyPDF2
- install kusto python sdk
- Create a service principal with access to Kusto

## Steps

- Install Gradio

```
pip install gradio
```

- Install Open AI

```
pip install openai
```

- Install PyPDF2

```
pip install PyPDF2
```

- install python-dotenv

```
pip install python-dotenv
```

- Import all libraries

```
import os
from openai import AzureOpenAI
import gradio as gr
from dotenv import dotenv_values
import time
from datetime import timedelta
import json
from azure.kusto.data import KustoClient, KustoConnectionStringBuilder, ClientRequestProperties
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
```

- Load the environment variables

```
config = dotenv_values("env.env")
```

- COnfigure open ai client

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)
```

- Create Kusto cluster information

```
cluster = config["KUSTO_URL"]
# In case you want to authenticate with AAD application.
client_id = config["KUSTO_CLIENTID"]
client_secret = config["KUSTO_SECRET"]

# read more at https://docs.microsoft.com/en-us/onedrive/find-your-office-365-tenant-id
authority_id = config["KUSTO_TENANTID"]
```

- Invoke the kusto client

```
kcsb = KustoConnectionStringBuilder.with_aad_application_key_authentication(cluster, client_id, client_secret, authority_id)

kclient = KustoClient(kcsb)
```

- function to get followup questions

```
def get_followup_questions(response,firstllm, message):
    returntext = ""
    system_prompt = """You are Kusto KQL agent. KQL Query generated was not correct. \n\n
    Provide suggestion questions to ask in the data. \n
    User: What are the top 10 stocks? \n\n
    Kusto KQL Agent:"""

    followup = f"Create Follow up questions or Suggestion based on user ask: {response}"

    messages = [
        { "role": "system", "content": system_prompt },
        { "role": "user", "content":  followup},
    ]

    #print("Followup: ",str(messages))

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=firstllm,
        #messages=history_openai_format,
        messages=messages,
        seed=42,
        temperature=0.0,
        #stream=True
    )

    #print("KQL query created: ", response.choices[0].message.content)

    returntext = response.choices[0].message.content

    return returntext
```

- Function to process intents

```
def get_intent_to_process(response,firstllm, message):
    returntext = ""
    system_prompt = """You are given a questionfrom customer. It is your job to identify
    the intent based on the question. Possible intents can be: 
    "Stocks", "CarsTraffic", "Consumption","Costs","DetectiveCases","PhoneCalls","StolenCars","Sales", "general question", "entertainment", "finance", "Marketing", "Human Resource", "IT Support", "other".
    Here are more information about Tables and schema available to use: \n
        Stocks(sdate: datetime, open: real, high: real, low: real, close: real, volume: int, symbol: string) #this table has stocks information based on symbols. \n
    CarsTraffic (Timestamp: datetime, VIN: string, Ave: int, Street: int) - #Table has information about cars traveeling across streets in a city \n
    Consumption (Timestamp: datetime, HouseholdId: string, MeterType: string, Consumed: real) #This table has information about electricity and water usage of households on daily basis. \n
    Costs (MeterType: string, Unit: string, Cost: real) # This table only has information about unit cost and no history data.\n
    DetectiveCases (Timestamp: datetime, EventType: string, DetectiveId: string, CaseId: string, Properties: dynamic) #has information about how cases were solved by detectives in a police station. \n
    PhoneCalls (Timestamp: datetime, EventType: string, CallConnectionId: string, Properties: dynamic)  #this table has call logs meta data, like who called and to whom.\n
    StolenCars (VIN: string) # This table only has stolen cars VIN numbers and no historical data. \n

    Try to identify the intent based on the question and provide only intent as output.
    Try to proritize the intent based on tables and schema available. \n
    Provide only intent as output.
    Question:\n\n"""

    messages = [
        { "role": "system", "content": system_prompt },
        { "role": "user", "content": response },
    ]

    #print("Intent Messages: ",str(messages))

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=firstllm,
        #messages=history_openai_format,
        messages=messages,
        seed=42,
        temperature=0.0,
        #stream=True
    )

    #print("KQL query created: ", response.choices[0].message.content)

    returntext = response.choices[0].message.content

    return returntext
```

- now function to get data

```
def get_data_from_kusto(response,firstllm, message):
    returntext = ""
    start_time = time.time()
    system_prompt = """You are Kusto KQL agent. Understand the question and provide by the user and create a syntactically correct Kusto KQL query to get the data. \n\n
    Here is the schema for tables provided: \n
    Schema: \n
    Stocks(sdate: datetime, open: real, high: real, low: real, close: real, volume: int, symbol: string) #this table has stocks information based on symbols. \n
    CarsTraffic (Timestamp: datetime, VIN: string, Ave: int, Street: int) - #Table has information about cars traveeling across streets in a city \n
    Consumption (Timestamp: datetime, HouseholdId: string, MeterType: string, Consumed: real) #This table has information about electricity and water usage of households on daily basis. \n
    Costs (MeterType: string, Unit: string, Cost: real) # This table only has information about unit cost and no history data.\n
    DetectiveCases (Timestamp: datetime, EventType: string, DetectiveId: string, CaseId: string, Properties: dynamic) #has information about how cases were solved by detectives in a police station. \n
    PhoneCalls (Timestamp: datetime, EventType: string, CallConnectionId: string, Properties: dynamic)  #this table has call logs meta data, like who called and to whom.\n
    StolenCars (VIN: string) # This table only has stolen cars VIN numbers and no historical data. \n
    Based on the table information on what the table has, try to formulate the correct Kusto KQL query. \n
    Convert string columns add lower case: tolower(columnname) and build the query. \n
    If the filter is string or text based, convert to data in the table to lower case (tolower()) and build the query. 
    For Example Water converted to water. for Example MeterType to tolower(MeterType)\n
    Return only KQL query, don't add anything else. \n
    If KQL cannot be created then return followup questions to the user. \n
    Provide suggestion questions to ask in the data. \n
    User: What are the top 10 stocks? \n\n
    Kusto KQL Agent:"""

    messages = [
        { "role": "system", "content": system_prompt },
        { "role": "user", "content": message.lower() },
    ]

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=firstllm,
        #messages=history_openai_format,
        messages=messages,
        seed=42,
        temperature=0.0,
        #stream=True
    )

    print("KQL query created: ", response.choices[0].message.content)
    partial_message = ""
    returntext = ""
    # calculate the time it took to receive the response
    response_time = time.time() - start_time
    # returntext = response.choices[0].message.content + f" \nTime Taken: ({response_time:.2f} seconds)"

    # once authenticated, usage is as following
    db = "pinballdata"
    #query = "Stocks | take 10"
    query = response.choices[0].message.content.replace("```","").replace("kql","").replace("Kusto KQL Agent:","").replace("kusto","").strip()

    print("Query: ", query)

    #responsekql = kclient.execute(db, query)

    try:
        responsekql = kclient.execute(db, query)
        # iterating over rows is possible
        # for row in responsekql.primary_results[0]:
        #     # printing specific columns by index
        #     #print("value at 0 {}".format(row[0]))
        #     #print("\n")
        #     rowtxt = f"Stock Information: {row[0], row[1], row[2], row[3], row[4], row[5]} \n"
        #     # printing specific columns by name
        #     # print("EventType:{}".format(row["sdate"]))
        #     #print(rowtxt)
        #     #returntext = returntext + rowtxt + + f" \nTime Taken: ({response_time:.2f} seconds)"
        #     returntext = f"{returntext} \n Data: {rowtxt} \n\n Time Taken: ({response_time:.2f} seconds)"

        #mktext = make_markdown_table_kusto(responsekql.primary_results[0])
        
        returntext = f"{returntext} \n\n KQL Data: {str(responsekql.primary_results[0])} \n\n Time Taken: ({response_time:.2f} seconds)"
    except KustoServiceError as error:
        print("2. Error:", error)
        Followuptext = get_followup_questions(query,firstllm, message)
        #returntext = returntext + "\n" + Followuptext + + f" \nTime Taken: ({response_time:.2f} seconds)"
        returntext = f"{returntext} \n\n Followup Questions: {Followuptext} \n\n Time Taken: ({response_time:.2f} seconds)"
        print("2. Is semantic error:", error.is_semantic_error())
        print("2. Has partial results:", error.has_partial_results())
        #print("2. Result size:", len(error.get_partial_results()))

    return returntext
```

- create a markdown table

```
def make_markdown_table_kusto(tables):
    markdown = ""
    for row in tables:
        markdown += "|"
        markdown += f" {str(row[0])} | {str(row[1])} | {str(row[2])} | {str(row[3])} | {str(row[4])} | {str(row[5])} \n"
        #markdown += "\n"

    return markdown
```

- Now write the predict function to process the question and get the relevant answers

```
def predict(message, history, firstllm, system_prompt, prompt_1):
    history_openai_format = []
    history_openai_format.append({"role": "system", "content": system_prompt })
    for human, assistant in history:
        history_openai_format.append({"role": "user", "content": human })
        history_openai_format.append({"role": "assistant", "content":assistant})
    history_openai_format.append({"role": "user", "content": message})

    start_time = time.time()

    print('model:', firstllm)

    intresponse = ""

    try:
        intresponse = get_intent_to_process(message,firstllm, message)
        print('Intent response: ', intresponse)

        #intenttext = json.load(intresponse)     

        #print("Intent Identified: ", intenttext[0])
        print("Intent Identified: ", intresponse.replace("Intent:","").strip())
    except Exception as e:
        print("Error: ", e)
        intenttext = "Error"

    kustotbl = ["Stocks", "CarsTraffic", "Consumption","Costs","DetectiveCases","PhoneCalls","StolenCars"]

    if(intresponse != ""):
        if(intresponse.startswith("Sales")):
            #returntext = get_data_from_kusto(message,firstllm, message) 
            returntext = "Sales"
        elif (intresponse.startswith(("Stocks", "CarsTraffic", "Consumption","Costs","DetectiveCases","PhoneCalls","StolenCars"))):
            returntext = get_data_from_kusto(message,firstllm, message)
        elif(intresponse.startswith("general")):
            #returntext = get_data_from_kusto(message,firstllm, message) 
            returntext = "General"
        elif(intresponse.startswith("entertainment")):
            #returntext = get_data_from_kusto(message,firstllm, message) 
            returntext = "Enterntainment"
        elif(intresponse.startswith("IT")):
            #returntext = get_data_from_kusto(message,firstllm, message) 
            returntext = "IT Support"
        else:
            returntext = "Unable to find intent. Please rephrase your question and try. Also here is information from my own knowledge \n"
            returntext += "Information provided might be incorrect. Please verify with correct source. \n"
            returntext += default_message(message, history, firstllm, system_prompt, prompt_1)
        #returntext = intresponse

    

    response_time = time.time() - start_time

    print(f"Full response received {response_time:.2f} seconds after request")

    #return response.choices[0].message.content + f" \nTime Taken: ({response_time:.2f} seconds)"
    return returntext
```

- now create the UI
- Provide sample questions to select from
- Drop down provides questions, we can select and then copy and paste inside the chat box

```
with gr.Blocks() as demo:
    with gr.Accordion("See Details"):
        gr.Markdown(
            r"Welcome to the Kusto Chatbot. This chatbot is designed to help you with Kusto KQL queries. Gpt 4 Turbo provides better outcomes."
        )
    with gr.Accordion("Help", open=False):
        gr.Markdown(
            r"Select one question and copy and paste to right side chat text box. <br> Select the Model as gpt 4 turbo for better outcomes."
        )

    with gr.Row():
        with gr.Column(scale=1, min_width=100): 
            qn = gr.Dropdown(
                    [ 
                        "None",
                        "What was the open and close Stock price for AAPL?","which avenues and streets are we seeing stolen cars passed?",
                        "show me top 10 rows for water consumption","what was the total water consumption on april 13th 2023?",
                        "what is the total electricity used for april 14 2023?",
                        "show me top 10 phone calls?","Show me the recent 10 calls from 06641258685 as origin?",
                        "Show me the recent 10 calls from 06439235296 as destination?",
                        "Show me recent 10 calls from origin as 06641258685 and destination as 06340109669",
                        "How many cases assigned in dec 22 2022?","who are top 5 detectives solved cases?"
                    ], label="questions", info="Select questions to answer!"
                )    
            ctinput = gr.Textbox("", label="QuestionSample")
            qn.select(Dropdown_list, inputs=qn, outputs=ctinput)

        with gr.Column(scale=2, elem_classes=["container"], min_width=500):
            chatbot = gr.ChatInterface(predict, title="LLM Model", fill_height=True, additional_inputs=[gr.Dropdown(
                ["gpt-35-turbo", "gpt-4-turbo", "gpt-35-turbo-instruct", "llama27b"], label="firstllm"
                ),gr.Textbox("You are helpful AI.", label="System Prompt"),]
            , ).queue()
    
if __name__ == "__main__":
    demo.launch()
```

- Now run the code and test the chatbot

```
python kustochatui.py
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/GradioUI/images/kustochatui1.jpg 'RagChat')