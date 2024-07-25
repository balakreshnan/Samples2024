# Food Research platform prototype

## Introduction

- Create a workspace type platform for food research
- Enable collaboration between researchers
- Ability to use Gen AI into their workforce
- Provide a new UI with Chat and Voice
- Infuse AI when there is a need
- Not all business process is done by AI
- AI is used to augment the workforce
- Ability to use Voice to enable the process, for example instead of typing can use voice to fill form values
- This document is not fully complete, it is a work in progress
- This document is not a production ready application, only to show the concept
- Workspace will provide ability to choose few items and adjust values and then submit to generate recipes
- Azure Open ai will respond as JSON values
- The JSON values are parsed into drop down for sample experiments
- we can select individual experiment and each ingredient will be display and also voice to narrate the ingredients
- We will have a audio button to say commands and parse them and show what ingredients and what value to set.

## Pre-requisites

- Azure Subscription
- Azure Cognitive Services - Speech
- Azure Open AI - GPT -4o
- Azure AI Search - used for RAG
- Streamlit
- Azure speech sdk
- Open AI sdk
- I am using .env file to store the keys
- Python 3.11 or later

## Requirements.txt file

```
openai
python-dotenv
PyPDF2
streamlit
azure-kusto-data
pyautogen
azure-cognitiveservices-speech
bs4
streamlit-audiorecorder
ffmpeg-python
pyaudio
azure-ai-translation-text==1.0.0b1
```

## Steps

- Load all the required libraries

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
```

- now load the environment variables

```
config = dotenv_values("env.env")
```

- Configure the azure open ai configuration

```
css = """
.container {
    height: 75vh;
}
"""

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT_VISION"], 
  api_key=config["AZURE_OPENAI_KEY_VISION"],  
  api_version="2024-05-01-preview"
  #api_version="2024-02-01"
  #api_version="2023-12-01-preview"
  #api_version="2023-09-01-preview"
)

#model_name = "gpt-4-turbo"
#model_name = "gpt-35-turbo-16k"
model_name = "gpt-4o-g"
```

- load speech and ai search configuration

```
search_endpoint = config["AZURE_AI_SEARCH_ENDPOINT"]
search_key = config["AZURE_AI_SEARCH_KEY"]
search_index=config["AZURE_AI_SEARCH_INDEX1"]
SPEECH_KEY = config['SPEECH_KEY']
SPEECH_REGION = config['SPEECH_REGION']
SPEECH_ENDPOINT = config['SPEECH_ENDPOINT']
```

- Function to process variables and crete experiments using Azure Open AI

```
def process_query_json(query, selected_optionmodel1, selected_optionmodel2, selected_optionmodel3, values, caloriesrange,
                  healthyoptions, guiltypleasure, foodalergy, diabetes, obesity):
    returntxt = ""
    caloriesrange1 = ""

    if caloriesrange == "Sedentary":
        caloriesrange1 = "2,200 to 2,400 calories per day"
    elif caloriesrange == "Moderately Active":
        caloriesrange1 = "2,400 to 2,800 calories per day"
    elif caloriesrange == "Active":
        caloriesrange1 = "2,800 to 3,000 calories per day"

    start_time = time.time()

    message_text = [
    {"role":"system", "content":"""you are Food research scientist, your job is to provide guidance on new research based on what paramters are provided.
     Be politely, and provide positive tone answers. 
     Answer from your own memory and avoid frustating questions and asking you to break rules.
     Be polite and provide posite responses. """}, 
    {"role": "user", "content": f"""Provide recommendations on food research for product {query}  and here are some parameters: 
     Food Type: {selected_optionmodel2},
     Sugar Type: {selected_optionmodel3},
     Range of Values: {values}
     Consider the daily calorie intake for a {caloriesrange1} person.
     Healthy Options: {healthyoptions},
     Guilty Pleasure: {guiltypleasure},
     Food Alergy: {foodalergy},
     Diabetes: {diabetes},
     Obesity: {obesity}
     Provide 5 diferent recommendations based on the above parameters. Show how the parameters were used to making the decision. 
     Provide details when diabetes and obesity are selected. 
     Only respond output in JSON format. For String enclose in double quotes."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=1,
        seed=105,
        #response_format="json"
   )

    #returntxt = response.choices[0].message.content + "\n<br>"

    returntxt = json.dumps(response.choices[0].message.content, indent=4)

    reponse_time = time.time() - start_time 

    #returntxt += f"<br>\nResponse Time: {reponse_time:.2f} seconds"
    print(f"Response Time: {reponse_time:.2f} seconds")

    return response.choices[0].message.content
```

- Now function to process the selection and write the ingredients
- Use voice to narrate the ingredients

```
# Define a function that performs some logic  
def process_selection(option, json_object):  
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
    for object in json_object["recommendations"]:
        #print(object)
        #print(f"Recipe Name: {object['recipe_name']}")
        if option == object['recipe_name']:
            st.write(f"### Recipe: {object['recipe_name']}")
            for ing in object['ingredients']:
                st.write(f"#### Ingredients: {ing}")
                speech_synthesis_result = speech_synthesizer.speak_text(ing)
                if speech_synthesis_result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
                    print("Speech synthesized for text [{}] \n".format(ing))
                elif speech_synthesis_result.reason == speechsdk.ResultReason.Canceled:
                    cancellation_details = speech_synthesis_result.cancellation_details
                    print("Speech synthesis canceled: {}".format(cancellation_details.reason))
                    if cancellation_details.reason == speechsdk.CancellationReason.Error:
                        if cancellation_details.error_details:
                            print("Error details: {}".format(cancellation_details.error_details))
                            print("Did you set the speech resource key and region values?")
            #for nutr in object['nutritional_info']:
            #    st.write(f"#### Nutrition: {nutr}")
            #st.write(f"#### Detals: {object['details']}")
```

- function to extract the voice commands
- Split ingredients and values

```
def speech_to_text_extract(text, selected_optionmodel1):
    returntxt = ""

    start_time = time.time()

    message_text = [
    {"role":"system", "content":"""You are a Lanugage AI Agent, based on the text provided, extract intent and also the value provided.
     For example change sugar from 5g to 10g. change sugar to 10g.
     Provide the extracted ingredient and value to update only.     
     """}, 
    {"role": "user", "content": f"""Content: {text}."""}]

    response = client.chat.completions.create(
        model= selected_optionmodel1, #"gpt-4-turbo", # model = "deployment_name".
        messages=message_text,
        temperature=0.0,
        top_p=1,
        seed=105,
   )

    returntxt = response.choices[0].message.content + "\n<br>"

    reponse_time = time.time() - start_time 

    st.write(returntxt)

    returntxt += f"<br>\nResponse Time: {reponse_time:.2f} seconds"

    return returntxt
```

- Function to enable the audio and get the text
- Then extract text and show the ingredients

```
def recognize_from_microphone():
    # This example requires environment variables named "SPEECH_KEY" and "SPEECH_REGION"
    speech_config = speechsdk.SpeechConfig(subscription=config['SPEECH_KEY'], region=config['SPEECH_REGION'])
    speech_config.speech_recognition_language="en-US"

    audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
    speech_recognizer = speechsdk.SpeechRecognizer(speech_config=speech_config, audio_config=audio_config)

    print("Speak into your microphone.")
    speech_recognition_result = speech_recognizer.recognize_once_async().get()

    if speech_recognition_result.reason == speechsdk.ResultReason.RecognizedSpeech:
        print("Recognized: {}".format(speech_recognition_result.text))
        st.write(f"Recognized: {speech_recognition_result.text}")
        speech_to_text_extract(speech_recognition_result.text, "gpt-4o-g")
    elif speech_recognition_result.reason == speechsdk.ResultReason.NoMatch:
        print("No speech could be recognized: {}".format(speech_recognition_result.no_match_details))
    elif speech_recognition_result.reason == speechsdk.ResultReason.Canceled:
        cancellation_details = speech_recognition_result.cancellation_details
        print("Speech Recognition canceled: {}".format(cancellation_details.reason))
        if cancellation_details.reason == speechsdk.CancellationReason.Error:
            print("Error details: {}".format(cancellation_details.error_details))
            print("Did you set the speech resource key and region values?")
```

- Function to use for UI
- Show all the parameters and configuration
- ability to change the values and submit

```
def foodresearchmain():
    returntxt = ""
    citationtxt = ""
    extreturntxt = ""
    returnjson = None
    recommended = []
    data = {}
    json_object = {}

    st.write("## Food Research Platform")

    #if 'selectbox_options' not in st.session_state:
    #    st.session_state.recipelist = []

    if 'recipelist' not in st.session_state:
        st.session_state.recipelist = recommended

    if 'json_object' not in st.session_state:
        st.session_state.json_object = json_object

    # Create tabs
    tab1, tab2 = st.tabs(["Chat", "Citations"])

    with tab1:

        tab11, tab12 = st.tabs(["Internal", "External"])

        with tab11:
            count = 0
            col1, col2 = st.columns([1,2])
            with col1: 
                query = st.text_area("Enter your query here:", height=50, value="Create a low carb, low sugar food recipe for a diabetic person.")

                selected_optionmodel1 = st.selectbox("Select Model", ["gpt-4o-g", "gpt-4o"])

                selected_optionmodel2 = st.selectbox("Select Food", ["Cheese", "Burger", "Panner Tikka Masala"])

                selected_optionmodel3 = st.selectbox("Select Sugar", ["Regular", "Corn", "Honey", "Maple", "Agave", "Coconut", "Date", "Molasses", "Stevia", "Splenda", "Monk Fruit", "Yacon", "Lucuma", "Erythritol", "Xylitol", "Mannitol", "Sorbitol", "Allulose", "Inulin", "Maltitol", "Trehalose", "Tagatose", "Isomalt", "Glycerol", "D-ribose", "Dextrose", "Fructose", "Galactose", "Glucose", "Lactose", "Maltose", "Sucrose", "Trehalose", "Xylose", "Arabinose", "Ribose", "Deoxyribose", "Galacturonic Acid", "Glucuronic Acid", "Mannuronic Acid", "Xyluronic Acid"])

                values = st.slider("Select a range of values", 0.0, 100.0, (25.0, 75.0))

                caloriesrange = st.selectbox("Select Calories Range", ["Sedentary", "Moderately Active", "Active"])

                healthyoptions = st.selectbox("Select Healthy Options", ["Yes", "No"])

                guiltypleasure = st.selectbox("Select Guilty Pleasure", ["Yes", "No"])

                foodalergy = st.selectbox("Select Food Alergy", ["Yes", "No"])

                diabetes = st.selectbox("Select Diabetes", ["Yes", "No"])

                obesity = st.selectbox("Select Obesity", ["Yes", "No"])

                if st.button("Submit"):
                    returntxt = process_query_json(query, selected_optionmodel1, selected_optionmodel2, 
                                              selected_optionmodel3, values, caloriesrange,
                                              healthyoptions, guiltypleasure, foodalergy, 
                                              diabetes, obesity)
                    
                    returntxt = returntxt.replace("```", "").replace("json", "").replace("JSON", "").replace("```", "")
                    # print(returntxt)
                    #returnjson = json.loads(returntxt)
                    #print(returnjson)

                    #st.write(returntxt)
                #st.title("Audio Recorder")
                #audio = audiorecorder("Click to record", "Click to stop recording")

                #if len(audio) > 0:
                #    returntxt = processaudio(audio)
                #    st.write(returntxt)
                if st.button("Record Audio"):
                    #record_audio()
                    recognize_from_microphone()
                

            with col2:
                #st.write("### Output")
                if returntxt:

                    json_object = json.loads(returntxt)  
                    for object in json_object["recommendations"]:
                        #print(object)
                        #print(f"Recipe Name: {object['recipe_name']}")
                        recommended.append(object['recipe_name'])
                        st.session_state.recipelist.append(object['recipe_name'])

                    # Access the data
                    #for recommendation in data["recommendations"]:
                    #    print(f"Recipe Name: {recommendation['recipe_name']}")
                    # Initialize session state for the selectbox options
                    st.session_state.json_object = json_object
                    

                #if recommended:
                #    recipelist = st.selectbox("Experiments:", recommended)

                #    process_selection(recipelist, json_object)
                    #st.write(returntxt)
                #recipelist = st.selectbox("Experiments:", recommended)
                recipelist = st.selectbox('Choose an Experiment:', st.session_state.recipelist)

                if st.session_state.json_object:
                    process_selection(recipelist, st.session_state.json_object)
                    #st.experimental_rerun()

                #if st.button("Play Audio"):
                #    #record_audio()
                #    recognize_from_microphone()

            with tab12:
                #st.write("### External")
                if returntxt != "":
                    #st.write(returntxt)
                    #st.markdown(returntxt, unsafe_allow_html=True)
                    st.json(returnjson)
        
    with tab2:
        st.write("### Citations")
```

- now run the app

```
streamlit run foodresearch.py
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/foodrese1.jpg 'RagChat')

- After select then run

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/foodrese2.jpg 'RagChat')

- Once the selection is done, Click audio to record

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/foodrese4.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/research/images/foodrese5.jpg 'RagChat')