# Azure Machine learning - Llama-3.2-1B

## Introduction

- Using Azure machine learning GPU show how easy to run a open source model
- using Llama-3.2-1B model from Hugging Face
- Using 2 GPU SKU like Standard_NC96ads_A100_v4
- using python 3.12 environment
- Code from - https://huggingface.co/openai/whisper-large-v3-turbo
  
## Pre-requisites

- Azure Machine Learning workspace
- Azure Subscription
- Azure Machine Learning GPU SKU like Standard_NC96ads_A100_v4
- Python 3.12 environment

- Create a new python 3.12 environment

### Code

- install libraries

```
pip install ipywidgets
pip install --upgrade transformers datasets[audio] accelerate
```

```
conda create --name py312
conda activate py312
conda install python==3.12
conda install ipykernel
python -m ipykernel install --user --name py312 --display-name "Python (py312)"
```

- load libraries

```
import torch
from transformers import pipeline
```

- define the model

```
model_id = "meta-llama/Llama-3.2-1B"
```

- login into huggingface

```
from huggingface_hub import notebook_login
notebook_login()
```

- Set the pipeline to load the model

```
pipe = pipeline(
    "text-generation", 
    model=model_id, 
    torch_dtype=torch.bfloat16, 
    device_map="auto"
)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/llama3-2-1B-1.jpg 'RagChat')

- Set the prompt
- also execute the model

```
result = pipe("The key to life is")
```

- here is the output

```
print(result[0]["generated_text"])
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/llama3-2-1B-2.jpg 'RagChat')

## Conclusion

- as you can see with few lines of code you can run a open source model in Azure Machine Learning
- there is not much difference between running in local and Azure Machine Learning
- It's also very much open source friendly.
- There is very less we need to install