# Azure Machine Learning - GPT 4 Vision

## Requirements

- Azure Machine Learning workspace
- Azure Storage account
- Azure Machine learning compute instance
- Install open ai v1.0 and above
- gpt 4 vision was only available in west us
- i deployed the mode as gpt-4-vision (deployment name)

## Code

- Initiate open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://aoairesource.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxxxxxxxxxx",  
  api_version="2023-09-01-preview"
)
```

- call open wi gpt 4 vision with image and ask question

```
response = client.chat.completions.create(
  model="gpt-4-vision",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "what's in the image?"},
        {
          "type": "image_url",
          "image_url": {
            "url" : "https://i.pinimg.com/736x/29/27/92/292792eec3363196450f71e4b5af381d.jpg",
          },
        },
      ],
    }
  ],
  max_tokens=300,
)

print(response.choices[0])
```

- output

```
Choice(finish_reason=None, index=0, message=ChatCompletionMessage(content='This is a data visualization of a persons lifetime from 1984 to 2011. It displays various aspects such as time spent in different locations, activities, education, and mood/happiness. The visualization is designed in a circular format with color coding for different categories and a bar graph around the edge indicating weekly hours spent on each activity.', role='assistant', function_call=None, tool_calls=None), finish_details={'type': 'stop', 'stop': '<|fim_suffix|>'}, content_filter_results={'hate': {'filtered': False, 'severity': 'safe'}, 'self_harm': {'filtered': False, 'severity': 'safe'}, 'sexual': {'filtered': False, 'severity': 'safe'}, 'violence': {'filtered': False, 'severity': 'safe'}})
```