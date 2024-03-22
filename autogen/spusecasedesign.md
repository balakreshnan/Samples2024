# Designing a software product for a specific use case using microsoft autogen

## Introduction

- Build a product for a specific use case
- I am using Supply chain contracts as use case
- Idea here is to have multiple persona to interact with each other
- Business onwer provides what they want to build
- Architect designs the product
- Product owner manages the product
- Concept to have software product brain storming agent

# Pre-requisites

- Azure Subscription
- Azure machine learning service
- Python 3.10 or greater
- microsoft autogen - pyautogen sdk

```
Note: please remember this is just imagination and not real story
```

## Steps/Code

- First install pyautogen

```
%pip install pyautogen
```

- import autogen

```
import autogen
```

- Load environment variables

```
from dotenv import dotenv_values
# specify the name of the .env file name 
env_name = "env.env" # change to use your own .env file
config = dotenv_values(env_name)
```

- Load open ai and it's configuration

```
import openai 

openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_API_KEY"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = config["AZURE_OPENAI_API_VERSION"]
openai.base_url = config["AZURE_OPENAI_ENDPOINT"]
```

- Open AI version used : 2024-02-01

- configure llm

```
llm_config={"config_list": [
    {"model": "gpt-4-turbo", "api_key": config["AZURE_OPENAI_API_KEY"], 
    "cache_seed" : None, "base_url" : config["AZURE_OPENAI_ENDPOINT"],
    "api_type" : "azure", "api_version" : "2024-02-01"}
    ]}
```

- load autogen assistant library

```
from typing_extensions import Annotated
#import chromadb

import autogen
from autogen import AssistantAgent
```

- set the termination function

```
def termination_msg(x):
    return isinstance(x, dict) and "TERMINATE" == str(x.get("content", ""))[-9:].upper()
```

- Now we build 3 persona: product owner, business owner and architect

```
productowner = autogen.UserProxyAgent(
    name="ProductOwner",
    is_termination_msg=termination_msg,
    human_input_mode="NEVER",
    code_execution_config=False,  # we don't want to execute code in this case.
    default_auto_reply="Reply `TERMINATE` if the task is done.",
    description="The boss who ask creates features and prioritize tasks. Come up with new features and method to solve the issue.",
)
```

- Architect

```
Architect = AssistantAgent(
    name="Architect",
    is_termination_msg=termination_msg,
    system_message="You are a Lead Architect, Ask followup questions to understand the business process and persona who will use the system and provide simple high level architecture. Reply `TERMINATE` in the end when everything is done.",
    llm_config=llm_config,
    description="I am Lead Architect and my job is to understand the use case from business owner and then design the technology pieces needed.",
)
```

- Business Owner

```
BusinessOwner = autogen.AssistantAgent(
    name="BusinessOwner",
    is_termination_msg=termination_msg,
    system_message="You are a business owner who works with customer to understand their use case and requirements and build new solutions using Generative AI. Reply `TERMINATE` in the end when everything is done.",
    llm_config=llm_config,
    description="I am business owner who want to solve complex customer problem",
)
```

- Now we have 3 persona ready to interact with each other

```
PROBLEM = "Create a Supply chain co pilot agent to answer question on supplier contracts and their delivery times."
```

- Now we will start the conversation

```
groupchat = autogen.GroupChat(
    agents=[productowner, BusinessOwner, Architect], messages=[], max_round=10, speaker_selection_method="round_robin"
)
manager = autogen.GroupChatManager(groupchat=groupchat, llm_config=llm_config)

# Start chatting with boss_aid as this is the user proxy agent.
result = productowner.initiate_chat(
        manager,
        message=PROBLEM,
    n_results=5,
)
```

```
for content in result.chat_history:
    print(content['content'])
```

- Here is the output of the conversation

```
Create a Supply chain co pilot agent to answer question on supplier contracts and their delivery times.
Designing a supply chain co-pilot agent that provides information on supplier contracts and delivery times involves several steps. This solution would leverage Generative AI to interpret user queries and generate accurate, context-aware responses based on the contract data and delivery schedules. Here's a high-level overview of how we can create such a system:

1. **Requirements Gathering**:
   - Identify the types of questions the co-pilot must answer.
   - Determine the data sources for supplier contracts and delivery information.

2. **Data Integration**:
   - Set up a secure method to access and retrieve information from your existing supply chain database or ERP system.
   - Ensure real-time or periodic data synchronization for accuracy.

3. **Model Selection**:
   - Choose a Generative AI model well-suited for natural language processing (NLP), such as GPT-3, BERT, or a custom-trained model, depending on the complexity and specificity of the data.

4. **Training the Model**:
   - If a custom model is required, collect and annotate a dataset of supply chain interactions and contract details.
   - Train the model to understand and respond to queries related to supplier contracts and delivery times.

5. **Integration with User Interface (UI)**:
   - Develop a UI for users to interact with the co-pilot, which could be a web portal or an internal application.
   - Ensure the UI is user-friendly and accessible to applicable departments.

6. **Access Control Implementation**:
   - Set permissions to control who can access the co-pilot agent.
   - Protect sensitive contract details and ensure compliance with data protection laws.

7. **Programming the Co-pilot Agent**:
   - Program the co-pilot agent to extract and serve information from the integrated data sources.
   - Implement natural language processing capabilities for understanding and responding to user queries.

8. **Testing and Iteration**:
   - Conduct comprehensive testing with a range of queries to ensure accuracy and reliability.
   - Collect feedback from users and make necessary adjustments to the agent's responses and functionality.

9. **Deployment**:
   - Deploy the agent onto a suitable platform where it's readily accessible to users.
   - Make sure it's scalable and capable of handling the expected query volume.

10. **User Training and Documentation**:
    - Provide training for users to get accustomed to working with the co-pilot.
    - Create documentation outlining how to use the agent, including examples of questions it can answer.

11. **Maintenance and Updates**:
    - Set up a system for ongoing maintenance and regular updates to the agent as contracts and supply chain dynamics change.
    - Monitor interactions and regularly re-train the AI model with new data to ensure continued performance and relevance.

12. **Feedback Loop Implementation**:
    - Gather user feedback for continuous improvement.
    - Refine the AI's responses and improve understanding of the model over time.

By following these steps, you'd establish a robust supply chain co-pilot tailored to your organization's needs. Of course, the complexities and specifics could vary, and more technical details would need to be ironed out during development. But this outline should give you a solid foundation for creating a functional AI-driven supply chain assistant.

Is there a particular area within this plan that you'd like to explore further or require specific assistance with?
To better understand your needs for the supply chain co-pilot agent, I have several follow-up questions:

1. **User Personas:**
   - Who will be the primary users of the co-pilot agent? (e.g., procurement officers, supply chain managers, etc.)
   - What level of technical proficiency do these users possess?

2. **User Interaction:**
   - How do users currently get information about supplier contracts and delivery times?
   - What pain points exist in the current process that the co-pilot agent should address?

3. **Data Sources:**
   - Can you provide details about the data sources that contain supplier contract information and delivery times?
   - How is this data structured, and is it centrally managed or distributed across multiple systems?

4. **Integration Considerations:**
   - Are there any specific enterprise systems or platforms (e.g., SAP, Oracle) the co-pilot agent needs to integrate with?
   - Do you have any preferences or requirements for cloud services or on-premises deployment?

5. **Security and Compliance:**
   - What security measures need to be in place to protect contract information?
   - Are there any industry-specific compliance standards that we need to adhere to with this system?

6. **User Interface Expectations:**
   - What are your expectations for the user interface of the system? Do you prefer a chatbot style interaction, an email-based system, or another mode of interaction?

7. **Performance Metrics:**
   - How will the performance of the co-pilot agent be measured?
   - What are the key performance indicators (KPIs) that you want to track?

8. **Scalability and Maintenance:**
   - How do you foresee the volume of inquiries scaling over time?
   - What are your expectations for system maintenance and updates?

Understanding your answers to these questions will help to ensure that the high-level architecture of the supply chain co-pilot agent will align with your business needs and user expectations.
Reply `TERMINATE` if the task is done.
TERMINATE
```