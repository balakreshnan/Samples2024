# Open AI Whisper 3 Large Model in Azure Machine learning from Hugging Face

## Introduction

- Using Azure machine learning GPU show how easy to run a open source model
- using whisper3large model from Hugging Face
- Using 2 GPU SKU like Standard_NC96ads_A100_v4
- using python 3.12 environment
- Code from - https://huggingface.co/openai/whisper-large-v3-turbo

## Pre-requisites

- Azure Machine Learning workspace
- Azure Subscription

## Code

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
%pip install --upgrade transformers datasets[audio] accelerate
```

- Load libraries

```
import torch
from transformers import AutoModelForSpeechSeq2Seq, AutoProcessor, pipeline
from datasets import load_dataset
```

- Set device and model name

```
device = "cuda:0" if torch.cuda.is_available() else "cpu"
torch_dtype = torch.float16 if torch.cuda.is_available() else torch.float32

model_id = "openai/whisper-large-v3-turbo"
```

- Set the model

```
model = AutoModelForSpeechSeq2Seq.from_pretrained(
    model_id, torch_dtype=torch_dtype, low_cpu_mem_usage=True, use_safetensors=True
)
model.to(device)
```

- Set the processor

```
processor = AutoProcessor.from_pretrained(model_id)

pipe = pipeline(
    "automatic-speech-recognition",
    model=model,
    tokenizer=processor.tokenizer,
    feature_extractor=processor.feature_extractor,
    torch_dtype=torch_dtype,
    device=device,
)
```

- Load the dataset

```
dataset = load_dataset("distil-whisper/librispeech_long", "clean", split="validation")
sample = dataset[0]["audio"]
```

- Run the model

```
result = pipe(sample, return_timestamps=True)
```

- print the result

```
print(result["text"])
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/whisper3-large-1.jpg 'RagChat')

## Conclusion

- as you can see with few lines of code you can run a open source model in Azure Machine Learning
- there is not much difference between running in local and Azure Machine Learning
- It's also very much open source friendly.