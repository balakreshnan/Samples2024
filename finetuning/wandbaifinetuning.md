# Fine tuning phi 2 with Azure Machine learning and upload to wandb ai

## Introduction

- Let's use Azure machine learning to fine tune the phi 2 model and upload to Wandb.ai.
- Using open source dataset for public documentation
- Using microsoft phi-2 model
- Log the training to wandb.ai

## Requirements

- Azure Account
- Azure Machine Learning Service
- Compute instance with GPU compute
- using Standard_NC48ads_A100_v4 (48 cores, 440 GB RAM, 128 GB disk)
- Model size is 6GB
- So we need enough memory to load the model and train
- Need to authorize notebook to Azure ML

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
- Download the utilities from wandb.ai

```
!wget https://github.com/wandb/edu/raw/main/llm-training-course/colab/utils.py
```

```
import wandb
wandb.init(project="alpaca_ft", # the project I am working on
           job_type="train",
           tags=["hf_sft_lora", "3b"]) # the Hyperparameters I want to keep track of
artifact = wandb.use_artifact('capecape/alpaca_ft/alpaca_gpt4_splitted:v4', type='dataset')
artifact_dir = artifact.download()
```

- Load dataset

```
from datasets import load_dataset
alpaca_ds = load_dataset("json", data_dir=artifact_dir)
```

```
# Let's subsample the training and test dataset - you may want to switch to full dataset in your experiments
alpaca_ds['train'] = alpaca_ds['train'].select(range(512))
alpaca_ds['test'] = alpaca_ds['test'].select(range(10))
```

- Setup prompt information

```
def prompt_no_input(row):
    return ("Below is an instruction that describes a task. "
            "Write a response that appropriately completes the request.\n\n"
            "### Instruction:\n{instruction}\n\n### Response:\n{output}").format_map(row)

def prompt_input(row):
    return ("Below is an instruction that describes a task, paired with an input that provides further context. "
            "Write a response that appropriately completes the request.\n\n"
            "### Instruction:\n{instruction}\n\n### Input:\n{input}\n\n### Response:\n{output}").format_map(row)

def create_prompt(row):
    return prompt_no_input(row) if row["input"] == "" else prompt_input(row)
```

```
train_dataset = alpaca_ds["train"]
eval_dataset = alpaca_ds["test"]
```

- now login into huggingface

```
from huggingface_hub import notebook_login
notebook_login()
```

- import necessary libraries

```
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
```

- Setup the model

```
#model_id = 'openlm-research/open_llama_3b_v2'
model_id = 'microsoft/phi-2'
#model_id = 'meta-llama/Llama-2-7b-hf'
```

- Use the write token to read and upload the model
- Setup peft LoRa config

```
from peft import LoraConfig, get_peft_model

peft_config = LoraConfig(
    r=64,  # the rank of the LoRA matrices
    lora_alpha=16, # the weight
    lora_dropout=0.1, # dropout to add to the LoRA layers
    bias="none", # add bias to the nn.Linear layers?
    task_type="CAUSAL_LM",
    target_modules=["q_proj", "k_proj","v_proj","o_proj"], # the name of the layers to add LoRA
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

- Now bits and bytes config

```
from transformers import BitsAndBytesConfig

bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_use_double_quant=True,
    bnb_4bit_compute_dtype=torch.float16
)
```

- model config

```
model_kwargs = dict(
    device_map={"" : 0},
    trust_remote_code=True,
    # low_cpu_mem_usage=True,
    torch_dtype=torch.float16,
    # use_flash_attention_2=True,
    use_cache=False,
    quantization_config=bnb_config,
)
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

- import the necessary libraries

```
from transformers import TrainingArguments
from trl import SFTTrainer
```

- Setup batch runs

```
batch_size = 2
gradient_accumulation_steps = 16
num_train_epochs = 10
```

- Setup training parameters

```
output_dir = "./output/"
training_args = TrainingArguments(
    num_train_epochs=num_train_epochs,
    output_dir=output_dir,
    per_device_train_batch_size=batch_size,
    per_device_eval_batch_size=batch_size,
    fp16=True,
    learning_rate=2e-4,
    lr_scheduler_type="cosine",
    warmup_ratio=0.1,
    gradient_accumulation_steps=gradient_accumulation_steps,
    gradient_checkpointing=True,
    gradient_checkpointing_kwargs=dict(use_reentrant=False),
    evaluation_strategy="epoch",
    logging_strategy="steps",
    logging_steps=1,
    save_strategy="epoch",
    report_to="wandb",
)
```

- Make sure batch size is set to 1
- other wise it will not fit into the GPU memory
- Setup the trainer

```
from utils import LLMSampleCB

trainer = SFTTrainer(
    model=model_id,
    model_init_kwargs=model_kwargs,
    train_dataset=train_dataset,
    eval_dataset=eval_dataset,
    packing=True,
    max_seq_length=1024,
    args=training_args,
    formatting_func=create_prompt,
    peft_config=peft_config,
)
```

```
# remove answers
def create_prompt_no_anwer(row):
    row["output"] = ""
    return {"text": create_prompt(row)}

test_dataset = eval_dataset.map(create_prompt_no_anwer)
```

- now train the model - fine tuning

```
wandb_callback = LLMSampleCB(trainer, test_dataset, num_samples=10, max_new_tokens=256)
trainer.add_callback(wandb_callback)
trainer.train()
wandb.finish()
```

- Done