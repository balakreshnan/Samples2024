# Azure Machine learning fine tuning LLama 7BHF model

## Use Case

- Fine tuning LLama 7BHF model
- Using compute instance
- Using GPU Compute SKU: Standard_NC48ads_A100
- Using Azure Machine Learning
- Idea here is to learn to fine tune a open source Meta's LLama 7BHF model using Azure Machine Learning
- This model is a large language model with 7B parameters
- Using a existing sample found in web with their data set
- Here is the sample i am replicating - https://www.mlexpert.io/machine-learning/tutorials/alpaca-fine-tuning

## Code

- First install all the necessary libraries
- Mostly install using pip install in each cell
  
- try this and see if not use individual pip install

```python
!pip install -U pip
!pip install accelerate==0.18.0
!pip install appdirs==1.4.4
!pip install bitsandbytes==0.37.2
!pip install datasets==2.10.1
!pip install fire==0.5.0
!pip install git+https://github.com/huggingface/peft.git
!pip install git+https://github.com/huggingface/transformers.git
!pip install torch==2.0.0
!pip install sentencepiece==0.1.97
!pip install tensorboardX==2.6
!pip install gradio==3.23.0
```

```
!pip install transformers
```

```
pip install transformers
```

```
pip install peft
```

```
pip install fire
```

```
pip install datasets
```

```
pip install matplotlib
```

```
pip install seaborn
```

```
pip install torch
```

```
pip install accelerate
```

```
pip install tensorboard
```

```
!pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

```
pip install accelerate
```

```
pip install bitsandbytes
```

```
pip install sentencepiece
```

```
pip install tensorboardX
```

```
pip install gradio
```

- IMport and configure

```
import transformers
import textwrap
from transformers import LlamaTokenizer, LlamaForCausalLM
import os
import sys
from typing import List

from peft import (
    LoraConfig,
    get_peft_model,
    get_peft_model_state_dict,
    prepare_model_for_int8_training,
)

import fire
import torch
from datasets import load_dataset
import pandas as pd

import matplotlib.pyplot as plt
import matplotlib as mpl
import seaborn as sns
from pylab import rcParams
import json

%matplotlib inline
sns.set(rc={'figure.figsize':(8, 6)})
sns.set(rc={'figure.dpi':100})
sns.set(style='white', palette='muted', font_scale=1.2)

DEVICE = "cuda" if torch.cuda.is_available() else "cpu"
DEVICE
```

- Login into Hugging face to download model

```
from huggingface_hub import notebook_login
#you need the key from hugging face to authenticate
notebook_login()
```

- Load LLama 7BHF model from Meta

``` 
BASE_MODEL = "meta-llama/Llama-2-7b-hf"

model = LlamaForCausalLM.from_pretrained(
    BASE_MODEL,
    load_in_8bit=True,
    torch_dtype=torch.float16,
    device_map="auto",
)

tokenizer = LlamaTokenizer.from_pretrained(BASE_MODEL)

tokenizer.pad_token_id = (
    0  # unk. we want this to be different from the eos token
)
tokenizer.padding_side = "left"
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama2alphaca-4.jpg 'RagChat')

- load the dataset

```
data = load_dataset("json", data_files="alpaca-bitcoin-sentiment-dataset.json")
data["train"]
```

- Configure parameters

```
CUTOFF_LEN = 256
```

- Define functions

```
def generate_prompt(data_point):
    return f"""Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.  # noqa: E501
### Instruction:
{data_point["instruction"]}
### Input:
{data_point["input"]}
### Response:
{data_point["output"]}"""
```

- functions

```
def tokenize(prompt, add_eos_token=True):
    # there's probably a way to do this with the tokenizer settings
    # but again, gotta move fast
    result = tokenizer(
        prompt,
        truncation=True,
        max_length=CUTOFF_LEN,
        padding=False,
        return_tensors=None,
    )
    if (
        result["input_ids"][-1] != tokenizer.eos_token_id
        and len(result["input_ids"]) < CUTOFF_LEN
        and add_eos_token
    ):
        result["input_ids"].append(tokenizer.eos_token_id)
        result["attention_mask"].append(1)

    result["labels"] = result["input_ids"].copy()

    return result

def generate_and_tokenize_prompt(data_point):
    full_prompt = generate_prompt(data_point)
    tokenized_full_prompt = tokenize(full_prompt)
    return tokenized_full_prompt
```

- train and validation split

```
train_val = data["train"].train_test_split(
    test_size=200, shuffle=True, seed=42
)
train_data = (
    train_val["train"].shuffle().map(generate_and_tokenize_prompt)
)
val_data = (
    train_val["test"].shuffle().map(generate_and_tokenize_prompt)
)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama2alphaca-5.jpg 'RagChat')

- Set parameters for Lora and experiments

```
LORA_R = 8
LORA_ALPHA = 16
LORA_DROPOUT= 0.05
LORA_TARGET_MODULES = [
    "q_proj",
    "v_proj",
]

BATCH_SIZE = 128
MICRO_BATCH_SIZE = 4
GRADIENT_ACCUMULATION_STEPS = BATCH_SIZE // MICRO_BATCH_SIZE
LEARNING_RATE = 3e-4
TRAIN_STEPS = 20
OUTPUT_DIR = "experiments"
```

- prepare model

```
model = prepare_model_for_int8_training(model)
config = LoraConfig(
    r=LORA_R,
    lora_alpha=LORA_ALPHA,
    target_modules=LORA_TARGET_MODULES,
    lora_dropout=LORA_DROPOUT,
    bias="none",
    task_type="CAUSAL_LM",
)
model = get_peft_model(model, config)
model.print_trainable_parameters()
```

- Setup training parameters for transformer

```
training_arguments = transformers.TrainingArguments(
    per_device_train_batch_size=MICRO_BATCH_SIZE,
    gradient_accumulation_steps=GRADIENT_ACCUMULATION_STEPS,
    warmup_steps=2,
    max_steps=TRAIN_STEPS,
    learning_rate=LEARNING_RATE,
    fp16=True,
    logging_steps=5,
    optim="adamw_torch",
    evaluation_strategy="steps",
    save_strategy="steps",
    eval_steps=5,
    save_steps=5,
    output_dir=OUTPUT_DIR,
    save_total_limit=3,
    load_best_model_at_end=True,
    report_to="tensorboard"
)
```

```
data_collator = transformers.DataCollatorForSeq2Seq(
    tokenizer, pad_to_multiple_of=8, return_tensors="pt", padding=True
)
```

- Train the model

```
import torch._dynamo
torch._dynamo.config.suppress_errors = True

trainer = transformers.Trainer(
    model=model,
    train_dataset=train_data,
    eval_dataset=val_data,
    args=training_arguments,
    data_collator=data_collator
)
model.config.use_cache = False
old_state_dict = model.state_dict
model.state_dict = (
    lambda self, *_, **__: get_peft_model_state_dict(
        self, old_state_dict()
    )
).__get__(model, type(model))

model = torch.compile(model)

trainer.train()
model.save_pretrained(OUTPUT_DIR)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama2alphaca-8.jpg 'RagChat')

- Load runs

```
%load_ext tensorboard
%tensorboard --logdir experiments/runs
```