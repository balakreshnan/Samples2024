# Fine tuning LLaMa 2 with Azure Machine learning and upload to hugging face - Cricket Dataset

## Introduction

- Let's use Azure machine learning to fine tune the phi 2 model and upload to hugging face.
- Using open source dataset for public documentation
- Using LLaMa 2 model
- Then uploading to a my new repo
- Creating my own Cricket dataset

## Requirements

- Azure Account
- Azure Machine Learning Service
- Compute instance with GPU compute
- using Standard_NC48ads_A100_v4 (48 cores, 440 GB RAM, 128 GB disk)
- Model size is 6GB
- So we need enough memory to load the model and train
- Need huggingace token with write access.
- We can use read access key to download the model from hugging face
- TO upload to huggingface we need write access token

## Steps

- First lets create the compute instance
- Use GPU SKU for model finetuning
- log into terminal and create conda environment

```
conda create -n finetune python=3.10 anaconda

ipython kernel install --user --name finetune --display-name "finetune"
```

- Now create a new notebook and select the kernel finetune
- Install necessary libraries

```
%pip install wandb
%pip install transformers
%pip install trl
%pip install datasets
%pip install protobuf
%pip install evaluate
%pip install peft
%pip install bitsandbytes
%pip install accelerate
%pip install sentencepiece
```

- Now code to fine tune

```
import os
import torch
from datasets import load_dataset
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    BitsAndBytesConfig,
    TrainingArguments,
    pipeline,
    logging,
)
from peft import LoraConfig
from trl import SFTTrainer
```

- Setup the model information like name for input and output model

```
# Model from Hugging Face hub
#base_model = "meta-llama/Llama-2-7b-chat-hf"
base_model = "microsoft/phi-2"

# New instruction dataset
# guanaco_dataset = "mlabonne/guanaco-llama2-1k"

# Fine-tuned model
new_model = "Balab2021/phi2"
```

- Convert the jsonl to LLaMa 2 format

```
import json
import pandas as pd

# Read JSONL file and extract information
data = []
with open('ipl20rowsoutput.jsonl', 'r') as file:
    for line in file:
        item = json.loads(line)
        question = item['question']
        answer = item['answer']
        # Format data with <inst> tag
        data.append({'text': f"<INST>{question}</INST> {answer}"})

# Convert to DataFrame
df = pd.DataFrame(data)

# Save LLAMA2-style dataset
df.to_csv('llama2_crick_dataset.csv', index=False)

```
- Load dataset

```
from datasets import load_dataset
dataset = load_dataset('csv', data_files='llama2_crick_dataset.csv', split="train")
```

- now login into huggingface

```
from huggingface_hub import notebook_login
notebook_login()
```

- Use the write token to read and upload the model
- Setup bits and bytes config

```
compute_dtype = getattr(torch, "float16")

quant_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=compute_dtype,
    bnb_4bit_use_double_quant=False,
)
```

- Setup the base model

```
model = AutoModelForCausalLM.from_pretrained(
    base_model,
    quantization_config=quant_config,
    device_map={"": 0}
)
model.config.use_cache = False
model.config.pretraining_tp = 1
```

- get the tokenizer

```
tokenizer = AutoTokenizer.from_pretrained(base_model, trust_remote_code=True)
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"
```

- Setup LoRA config

```
peft_params = LoraConfig(
    lora_alpha=16,
    lora_dropout=0.1,
    r=64,
    bias="none",
    task_type="CAUSAL_LM",
)
```

- Setup training parameters

```
training_params = TrainingArguments(
    output_dir="./results",
    num_train_epochs=1,
    per_device_train_batch_size=1,
    gradient_accumulation_steps=1,
    optim="paged_adamw_32bit",
    save_steps=25,
    logging_steps=25,
    learning_rate=2e-4,
    weight_decay=0.001,
    fp16=False,
    bf16=False,
    max_grad_norm=0.3,
    max_steps=-1,
    warmup_ratio=0.03,
    group_by_length=True,
    lr_scheduler_type="constant",
    report_to="tensorboard"
)
```

- Make sure batch size is set to 1
- other wise it will not fit into the GPU memory
- Setup the trainer

```
trainer = SFTTrainer(
    model=model,
    train_dataset=dataset,
    peft_config=peft_params,
    dataset_text_field="text",
    max_seq_length=None,
    tokenizer=tokenizer,
    args=training_params,
    packing=False,
)
```

- now train the model - fine tuning

```
trainer.train()
```

- Save the output to upload

```
trainer.model.save_pretrained(new_model)
trainer.tokenizer.save_pretrained(new_model)
```

- Test the output

```
logging.set_verbosity(logging.CRITICAL)

prompt = "What is Mayank Agarwal's batting average?"
pipe = pipeline(task="text-generation", model=model, tokenizer=tokenizer, max_length=200)
result = pipe(f"<s>[INST] {prompt} [/INST]")
print(result[0]['generated_text'])
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/crickllama2-1.jpg 'RagChat')

- now upload to huggingface

```
from peft import PeftModel
```

- Setup the model to upload

```
# Reload model in FP16 and merge it with LoRA weights
load_model = AutoModelForCausalLM.from_pretrained(
    base_model,
    low_cpu_mem_usage=True,
    return_dict=True,
    torch_dtype=torch.float16,
    device_map={"": 0},
)

model = PeftModel.from_pretrained(load_model, new_model)
model = model.merge_and_unload()

# Reload tokenizer to save it
tokenizer = AutoTokenizer.from_pretrained(base_model, trust_remote_code=True)
tokenizer.add_special_tokens({'pad_token': '[PAD]'})
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"
```

- login into huggingface

```
from huggingface_hub import notebook_login

notebook_login()
```

- setup up the model repo name

```
hugginfacemoderepo = 'Balab2021/phi2'
new_model = "phi2-chat-g"
```

- upload the model to hub

```
model.push_to_hub(new_model, use_temp_dir=False)
tokenizer.push_to_hub(new_model, use_temp_dir=False)
```