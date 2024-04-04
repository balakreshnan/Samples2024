# Take a picture and apply compliance checks

## Use Case

- Take a picture of screen and look for compliance checks
- Are they using right color
- is the correct font used
- Only for concepts and ideation for now.
- for Video production

## Pre-requisites

- Azure subscription
- Azure Open AI GPT 4 vision
- Azure machine learning compute intance

## Code - Steps

- first load environment

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure open ai

```
import os
import openai
import pandas as pd

openai.api_type = "azure"
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = "2024-02-01"
openai.api_key = config["AZURE_OPENAI_KEY"]
```

- Setup open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  #api_version="2024-02-01"
  api_version="2024-02-15-preview"
)
```

- Encoding function for image

```
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')
```

- image path

```
image_path = "images/accpres4.jpg"
```

- Load the base 64 image

```
image_base64 = encode_image(image_path)
```

- Setup the prompt

```
imgprompt = """You are Marketing AI agent, your job is to check for compliance based on the 
details provided below:
1. Brand guideline recognition: identify Accenture’s specific brand guidelines. 
Develop a recognition system to scan each viceo frame for compliance with these 
guidelines. 
o Color -brand (solids, tints, percentages, gradients, usage, crops)
o Typography and fonts (Graphik, GT Sectra, kerning, tracking, leading, usage, 
position, hierarchy)
o Tone (comms style, terminology, grammar, values, messaging)
o Logo (size, crop, position, usage, style, co-brand)
o Icons (usage, color, position, weight)
o Motion (lower thirds, transitions, intro titles, end cards, principles)
o Photography (usage, alignment, DE&I)
o Illustration (usage, style, context)
o Audio (SFX, background music, VO, brand mnemonics)
o Stock – DE&I, following brand photo guides, quality of acting and 
performance, watermarks, usage rights 
2. Content quality monitoring: Implement video and audio analysis algortihms 
capable of detecting issues like poor resolution, improper lighting, audio glitches, 
black frames. Provie detailed report highlighting parts of the video that may require 
attention. 
o Video resolution 
o Color – skin tones, uniformity between clips – grading/LUTS, 
shadows/midtones highlights, saturation, brightness
o Audio quality (spatial, 3D, Dolby Atmos)
o Accessibility
o Platform specs & dimensions (YouTube, TikTok, Insta, etc)
o Platform contextual best practices (YouTube, TikTok, Insta, etc)
o Legal & Ethical (copyright, privacy, Ai()

Does the image have all the compliance? What is missing?
"""
```

- Process the image

```
response = client.chat.completions.create(
  model="gpt-4-vision",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": f"{imgprompt}"},
        {
          "type": "image_url",
          "image_url": {
            "url" : f"data:image/jpeg;base64,{base64_image}",
          },
        },
      ],
    }
  ],
  max_tokens=300,
)

print(response.choices[0].message.content)
```

- Response

```
Based on the provided image, we cannot fully assess all the compliance details regarding Accenture's specific brand guidelines as we do not have sight of elements such as motion, audio, or the full range of color usage. However, there are a few observations we can make:

1. **Brand Guideline Recognition**:
   - **Color**: The colors used in the "Accenture" logo and the graphical elements seem to be on-brand, using what appears to be the identifiable Accenture blue and green. However, without the specific RGB or CMYK values, we cannot confirm if they are the exact shades as per the brand guidelines.
   - **Typography and Fonts**: The fonts used in the texts "Accenture CEO on Quarterly Results" and "Q2 EPS Beats, Cuts Annual Revenue Forecast" appear to be clean and professional, possibly aligning with the brand's guidelines, but without further information on the exact typefaces (Graphik, GT Sectra), it is hard to confirm complete compliance.
   - **Logo**: The Accenture logo is visible and appears to be correctly proportioned without any crop issues. It is in a prominent and recognizably acceptable position within the frame.

2. **Content Quality Monitoring**:
   - **Video Resolution**: The image provided doesn't allow us to determine the video resolution. However, clarity and sharpness are necessary to assess meeting HD or other required resolutions.
   - **Color Grading**: The image seems well-balanced in terms
```

- more to come soon