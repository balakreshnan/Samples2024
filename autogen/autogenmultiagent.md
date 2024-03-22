# Microsoft AutoGen building multi agent story generation with Azure Machine Learning

## Introduction

- Build a multi-agent story generation framework
- Using autogen to create a fictional story
- This is only imagination, nothing article created is true
- Only Experimental to show case multi agent system how they work
- Use Azure Machine Learning as Development environment
- Using compute instance to do coding

## Pre-requisites

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

- now i am creating persona like commissioner, detective, and suspect

```
commisioner = autogen.UserProxyAgent(
    name="commisioner",
    is_termination_msg=termination_msg,
    human_input_mode="NEVER",
    code_execution_config=False,  # we don't want to execute code in this case.
    default_auto_reply="Reply `TERMINATE` if the task is done.",
    description="The boss who ask questions and give tasks. He is also great detective to analyze and ask for followup.",
)
```

- now detective

```
Detective = AssistantAgent(
    name="Detective",
    is_termination_msg=termination_msg,
    system_message="You are a senior Detective, introgate the suspect. Reply `TERMINATE` in the end when everything is done.",
    llm_config=llm_config,
    description="I am Lead detective and my job is to introgate the suspect and find answers..",
)
```

- now suspect

```
Suspect = autogen.AssistantAgent(
    name="Suspect",
    is_termination_msg=termination_msg,
    system_message="You are a thief and police are introgating. Reply `TERMINATE` in the end when everything is done.",
    llm_config=llm_config,
    description="suspect who took money from bank and ran away.",
)
```

- here is the problem statment

```
PROBLEM = "Suspect robbed a bank and took around 100000 dollars worth of cash."
```

- now we will start the conversation
- Enable group chat

```
groupchat = autogen.GroupChat(
    agents=[commisioner, Detective, Suspect], messages=[], max_round=5, speaker_selection_method="round_robin"
)
manager = autogen.GroupChatManager(groupchat=groupchat, llm_config=llm_config)

# Start chatting with boss_aid as this is the user proxy agent.
result = commisioner.initiate_chat(
        manager,
        message=PROBLEM,
    n_results=3,
)
```

- Now we will see the conversation

```

print(result.chat_history[-1]['content'])
```

- now here is the output. Only for fun

```
All right, I understand the suspect is in custody for the bank robbery. Let's begin the interrogation.

[The scene is set in an interrogation room with a one-way mirror on the wall. There's a table and chairs, one of which is occupied by the suspect. I enter the room and sit down across from the suspect.]

Detective: "Good [morning/afternoon/evening], I am Detective [Your Name]. I'm sure you know why you're here, but for the record, you're being held as a suspect in the robbery of [Bank Name], where $100,000 was taken. Let's start with the basics. Can you tell me your name and where you were during the time the robbery took place?"

[Wait for response. The interrogation would continue with questions designed to assess the suspect's alibi, presence at the scene, and involvement in the crime.]

Detective: "Do you understand the charges that are being brought against you? Taking that amount of money through a bank robbery is a serious offense, and we have ample evidence to pursue a conviction. But we're interested in your side of the story. Why don't you tell me where you were and what you were doing at the time of the robbery?"

[Wait for suspect's answer and observe their behavior, verbal and non-verbal cues.]

Detective: "We have witnesses who have identified someone matching your description at the scene. Can you explain that?"

[Assess suspect's response.]

Detective: "Is there anyone who can confirm your whereabouts? We need someone who can vouch for you if you’re saying you weren't involved."

[Listen to any potential alibi provided.]

Detective: "We also found fingerprints at the scene that we are running through our system. If your fingerprints match, it's going to be very challenging to maintain your innocence. Now would be a good time to start being honest with us if you have anything you want to share."

[Observe for any signs of stress or admission of guilt.]

Detective: "Security cameras from the bank and the surrounding area have provided us with footage. Are we going to see you on those cameras?"

[Wait for response.]

Detective: "Look, we can do this the easy way or the hard way. The evidence will speak for itself in the end, but cooperation goes a long way in these situations. We are trying to piece everything together, and if you were involved, it would benefit you to tell us now."

[Offer another opportunity for the suspect to come forward with any information.]

Detective: "We're also looking into the possibility that this wasn’t a one-person job. Were you working with anyone else? If so, it might be in your best interest to let us know before your accomplices decide to talk and potentially leave you holding the bag."

[Assess the suspect's reaction and willingness to cooperate.]

Detective: "Alright, if you’re not ready to speak now, that's your choice. But we'll continue our investigation, and the evidence will lead us to the truth. You will be kept here while we further our work and until your legal situation is resolved. If you decide you want to talk, let the officers know. We will be available to listen to what you have to say."

[Stand up, signaling the end of the interrogation.]

Detective: "Remember, it's in your best interest to work with us. We will find out what happened, with or without your help. Thank you for your time."

[Exit the interrogation room.]

TERMINATE
```