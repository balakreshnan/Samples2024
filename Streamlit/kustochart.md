# Ability to create Kusto KQL output with D3 Charts

## Introduction

- Use Case to create charting based on question asked in Kusto Query Language
- Natural language to Kusto Query Language
- From Kusto output create Javascript D3 Charts
- Then display that in Stream lit application

## pre-requisites

- Azure Account
- Azure data explorer cluster
- Load sample dataset
- Visual Studio Code
- Python
- Streamlit
- Azure open ai
- Azure data explorer sdk
- D3 Javascript library

## Code

- import libraries

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
import streamlit as st

config = dotenv_values("env.env")

css = """
.container {
    height: 75vh;
}
"""


```

- COnfigure Azure Open AI

```
client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)
```

- Setup Kusto Connection

```
cluster = config["KUSTO_URL"]

# In case you want to authenticate with AAD application.
client_id = config["KUSTO_CLIENTID"]
client_secret = config["KUSTO_SECRET"]

# read more at https://docs.microsoft.com/en-us/onedrive/find-your-office-365-tenant-id
authority_id = config["KUSTO_TENANTID"]

kcsb = KustoConnectionStringBuilder.with_aad_application_key_authentication(cluster, client_id, client_secret, authority_id)

kclient = KustoClient(kcsb)
```

- create the following functions

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

- Create the function to create kusto query and execute

```
def get_data_from_kusto(firstllm, message):
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
    Convert date to yyyy-mm-dd format. Use format_datetime(dt, "yyyy-MM-dd"). for example sdate to format_datetime(sdate, "yyyy-MM-dd")\n
    Date field could also be called sdate or timestamp. \n
    Return only KQL query, don't add anything else. \n
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
        
        #returntext = f"{returntext} \n\n KQL Data: {str(responsekql.primary_results[0])} \n\n Time Taken: ({response_time:.2f} seconds)"
        returntext = responsekql.primary_results[0]
        #print("1. Result:", responsekql.primary_results[0])
    except KustoServiceError as error:
        print("2. Error:", error)
        #Followuptext = get_followup_questions(query,firstllm, message)
        #returntext = returntext + "\n" + Followuptext + + f" \nTime Taken: ({response_time:.2f} seconds)"
        #returntext = f"{returntext} \n\n Followup Questions: {Followuptext} \n\n Time Taken: ({response_time:.2f} seconds)"
        print("2. Is semantic error:", error.is_semantic_error())
        print("2. Has partial results:", error.has_partial_results())
        #print("2. Result size:", len(error.get_partial_results()))

    return returntext
```

- Create the D3 Chart function

```
def create_d3_chart(data, selected_optionmodel, message, selected_optioncharttype):
    htmloutput = """
    """
    start_time = time.time()
    system_prompt = f"""You are D3 JS Chart Expert in javascript. Provide only the D3 code without any explanation.
    Adjust the height and width to show the chart properly.
    Show axis labels, values and title for the chart. Also show series name in the legend.
    Do Not provide any explanation or comments in the code. Use the dataset provide below.
    Create code for D3 version 6.0.0
    Only use the data provided. if can't find data then respond with no data found.
    Use Camel case for javascript functions.
    Create D3 code for version 6.0.0 and above
    Use Javascript for parsing date using Date function and format the date to yyyy-mm-dd format.

    Sources:
    {data}
    """

    #message = f"Create a {selected_optioncharttype} chart plot for stocks for 6 months of tesla?"
    #message1 = f"Create a {selected_optioncharttype} chart for {message}"
    #message1 = f"{message}"
    message1 = f"Create a {selected_optioncharttype} chart."

    print('Chart selected: ' , message1)

    messages = [
        { "role": "system", "content": system_prompt },
        { "role": "user", "content": message1.lower() },
    ]

    response = client.chat.completions.create(
        #model= "gpt-35-turbo", #"gpt-4-turbo", # model = "deployment_name".
        model=selected_optionmodel,
        #messages=history_openai_format,
        messages=messages,
        seed=42,
        temperature=0.0,
        max_tokens=2000,
        top_p=1.0,
        #stream=True
    )

    #print("KQL query created: ", response.choices[0].message.content)
    
    #htmloutput = """<script> 
    #response.choices[0].message.content
    #</script>"""
    htmloutput = response.choices[0].message.content.replace("```","").replace("Javascript","").replace("javascript","").strip()

    return htmloutput
```

- now let's put all together in Streamlit

```
def main():
    st.title("Analytics apps to create D3.js charts using OpenAI")
    # Load D3.js script
    #d3_js = """
    #<script src="https://d3js.org/d3.v6.min.js"></script>
    #<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    #"""

    #st.components.v1.html(d3_js, height=100)

    # Split the app layout into two columns
    col1, col2 = st.columns(2)

     # HTML content for column 1
    with col1:
        charttype = ["line", "area", "bar", "scatter", "pie", "donut", "bubble", "heatmap", "radar", "treemap", 
                "choropleth", "table", "gauge", "funnel", "pyramid", "sankey", "sunburst", "network", 
                "wordcloud", "calendar", "pictorial", "radialbar", "venn", "timeline", "bullet", "boxplot", 
                "histogram", "density", "contour", "candlestick", "ohlc", "waterfall", "funnelarea", "pyramidarea"]
        
        selected_optioncharttype = st.selectbox("Select an Chart Type:", charttype)

        modeloptions = ["gpt-35-turbo", "gpt-4-turbo", "llama2", "mixstral"]

        # Create a dropdown menu using selectbox method
        selected_optionmodel = st.selectbox("Select an Model:", modeloptions)
        
        # Get user input
        user_input = st.text_input("Enter HTML content:")

        # Define a set of values for the dropdown menu
        options = ["Show me top 10 open and close Stock price for AAPL?",
                "Show me top 10 open and close Stock price for MSFT?",
                    "What was the open and close Stock price for AAPL?","which avenues and streets are we seeing stolen cars passed?",
                    "show me top 10 rows for water consumption","what was the total water consumption on april 13th 2023?",
                    "what is the total electricity used for april 14 2023?",
                    "show me top 10 phone calls?","Show me the recent 10 calls from 06641258685 as origin?",
                    "Show me the recent 10 calls from 06439235296 as destination?",
                    "Show me recent 10 calls from origin as 06641258685 and destination as 06340109669",
                    "How many cases assigned in dec 22 2022?","who are top 5 detectives solved cases?"
                ]

        # Create a dropdown menu using selectbox method
        selected_option = st.selectbox("Select an option:", options)

        # Display the selected option
        st.write("You selected:", selected_option)

    # HTML content for column 2
    with col2:
        returntext = get_data_from_kusto(selected_optionmodel, selected_option)
        htmloutput = create_d3_chart(returntext, selected_optionmodel, selected_option, selected_optioncharttype)
        
        #print("Return Text: ", htmloutput)
        # Display HTML content
        st.write("HTML Output:")
        # st.write(user_input, unsafe_allow_html=True)
        htmltxt = f"""
        <html>
        <body>
        <div id="my_dataviz"></div>
        </body>
        </html>
        """
        # st.components.v1.html(htmltxt)

        # st.write(returntext, unsafe_allow_html=True)
            # JavaScript code for creating the chart
        chart_js = f"""
        <div id="my_dataviz"></div>
        <script src="https://d3js.org/d3.v6.min.js"></script> 
        <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
        <script>
            {htmloutput}
        </script>    
        """

        print("Javascript Text: ", chart_js)

        # Embed HTML content with JavaScript code for the chart
        st.components.v1.html(chart_js, height=350, width=700, scrolling=True)

    #st.components.v1.html(
    #"""
    #<div id="chart"></div>
    #<script src="https://d3js.org/d3.v6.min.js"></script>
    #<script>
    #    // D3.js code to create a simple bar chart
    #    var data = [4, 8, 15, 16, 23, 42];
    #    var svg = d3.select("#chart")
    #                .append("svg")
    #                .attr("width", 400)
    #                .attr("height", 200);
    #
    #    svg.selectAll("rect")
    #        .data(data)
    #        .enter().append("rect")
    #        .attr("x", function(d, i) { return i * 50; })
    #        .attr("y", function(d) { return 200 - d * 5; })
    #        .attr("width", 40)
    ##        .attr("height", function(d) { return d * 5; })
    #        .attr("fill", "steelblue");
    #</script>
    #"""
    #)

    #st.write(htmloutput, unsafe_allow_html=True)

if __name__ == "__main__":
    main()
```

- Run the Streamlit application

```
streamlit run kustochart.py
```

- now try few options

![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/kustochart1.png'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/kustochart2.png'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/kustochart3.png'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/Streamlit/images/kustochart4.png'RagChat')