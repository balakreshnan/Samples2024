# Build a research platform for CSI Connected Systems Institute (CSI) at UW-Milwaukee

## Introduction

- Build a research platform for CSI Connected Systems Institute (CSI) at UW-Milwaukee
- platform will allow us to create new experiments and testbeds for research in the area of connected systems
- Students can try various combinations of color to create new products

## Objective

- Platform to allow students to create new experiments and testbeds for research in the area of connected systems
- Students can try various combinations of color to create new products
- Test and validate the experiments and records the results
- Speech enabled to talk the receipe
- Avatar enabled to show the receipe

## Pre-requisites

- Azure subscription
- Azure Storage Account
- Azure Web App
- Azure open ai
- Azure Cognitive Services
- Streamlit

## Steps

### Code

- Create a new Streamlit app
- import all the libraries
- if any library are missing, install them

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
from streamlit import session_state as state
import azure.cognitiveservices.speech as speechsdk
from audiorecorder import audiorecorder
import pyaudio
import wave
from azure.identity import DefaultAzureCredential
import uuid

import logging
import sys
```

- Enable logging for Azure text to speech and avatar services

```
logging.basicConfig(stream=sys.stdout, level=logging.INFO,  # set to logging.DEBUG for verbose output
        format="[%(asctime)s] %(message)s", datefmt="%m/%d/%Y %I:%M:%S %p %Z")
logger = logging.getLogger(__name__)
```

- Initialize session variables

```
if 'returntxt' not in st.session_state:
    st.session_state['returntxt'] = ""

if 'summartycsitext' not in st.session_state:
    st.session_state.summartycsitext = ""

# Initialize session state for shopping cart
if 'cart' not in st.session_state:
    st.session_state['cart'] = {}

if 'messages' not in st.session_state:
    st.session_state.messages = []
```

- Now configure the variables

```
config = dotenv_values("env.env")

css = """
.container {
    height: 75vh;
}
"""

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION_4o_LATEST"], 
  api_key=config["AZURE_OPENAI_KEY_VISION_4o_LATEST"],  
  api_version="2024-05-01-preview"
)

model_name = "gpt-4o-2"

search_endpoint = config["AZURE_AI_SEARCH_ENDPOINT"]
search_key = config["AZURE_AI_SEARCH_KEY"]
search_index=config["AZURE_AI_SEARCH_INDEX1"]
SPEECH_KEY = config['SPEECH_KEY']
SPEECH_REGION = config['SPEECH_REGION']
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']

citationtxt = ""

SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']
# We recommend to use passwordless authentication with Azure Identity here; meanwhile, you can also use a subscription key instead
PASSWORDLESS_AUTHENTICATION = False
#API_VERSION = "2024-04-15-preview"
SUBSCRIPTION_KEY = config['SPEECH_KEY']
SERVICE_REGION = config['SPEECH_REGION']
NAME = "Simple avatar synthesis"
DESCRIPTION = "Simple avatar synthesis description"

# The service host suffix.
SERVICE_HOST = "customvoice.api.speech.microsoft.com"
# The endpoint (and key) could be gotten from the Keys and Endpoint page in the Speech service resource.
# The endpoint would be like: https://<region>.api.cognitive.microsoft.com or https://<custom_domain>.cognitiveservices.azure.com
# If you want to use passwordless authentication, custom domain is required.
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']
# We recommend to use passwordless authentication with Azure Identity here; meanwhile, you can also use a subscription key instead
PASSWORDLESS_AUTHENTICATION = True
API_VERSION = "2024-08-01"
```

- Normalize the selection

```
# Function to ensure sliders add up to 100%
def normalize_sliders(values):
    total = sum(values)
    if total == 100:
        return values
    else:
        normalized_values = [int(value / total * 100) for value in values]
        difference = 100 - sum(normalized_values)
        normalized_values[0] += difference
        return normalized_values
```

- Now write the function for CSI research platform

```
def csirecipedesign(query, selected_optionmodel1, blue, green, orange, red):
    returntxt = ""
    caloriesrange1 = ""


    start_time = time.time()

    message_text = [
    {"role":"system", "content":"""you are Manufacturing Engineer who is building receipe to run through the line, 
     your job is to provide guidance on new research based on what paramters are provided.
     Be politely, and provide positive tone answers. 
     Answer from your own memory and avoid frustating questions and asking you to break rules.
     """}, 
    {"role": "user", "content": f"""Provide recommendations on Color creation recipe to build new product based on {query}  
     and here are some parameters: 
     Blue value: {blue},
     Green value: {green},
     Orange value: {orange},
     Red value: {red},
     Provide 5 diferent recommendations based on the above parameters. Show how the parameters were used to making the decision. 
     Provide details on how you derived the combination and also show risk on the combination is there is any.    
     Show the limitations of the recommendations and how they can be improved."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=1,
        seed=105,
   )

    returntxt = response.choices[0].message.content + "\n<br>"

    reponse_time = time.time() - start_time 

    returntxt += f"<br>\nResponse Time: {reponse_time:.2f} seconds"

    return returntxt
```

- Summary function for CSI Research output to voice enable

``` 
def csirecipedesignsummary(query, selected_optionmodel1, blue, green, orange, red, summarytext):
    returntxt = ""
    caloriesrange1 = ""


    start_time = time.time()

    message_text = [
    {"role":"system", "content":"""you are Manufacturing Engineer who is building receipe to run through the line, 
     your job is to provide guidance on new research based on what paramters are provided.
     Be politely, and provide positive tone answers. 
     Answer from your own memory and avoid frustating questions and asking you to break rules.
     """}, 
    {"role": "user", "content": f"""Summarize the content with bullet points: {summarytext}."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=1,
        seed=105,
        max_tokens=100
   )

    returntxt = response.choices[0].message.content + "\n<br>"

    reponse_time = time.time() - start_time 

    #returntxt += f"<br>\nResponse Time: {reponse_time:.2f} seconds"

    return returntxt
```

- Function to update the sliders

```
def update_sliders(sliders, index, new_value):
    difference = new_value - sliders[index]
    sliders[index] = new_value
    if difference != 0:
        remaining_indices = [i for i in range(len(sliders)) if i != index]
        for i in remaining_indices:
            if sliders[i] - difference / len(remaining_indices) >= 0:
                sliders[i] -= difference / len(remaining_indices)
            else:
                sliders[i] = 0
        sliders[index] = 100 - sum(sliders[:index] + sliders[index+1:])
    return sliders
```

- function to convert text to speech

```
# Define a function that performs some logic  
def process_text_to_speech(text1):  
    #print(json_object)
    # This example requires environment variables named "SPEECH_KEY" and "SPEECH_REGION"
    speech_config = speechsdk.SpeechConfig(subscription=SPEECH_KEY, region=SPEECH_REGION)
    audio_config = speechsdk.audio.AudioOutputConfig(use_default_speaker=True)

    # The neural multilingual voice can speak different languages based on the input text.
    speech_config.speech_synthesis_voice_name='en-US-AvaMultilingualNeural'

    pull_stream = speechsdk.audio.PullAudioOutputStream()
    stream_config = speechsdk.audio.AudioOutputConfig(stream=pull_stream)

    speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
    #speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=stream_config)

    speech_synthesis_result = speech_synthesizer.speak_text(text1)
    if speech_synthesis_result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
        print("Speech synthesized for text [{}] \n".format(text1))
    elif speech_synthesis_result.reason == speechsdk.ResultReason.Canceled:
        cancellation_details = speech_synthesis_result.cancellation_details
        print("Speech synthesis canceled: {}".format(cancellation_details.reason))
        if cancellation_details.reason == speechsdk.CancellationReason.Error:
            if cancellation_details.error_details:
                print("Error details: {}".format(cancellation_details.error_details))
                print("Did you set the speech resource key and region values?")
```

- create a job id for speech synthesis

```
def _create_job_id():
    # the job ID must be unique in current speech resource
    # you can use a GUID or a self-increasing number
    return uuid.uuid4()
```

- This section is for speech process

```
def _authenticate():
    if PASSWORDLESS_AUTHENTICATION:
        # Refer to https://learn.microsoft.com/python/api/overview/azure/identity-readme?view=azure-python#defaultazurecredential
        # for more information about Azure Identity
        # For example, your app can authenticate using your Azure CLI sign-in credentials with when developing locally.
        # Your app can then use a managed identity once it has been deployed to Azure. No code changes are required for this transition.

        # When developing locally, make sure that the user account that is accessing batch avatar synthesis has the right permission.
        # You'll need Cognitive Services User or Cognitive Services Speech User role to submit batch avatar synthesis jobs.
        credential = DefaultAzureCredential()
        token = credential.get_token('https://cognitiveservices.azure.com/.default')
        return {'Authorization': f'Bearer {token.token}'}
    else:
        SUBSCRIPTION_KEY = os.environ.get('SPEECH_KEY')
        return {'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY}
    
def submit_synthesis(job_id: str, displaytext: str):
    url = f'{SPEECH_ENDPOINT}/avatar/batchsyntheses/{job_id}?api-version={API_VERSION}'
    #header = {
    #    'Content-Type': 'application/json'
    #}
    #header.update(_authenticate())
    print("URL: ", url)

    header = {
        'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY,
        'Content-Type': 'application/json'
    }

    payload = {
        'synthesisConfig': {
            "voice": "en-US-JennyNeural",
        },
        # Replace with your custom voice name and deployment ID if you want to use custom voice.
        # Multiple voices are supported, the mixture of custom voices and platform voices is allowed.
        # Invalid voice name or deployment ID will be rejected.
        'customVoices': {
            # "YOUR_CUSTOM_VOICE_NAME": "YOUR_CUSTOM_VOICE_ID"
        },
        "inputKind": "PlainText",  # PlainText or SSML
        "inputs": [
            {
                "content": f"{displaytext}",
            },
        ],
        "avatarConfig": {
            "customized": False, # set to True if you want to use customized avatar
            "talkingAvatarCharacter": "lisa",  # talking avatar character
            "talkingAvatarStyle": "graceful-sitting",  # talking avatar style, required for prebuilt avatar, optional for custom avatar
            "videoFormat": "mp4",  # mp4 or webm, webm is required for transparent background
            "videoCodec": "h264",  # hevc, h264 or vp9, vp9 is required for transparent background; default is hevc
            "subtitleType": "soft_embedded",
            #"backgroundColor": "#FFFFFFFF", # background color in RGBA format, default is white; can be set to 'transparent' for transparent background
            #"backgroundColor": "transparent",
            "backgroundImage": "https://samples-files.com/samples/Images/jpg/1920-1080-sample.jpg", # background image URL, only support https, either backgroundImage or backgroundColor can be set
            #"backgroundImage": "https://github.com/balakreshnan/publicimageaccess/blob/main/aidesignlab3.png",
        }
    }

    response = requests.put(url, json.dumps(payload), headers=header)
    if response.status_code < 400:
        logger.info('Batch avatar synthesis job submitted successfully')
        logger.info(f'Job ID: {response.json()["id"]}')
        return True
    else:
        logger.error(f'Failed to submit batch avatar synthesis job: [{response.status_code}], {response.text}')


def get_synthesis(job_id):
    status = None
    url1 = ""
    url = f'{SPEECH_ENDPOINT}/avatar/batchsyntheses/{job_id}?api-version={API_VERSION}'
    #header = _authenticate()
    header = {
        'Ocp-Apim-Subscription-Key': SUBSCRIPTION_KEY
    }

    response = requests.get(url, headers=header)
    if response.status_code < 400:
        logger.debug('Get batch synthesis job successfully')
        logger.debug(response.json())
        if response.json()['status'] == 'Succeeded':
            status = 'Succeeded'
            url1 = response.json()["outputs"]["result"]
            logger.info(f'Batch synthesis job succeeded, download URL: {response.json()["outputs"]["result"]}')
        #return response.json()['status']
    else:
        logger.error(f'Failed to get batch synthesis job: {response.text}')
    return status, url1


def list_synthesis_jobs(skip: int = 0, max_page_size: int = 100):
    """List all batch synthesis jobs in the subscription"""
    url = f'{SPEECH_ENDPOINT}/avatar/batchsyntheses?api-version={API_VERSION}&skip={skip}&maxpagesize={max_page_size}'
    header = _authenticate()

    response = requests.get(url, headers=header)
    if response.status_code < 400:
        logger.info(f'List batch synthesis jobs successfully, got {len(response.json()["values"])} jobs')
        logger.info(response.json())
    else:
        logger.error(f'Failed to list batch synthesis jobs: {response.text}')
```

- Now process the video for speech services

```
def processtextovideo(displaytext):
    returntxt = ""
    status = None
    url1 = ""
    job_id = _create_job_id()
    if submit_synthesis(job_id, displaytext):
        while True:
            status, url1 = get_synthesis(job_id)
            if status == 'Succeeded':
                logger.info('batch avatar synthesis job succeeded')
                break
            elif status == 'Failed':
                logger.error('batch avatar synthesis job failed')
                break
            else:
                logger.info(f'batch avatar synthesis job is still running, status [{status}]')
                time.sleep(5)
    return status, url1
```

- Now to the main function to use for research platform

```
def csirecipedesignmain():
    returntxt = ""
    citationtxt = ""
    extreturntxt = ""
    summartycsitext = ""
    sliders = [25, 25, 25, 25]

    st.write("## Manufacturing Recipe Research Platform")

    if 'recipelist' not in st.session_state:
        st.session_state.recipelist = returntxt

    if 'summartycsitext' not in st.session_state:
        st.session_state.summartycsitext = summartycsitext

    if 'returntxt' not in st.session_state:
        st.session_state.returntxt = returntxt

    # Create tabs
    tab1, tab2 = st.tabs(["Chat", "Citations"])

    with tab1:

        tab11, tab12 = st.tabs(["Internal", "External"])

        with tab11:
            count = 0
            col1, col2, col3 = st.columns([1,1,1])
            with col1: 
                st.write("### Chat")
                query = st.text_area("Enter your query here:", height=20, value="Create a new receipe based on the selections")

                selected_optionmodel1 = st.selectbox("Select Model", ["gpt-4o-2", "gpt-4o-g", "gpt-4o"])

                #blue = st.slider("Select a Blue range of values", 0, 100, 25)
                #green = st.slider("Select a Green range of values", 0, 100, 25)
                #orange = st.slider("Select a orange range of values", 0, 100, 25)
                #red = st.slider("Select a Red range of values", 0, 100, 25)

                #sliders = normalize_sliders([blue, green, orange, red])

                # Streamlit sliders
                slider1 = st.slider('Blue', 0, 100, int(sliders[0]), key='slider1')
                sliders = update_sliders(sliders, 0, slider1)
                slider2 = st.slider('Green', 0, 100, int(sliders[1]), key='slider2')
                sliders = update_sliders(sliders, 1, slider2)
                slider3 = st.slider('Orange', 0, 100, int(sliders[2]), key='slider3')
                sliders = update_sliders(sliders, 2, slider3)
                slider4 = st.slider('Red', 0, 100, int(sliders[3]), key='slider4')
                sliders = update_sliders(sliders, 3, slider4)


                # Display the sliders and their total
                st.write(f'Blue: {sliders[0]}%')
                st.write(f'Green: {sliders[1]}%')
                st.write(f'Orange: {sliders[2]}%')
                st.write(f'Red: {sliders[3]}%')
                st.write(f'Total: {sum(sliders)}%')

                # Update slider values to normalized values
                #blue, green, orange, red = sliders

                if st.button("Submit"):
                    #returntxt = csirecipedesign(query, selected_optionmodel1, blue, green, orange, red)
                    returntxt = csirecipedesign(query, selected_optionmodel1, slider1, slider2, slider3, slider4)
                    st.session_state.returntxt = returntxt
                    summartycsitext = csirecipedesignsummary(query, selected_optionmodel1, slider1, slider2, slider3, slider4, st.session_state.returntxt)
                    print('CSI Summary Text: ' , summartycsitext)
                    st.session_state.summartycsitext = summartycsitext

                    #st.write(returntxt)
            with col2:
                if st.button("Play Audio"):
                    if st.session_state.returntxt:
                        process_text_to_speech(st.session_state.returntxt)
                        st.markdown(st.session_state.returntxt, unsafe_allow_html=True)                        
                    #process_text_to_speech(returntxt)
                if returntxt:                    
                    #st.write(returntxt)
                    st.markdown(returntxt, unsafe_allow_html=True)
                elif st.session_state.returntxt:
                    st.markdown(st.session_state.returntxt, unsafe_allow_html=True)
                
                #st.write(returntxt)
            with col3:
                if st.button("Create Video"):
                    if st.session_state.summartycsitext:
                        st.write('Summarize Text: ', st.session_state.summartycsitext)
                        status, url1 = processtextovideo(st.session_state.summartycsitext)
                        st.video(url1)
                        st.write(f"Status: {status}")
                        #st.write(f"Video URL: {url1}")
                        #print("Yes to process video: ", st.session_state.summartycsitext) 
                        
                    else:
                        st.write("No text to create video") 
            with tab12:
                #st.write("### External")
                if returntxt != "":
                    #st.write(returntxt)
                    st.markdown(returntxt, unsafe_allow_html=True)
        
    with tab2:
        st.write("### Citations")
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/csiresearch1.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/csiresearch2.jpg 'RagChat')