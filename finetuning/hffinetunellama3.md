# Fine tuning LLama3 model - meta-llama/Meta-Llama-3-8B-Instruct using Azure Machine Learning

## Use Case

- Fine tuning Meta-Llama-3-8B-Instruct model
- Using compute instance
- Using GPU Compute SKU: Standard_NC48ads_A100
- Using Azure Machine Learning
- Idea here is to learn to fine tune a open source Meta's LLama 7BHF model using Azure Machine Learning
- This model is a large language model with 8B parameters
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

- Login into Hugging face to download model

```
from huggingface_hub import notebook_login
#you need the key from hugging face to authenticate
notebook_login()
```

- Load LLama 7BHF model from Meta

``` 
base_model = "meta-llama/Meta-Llama-3-8B-Instruct"

# New instruction dataset
guanaco_dataset = "mlabonne/guanaco-llama2-1k"

# Fine-tuned model
#new_model = "Balab2021/llama3"
new_model = "llama3"
```

- load dataset

```
dataset = load_dataset(guanaco_dataset, split="train")
```

```
compute_dtype = getattr(torch, "float16")

quant_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=compute_dtype,
    bnb_4bit_use_double_quant=False,
)

model = AutoModelForCausalLM.from_pretrained(
    base_model,
    quantization_config=quant_config,
    device_map={"": 0}
)
model.config.use_cache = False
model.config.pretraining_tp = 1

tokenizer = AutoTokenizer.from_pretrained(base_model, trust_remote_code=True)
tokenizer.pad_token = tokenizer.eos_token
tokenizer.padding_side = "right"
```

- peft config

```
peft_params = LoraConfig(
    lora_alpha=16,
    lora_dropout=0.1,
    r=64,
    bias="none",
    task_type="CAUSAL_LM",
)
```

- Training parameters

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

- SFT trainer

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

- now start the training

```
trainer.train()
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama31.jpg 'RagChat')

- Save the model

```trainer.model.save_pretrained(new_model)
trainer.tokenizer.save_pretrained(new_model)
```

- Test the output

```
logging.set_verbosity(logging.CRITICAL)

prompt = "Who is Leonardo Da Vinci?"
pipe = pipeline(task="text-generation", model=model, tokenizer=tokenizer, max_length=200)
result = pipe(f"<s>[INST] {prompt} [/INST]")
print(result[0]['generated_text'])
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama32.jpg 'RagChat')

- now load the model to upload

```
from peft import PeftModel

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

- upload the model to hugging face

```
model.push_to_hub(new_model, use_temp_dir=False)
tokenizer.push_to_hub(new_model, use_temp_dir=False)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/llama33.jpg 'RagChat')