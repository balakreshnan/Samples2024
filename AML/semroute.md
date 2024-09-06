# Semnatic kernel dynamically selecting plugin to execute

## Use Case

- Built a multi tier system to select a product
- Within the product which business unit does the input belong to
- Idea here is to understand what customer's are asking and provide the right answer
- So we need to understand what is the context of the question
- Which products they are talking about and provide that information

## Pre-requisite

- Azure Subscription
- Azure Open AI
- Semantic Kernel
- Python 3.11
- Visual Studio Code

## Plugin Code

- Create a plugin folder
- Create a folder under plugin as ProductSelector, Product selector will select what product is talked about.
- Create a folder under plugin as Selector - This is the main one which decides which plugin to execute
- Create a folder under plugin as Sales - This is the plugin which will provide the sales information
- Create a folder under plugin as TechnicalSupport - This is the plugin which will provide the technical support information
- Each Plugin will have 2 files

## ProductSelector

- Config.json

```
{
    "schema": 1,
    "description": "Product Selection based on the input",
    "execution_settings": {
      "default": {
        "max_tokens": 1000,
        "temperature": 0.9,
        "top_p": 0.0,
        "presence_penalty": 0.0,
        "frequency_penalty": 0.0
      }
    },
    "input_variables": [
      {
        "name": "input",
        "description": "product Information",
        "default": ""
      },
      {
        "name": "style",
        "description": "Provide the precise style of the product",
        "default": ""
      }
    ]
  }
```

- skprompt.txt

```
Extract the product name from the given text

NO SEXISM, RACISM OR OTHER BIAS/BIGOTRY
ONLY PROVIDE THE PRODUCT NAME, NOT THE WHOLE TEXT

+++++

{{$input}}
+++++
```

## Selector

- Config.json

```
{
    "schema": 1,
    "description": "Select which plugin to execute",
    "execution_settings": {
      "default": {
        "max_tokens": 1000,
        "temperature": 0.9,
        "top_p": 0.0,
        "presence_penalty": 0.0,
        "frequency_penalty": 0.0
      }
    },
    "input_variables": [
      {
        "name": "input",
        "description": "Select plugin subject",
        "default": ""
      },
      {
        "name": "style",
        "description": "Select the plugin to execute based on the input",
        "default": ""
      }
    ]
  }
```

- skprompt.txt

```
SELECT The plugin to execute based on input provided with the list of plugins to select from

Sales
TechnicalSupport

NO SEXISM, RACISM OR OTHER BIAS/BIGOTRY
ONLY PROVIDE THE PRODUCT NAME, NOT THE WHOLE TEXT

RESPOND ONLY THE PLUGIN NAME
+++++

{{$input}}
+++++
```

## Sales

- config.json

```
{
    "schema": 1,
    "description": "Provide Sales Support",
    "execution_settings": {
      "default": {
        "max_tokens": 1000,
        "temperature": 0.9,
        "top_p": 0.0,
        "presence_penalty": 0.0,
        "frequency_penalty": 0.0
      }
    },
    "input_variables": [
      {
        "name": "input",
        "description": "Sales Support subject",
        "default": ""
      },
      {
        "name": "style",
        "description": "Provide instruction on questions asked and solution",
        "default": ""
      }
    ]
  }
```

- skprompt.txt

```
PROVIDE SALES INFORMATION FOR THE QUESTION

NO SEXISM, RACISM OR OTHER BIAS/BIGOTRY
DONT GET INTO PERSONAL ARGUMENTS
IF YOU DONT KNOW THE ANSWER, DONT ANSWER

+++++

{{$input}}
+++++
```

## TechnicalSupport

- config.json

```
{
    "schema": 1,
    "description": "Provide Techincal Support",
    "execution_settings": {
      "default": {
        "max_tokens": 1000,
        "temperature": 0.9,
        "top_p": 0.0,
        "presence_penalty": 0.0,
        "frequency_penalty": 0.0
      }
    },
    "input_variables": [
      {
        "name": "input",
        "description": "Technical Support subject",
        "default": ""
      },
      {
        "name": "style",
        "description": "Provide instruction on questions asked and solution",
        "default": ""
      }
    ]
  }
```

- skprompt.txt

```
PROVIDE SOLUTIONS FOR THE TECHNICAL QUESTION ASKED

NO SEXISM, RACISM OR OTHER BIAS/BIGOTRY
DONT GET INTO PERSONAL ARGUMENTS
IF YOU DONT KNOW THE ANSWER, DONT ANSWER

+++++

{{$input}}
+++++
```

- the above plugins are needed to execute the main code
- Each plugin should be in separate folder

## Code

- Install semantic kernel

```
pip install semantic-kernel
```

- Make sure .env file is created with the following values

```
GLOBAL_LLM_SERVICE=""
AZURE_OPENAI_CHAT_DEPLOYMENT_NAME=""
AZURE_OPENAI_TEXT_DEPLOYMENT_NAME=""
AZURE_OPENAI_EMBEDDING_DEPLOYMENT_NAME=""
AZURE_OPENAI_ENDPOINT=""
AZURE_OPENAI_API_KEY=""
AZURE_AISEARCH_API_KEY=""
AZURE_AISEARCH_URL=""
```

- it's important to keep the variable names as is, other wise resource not found error will be thrown

- Import the libraries

```
import asyncio

from semantic_kernel import Kernel
from semantic_kernel.functions import kernel_function
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
from semantic_kernel.connectors.ai.function_call_behavior import FunctionCallBehavior
from semantic_kernel.connectors.ai.chat_completion_client_base import ChatCompletionClientBase
from semantic_kernel.contents.chat_history import ChatHistory
from semantic_kernel.functions.kernel_arguments import KernelArguments
from dotenv import dotenv_values
import logging
#import LightsPlugin
from typing import Annotated
from semantic_kernel.functions import kernel_function
from semantic_kernel import __version__

__version__
from services import Service
import os

from service_settings import ServiceSettings
import streamlit as st

from semantic_kernel.connectors.ai.open_ai.prompt_execution_settings.azure_chat_prompt_execution_settings import (
    AzureChatPromptExecutionSettings,
)
config = dotenv_values("env.env")
```

- Here is the main code to execute the plugin

```
async def main1():
    from semantic_kernel import Kernel

    kernel = Kernel()

    service_settings = ServiceSettings.create()

    # Select a service to use for this notebook (available services: OpenAI, AzureOpenAI, HuggingFace)
    selectedService = (
        Service.AzureOpenAI
        if service_settings.global_llm_service is None
        else Service(service_settings.global_llm_service.lower())
    )
    print(f"Using service type: {selectedService}")

    # Remove all services so that this cell can be re-run without restarting the kernel
    kernel.remove_all_services()

    service_id = None
    if selectedService == Service.OpenAI:
        from semantic_kernel.connectors.ai.open_ai import OpenAIChatCompletion

        service_id = "default"
        kernel.add_service(
            OpenAIChatCompletion(
                service_id=service_id,
            ),
        )
    elif selectedService == Service.AzureOpenAI:
        from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion

        service_id = "default"
        kernel.add_service(
            AzureChatCompletion(
                service_id=service_id,
            ),
        )

    print(os.getcwd())
    base_directory = os.getcwd()  # Gets the current working directory

    # Combine the base directory with the 'plugins' directory
    plugins_directory = os.path.join(base_directory, 'plugins')

    # Add a plugin (the LightsPlugin class is defined below)
    plugin = kernel.add_plugin(parent_directory=base_directory, plugin_name="plugins")
    # joke_function = plugin["joke"]

    # joke = await kernel.invoke(
    #     joke_function,
    #     KernelArguments(input="time travel to dinosaur age", style="super silly"),
    # )
    # print(joke)
    # st.write(joke.value[0].inner_content.choices[0].message.content)

    product_plugin = plugin["ProductSelector"]
    product = await kernel.invoke(
        product_plugin,
        KernelArguments(input="How is power apps used as generative ai application development", style="super silly"),
    )
    print(product)
    st.write(product.value[0].inner_content.choices[0].message.content)

    select_plugin = plugin["Selector"]
    pluginselect = await kernel.invoke(
        select_plugin,
        KernelArguments(input="How is power apps used as generative ai application development", style="super silly"),
    )
    print(pluginselect)
    st.write(pluginselect.value[0].inner_content.choices[0].message.content)

    # time to run the selected plugin
    sel_plugin = plugin[pluginselect.value[0].inner_content.choices[0].message.content]
    pluginsel = await kernel.invoke(
        sel_plugin,
        KernelArguments(input="How is power apps used as generative ai application development", style="super silly"),
    )
    print(pluginsel)
    st.write(pluginsel.value[0].inner_content.choices[0].message.content)
```

- Run the code

```
def semroute():
    asyncio.run(main1())
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/semroute-1.jpg 'RagChat')