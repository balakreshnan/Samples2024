# Speech Translation from one language to another - Voice to Voice

## Introduction

- Translate audio from one language to another
- using wav audio
- Testing in batch mode
- can be used for real time translation as well

## Pre-requisites

- Azure Subscription
- Azure Speech SDK
- Azure AI Speech services
- Check the region and Key

## Steps

- I am using Visual Studio Code local to develop
- Using python environment
- Install libraries

```
pip install azure-cognitiveservices-speech
pip install streamlit
pip install openai
pip install python-dotenv
```

- Import libraries

```
import streamlit as st
import azure.cognitiveservices.speech as speechsdk
from dotenv import dotenv_values
import io

config = dotenv_values("env.env")
```

- Create the function to get the audio file and translate to corresponding target language
- Lanugage to translate to is passed as parameter
- Lanugage to create target speech is passed as parameter

```
def translateaudio(option1, option2, audio_bytes, option3):
    
    rttext = ""
    #audio_io = io.BytesIO(audio_bytes)
    #audio_bytes.save("temp1.wav")
    with open("temp1.wav", "wb") as f:
        f.write(audio_bytes)
    audio_filename = "temp1.wav"

    # This example requires environment variables named "SPEECH_KEY" and "SPEECH_REGION"
    
    speechregion = config["SPEECH_REGION"]
    speechkey = config["SPEECH_KEY"]
     speech_translation_config = speechsdk.translation.SpeechTranslationConfig(subscription=speechkey, region=speechregion)
    speech_translation_config.speech_recognition_language="en-US"
    

    target_language = option1
    speech_translation_config.add_target_language(target_language)
    #print('Audio bytes:', audio_bytes)

    #audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)
    #audio_config = speechsdk.audio.AudioConfig(stream=audio_bytes)
    #audio_config = speechsdk.audio.AudioConfig(stream=audio_io)
    #audio_config = speechsdk.audio.AudioConfig(filename="Call3_separated_16k_pharmacy_call.wav")
    #audio_config = speechsdk.audio.AudioConfig(stream=audio_input)
    audio_config = speechsdk.audio.AudioConfig(filename="temp1.wav")
    translation_recognizer = speechsdk.translation.TranslationRecognizer(translation_config=speech_translation_config, audio_config=audio_config)

    #print("Speak into your microphone.")
    print("Processing .....")
    translation_recognition_result = translation_recognizer.recognize_once_async().get()

    if translation_recognition_result.reason == speechsdk.ResultReason.TranslatedSpeech:
        print("Recognized: {} \n".format(translation_recognition_result.text))
        print("""Translated into '{}': {} \n""".format(
            target_language, 
            translation_recognition_result.translations[target_language]))
        #rttext = translation_recognition_result.text
        rttext = translation_recognition_result.translations[target_language]
        audio_config = speechsdk.audio.AudioOutputConfig(use_default_speaker=True)

        # The neural multilingual voice can speak different languages based on the input text.
        speech_config = speechsdk.SpeechConfig(subscription=speechkey, region=speechregion)
        #speech_config.speech_synthesis_voice_name='en-US-AvaMultilingualNeural'
        speech_config.speech_synthesis_voice_name=option3
        #speech_config.speech_recognition_language='ta-IN'
        #speech_config.speech_synthesis_voice_name='ta-IN-PallaviNeural'
        #speech_config.speech_synthesis_language='ta-IN'

        speech_config.speech_recognition_language=option2
        speech_config.speech_synthesis_language=option2
        

        #speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=audio_config)
        #speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)
        pull_stream = speechsdk.audio.PullAudioOutputStream()
        
        # Creates a speech synthesizer using pull stream as audio output.
        stream_config = speechsdk.audio.AudioOutputConfig(stream=pull_stream)
        speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=stream_config)
        
        #speech_synthesis_result = speech_synthesizer.speak_text_async(rttext).get()
        speech_synthesis_result = speech_synthesizer.speak_text(rttext)

        #rsstream = speechsdk.AudioDataStream(speech_synthesis_result)
        rsstream = speech_synthesis_result.audio_data
        print("Audio duration: {} seconds \n".format(speech_synthesis_result.audio_duration.total_seconds()))
        #rsstream = speechsdk.AudioDataStream(speech_synthesis_result)
       

        if speech_synthesis_result.reason == speechsdk.ResultReason.SynthesizingAudioCompleted:
            print("Speech synthesized for text [{}] \n".format(rttext))
        elif speech_synthesis_result.reason == speechsdk.ResultReason.Canceled:
            cancellation_details = speech_synthesis_result.cancellation_details
            print("Speech synthesis canceled: {}".format(cancellation_details.reason))
            if cancellation_details.reason == speechsdk.CancellationReason.Error:
                if cancellation_details.error_details:
                    print("Error details: {}".format(cancellation_details.error_details))
                    print("Did you set the speech resource key and region values?")
    elif translation_recognition_result.reason == speechsdk.ResultReason.NoMatch:
        print("No speech could be recognized: {}".format(translation_recognition_result.no_match_details))
        rttext = translation_recognition_result.no_match_details
    elif translation_recognition_result.reason == speechsdk.ResultReason.Canceled:
        cancellation_details = translation_recognition_result.cancellation_details
        print("Speech Recognition canceled: {}".format(cancellation_details.reason))
        rttext = cancellation_details.reason
        if cancellation_details.reason == speechsdk.CancellationReason.Error:
            print("Error details: {}".format(cancellation_details.error_details))
            print("Did you set the speech resource key and region values?")
            rttext = cancellation_details.error_details
    return rttext, rsstream
```

- Now create the main function
- Create input for audio translation to text first
- Then convert the text into audio in the target language

```
def main():
    count = 0
    col1, col2 = st.columns(2)
    status = None
    url1 = ""
    video_file = open(VIDEO_URL, 'rb')
    video_bytes = video_file.read()

    #st.video(video_bytes)
    with col1:
        option1 = st.selectbox('Input language Conversion:',
                      ('ta', 'en', 'es'))
        
        option2 = st.selectbox('Output Voice language:',
                      ('ta-IN', 'en-US', 'es-ES', 'en-IN'))
        #st.video(video_bytes)

        options3 = st.selectbox('Output Voice language:',
                      ('ta-IN-PallaviNeural', 'en-US-AvaMultilingualNeural', 'en-US-EmmaNeural', 'en-US-BrandonNeural'))

        audio_file = open("Call3_separated_16k_pharmacy_call.wav", "rb")  
        audio_bytes = audio_file.read()     

        st.audio(audio_bytes)

        # Upload audio file
        uploaded_file = st.file_uploader("Choose an audio file", type=["wav", "mp3"])
        
        if uploaded_file is not None:
            audio_bytes = uploaded_file.read()
            status = "Upload done"
            st.audio(audio_bytes)
            displaytext, rsstream = translateaudio(option1, option2, audio_bytes, options3)
            st.markdown(displaytext, unsafe_allow_html=True)
            st.audio(rsstream)

        if st.button('Translate Sentence'):            
            displaytext, rsstream = translateaudio(option1, option2, audio_bytes, options3)
            count += 1
            status = "Translation done"

            st.markdown(displaytext, unsafe_allow_html=True)
            st.audio(rsstream)
```

- now invoke the main function

```
if __name__ == "__main__":
    main()
```

- Screen will out put the text
- Then there will be a audio play button to listen to the audio in the target language