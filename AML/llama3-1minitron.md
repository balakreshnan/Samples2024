# Azure Machine learning - Llama-3.1-Minitron-4B-Width-Base

## Introduction

- Using Azure machine learning GPU show how easy to run a open source model
- using Llama-3.1-Minitron-4B-Width-Base model from Hugging Face
- Using 2 GPU SKU like Standard_NC96ads_A100_v4
- using python 3.12 environment
- Code from - https://huggingface.co/openai/whisper-large-v3-turbo
  
## Pre-requisites

- Azure Machine Learning workspace
- Azure Subscription
- Azure Machine Learning GPU SKU like Standard_NC96ads_A100_v4
- Python 3.12 environment

- Create a new python 3.12 environment

```
conda create --name py312
conda activate py312
conda install python==3.12
conda install ipykernel
python -m ipykernel install --user --name py312 --display-name "Python (py312)"
```

- Install the required packages

```
import torch
from transformers import AutoTokenizer, LlamaForCausalLM
```

- Set the model

```
# Load the tokenizer and model
model_path = "nvidia/Llama-3.1-Minitron-4B-Width-Base"
tokenizer = AutoTokenizer.from_pretrained(model_path)
```

- set the device

```
device = 'cuda'
dtype = torch.bfloat16
model = LlamaForCausalLM.from_pretrained(model_path, torch_dtype=dtype, device_map=device)
```

- Set the prompt

```
# Prepare the input text
prompt = 'Complete the paragraph: our solar system is'
inputs = tokenizer.encode(prompt, return_tensors='pt').to(model.device)
```

- Execute the model

```
# Generate the output
outputs = model.generate(inputs, max_length=20)
```

- Display the output

```
# Decode and print the output
output_text = tokenizer.decode(outputs[0])
print(output_text)
```

- Output

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/llama3-1minitron-1.jpg 'RagChat')

## Conclusion

- as you can see with few lines of code you can run a open source model in Azure Machine Learning
- there is not much difference between running in local and Azure Machine Learning
- It's also very much open source friendly.