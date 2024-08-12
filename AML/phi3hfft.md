# PHI 3 fine tuning locally from Hugging Face (Simulated in Azure Machine learning)

## Introduction

- Run PHI 3 in local GPU Compute instance
- Use Hugging Face Transformers
- Use Azure Machine Learning for logging and tracking
- I am using A100 4 GPU machine with 128GB RAM - Standard_NC96ads_A100_v4 (96 cores, 880 GB RAM, 256 GB disk)
- I am using 4 GPUs for this run

## Pre-requisites

- Azure Machine Learning Workspace
- Azure subcription
- Hugging Face Account
- API key to access Hugging Face model
- Hugging face api should have write access
- Following this tutorial: https://github.com/microsoft/Phi-3CookBook/blob/main/code/04.Finetuning/Phi-3-finetune-lora-python.ipynb

## Steps

- install all the libraries

```
%pip install python-dotenv
```

```
%pip install huggingface_hub
```

```
%pip install ipywidgets
```

```
%pip install transformers
```

```
%pip install accelerate==0.31.0
```

```
%pip install torch
```

```
%pip install flash_attn
```

```
%pip install absl-py nltk rouge_score
```

```
%pip install peft
```

```
%pip install datasets
```

```
%pip install trl
```

```
%pip install wandb
```

- get a wandb account and login
- create a api key and save it in the env.env file

- Restart the kernel
- now load th environment variables

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Log in to Hugging Face

```
from huggingface_hub import notebook_login
notebook_login()
```

## Fine tuning Code


```
from random import randrange
import torch
from datasets import load_dataset
from peft import LoraConfig, prepare_model_for_kbit_training, TaskType, PeftModel
```

```
from transformers import (
    AutoModelForCausalLM,
    AutoTokenizer,
    BitsAndBytesConfig,
    TrainingArguments,
    set_seed,
    pipeline
)

# 'SFTTrainer' is a class from the 'trl' library that provides a trainer for soft fine-tuning.
from trl import SFTTrainer
```

- Setup the huggingface Phi 3 model

```
model_id = "microsoft/Phi-3-mini-4k-instruct"
model_name = "microsoft/Phi-3-mini-4k-instruct"
```

- Setup the dataset name

```
dataset_name = "iamtarun/python_code_instructions_18k_alpaca"
```

- Now configure the new model name

```
dataset_split= "train"
new_model = "bbphi3ftv1"
hf_model_repo="Balab2021/"+new_model
device_map = {"": 0}
```

- Setup LORA parameters

```
# The following are parameters for the LoRA (Learning from Random Architecture) model.

# 'lora_r' is the dimension of the LoRA attention.
lora_r = 16

# 'lora_alpha' is the alpha parameter for LoRA scaling.
lora_alpha = 16

# 'lora_dropout' is the dropout probability for LoRA layers.
lora_dropout = 0.05

# 'target_modules' is a list of the modules in the model that will be replaced with LoRA layers.
target_modules= ['k_proj', 'q_proj', 'v_proj', 'o_proj', "gate_proj", "down_proj", "up_proj"]

# 'set_seed' is a function that sets the seed for generating random numbers, 
# which is used for reproducibility of the results.
set_seed(1234)
```

- Load the dataset

```
# This code block is used to load a dataset from the Hugging Face Dataset Hub, print its size, and show a random example from the dataset.

# 'load_dataset' is a function from the 'datasets' library that loads a dataset from the Hugging Face Dataset Hub.
# 'dataset_name' is the name of the dataset to load, and 'dataset_split' is the split of the dataset to load (e.g., 'train', 'test').
dataset = load_dataset(dataset_name, split=dataset_split)

# The 'len' function is used to get the size of the dataset, which is then printed.
print(f"dataset size: {len(dataset)}")

# 'randrange' is a function from the 'random' module that generates a random number within the specified range.
# Here it's used to select a random example from the dataset, which is then printed.
print(dataset[randrange(len(dataset))])
```

- Data set output

```
dataset size: 18612
{'instruction': 'Compare two strings to check if they are anagrams or not in Python.', 'input': '“silent”, “listen”', 'output': 'def is_anagram(w1, w2):\n    # Check if lengths are equal\n    if len(w1) == len(w2):\n        # Sort the strings\n        s1 = sorted(w1)\n        s2 = sorted(w2)\n        # Check if sorted strings are equal\n        if s1 == s2:\n            return True\n    return False\n\n# Example\nw1 = "silent"\nw2 = "listen"\n\nprint(is_anagram(w1, w2)) #Output: True', 'prompt': 'Below is an instruction that describes a task. Write a response that appropriately completes the request.\n\n### Instruction:\nCompare two strings to check if they are anagrams or not in Python.\n\n### Input:\n“silent”, “listen”\n\n### Output:\ndef is_anagram(w1, w2):\n    # Check if lengths are equal\n    if len(w1) == len(w2):\n        # Sort the strings\n        s1 = sorted(w1)\n        s2 = sorted(w2)\n        # Check if sorted strings are equal\n        if s1 == s2:\n            return True\n    return False\n\n# Example\nw1 = "silent"\nw2 = "listen"\n\nprint(is_anagram(w1, w2)) #Output: True'}
```

- print dataset

```
dataset
```

- print the len

```
print(dataset[randrange(len(dataset))])
```

- download the tokenizer

```
# This code block is used to load a tokenizer from the Hugging Face Model Hub.

# 'tokenizer_id' is set to the 'model_id', which is the identifier for the pre-trained model.
# This assumes that the tokenizer associated with the model has the same identifier as the model.
tokenizer_id = model_id

# 'AutoTokenizer.from_pretrained' is a method that loads a tokenizer from the Hugging Face Model Hub.
# 'tokenizer_id' is passed as an argument to specify which tokenizer to load.
tokenizer = AutoTokenizer.from_pretrained(tokenizer_id)

# 'tokenizer.padding_side' is a property that specifies which side to pad when the input sequence is shorter than the maximum sequence length.
# Setting it to 'right' means that padding tokens will be added to the right (end) of the sequence.
# This is done to prevent warnings that can occur when the padding side is not explicitly set.
tokenizer.padding_side = 'right'
```

- function to format the dataset to match the model

```
# This code block defines two functions that are used to format the dataset for training a chat model.

# 'create_message_column' is a function that takes a row from the dataset and returns a dictionary 
# with a 'messages' key and a list of 'user' and 'assistant' messages as its value.
def create_message_column(row):
    # Initialize an empty list to store the messages.
    messages = []
    
    # Create a 'user' message dictionary with 'content' and 'role' keys.
    user = {
        "content": f"{row['instruction']}\n Input: {row['input']}",
        "role": "user"
    }
    
    # Append the 'user' message to the 'messages' list.
    messages.append(user)
    
    # Create an 'assistant' message dictionary with 'content' and 'role' keys.
    assistant = {
        "content": f"{row['output']}",
        "role": "assistant"
    }
    
    # Append the 'assistant' message to the 'messages' list.
    messages.append(assistant)
    
    # Return a dictionary with a 'messages' key and the 'messages' list as its value.
    return {"messages": messages}

# 'format_dataset_chatml' is a function that takes a row from the dataset and returns a dictionary 
# with a 'text' key and a string of formatted chat messages as its value.
def format_dataset_chatml(row):
    # 'tokenizer.apply_chat_template' is a method that formats a list of chat messages into a single string.
    # 'add_generation_prompt' is set to False to not add a generation prompt at the end of the string.
    # 'tokenize' is set to False to return a string instead of a list of tokens.
    return {"text": tokenizer.apply_chat_template(row["messages"], add_generation_prompt=False, tokenize=False)}
```

- convert the dataset to the model format

```
# This code block is used to prepare the 'dataset' for training a chat model.

# 'dataset.map' is a method that applies a function to each example in the 'dataset'.
# 'create_message_column' is a function that formats each example into a 'messages' format suitable for a chat model.
# The result is a new 'dataset_chatml' with the formatted examples.
dataset_chatml = dataset.map(create_message_column)

# 'dataset_chatml.map' is a method that applies a function to each example in the 'dataset_chatml'.
# 'format_dataset_chatml' is a function that further formats each example into a single string of chat messages.
# The result is an updated 'dataset_chatml' with the further formatted examples.
dataset_chatml = dataset_chatml.map(format_dataset_chatml)
```

- Display the data set

```
dataset_chatml[0]

dataset_chatml = dataset_chatml.train_test_split(test_size=0.05, seed=1234)

# This line of code is used to display the structure of the 'dataset_chatml' after the split.
# It will typically show information such as the number of rows in the training set and the testing set.
dataset_chatml
```

- Set the floating point

```
# This code block is used to set the compute data type and attention implementation based on whether bfloat16 is supported on the current CUDA device.

# 'torch.cuda.is_bf16_supported()' is a function that checks if bfloat16 is supported on the current CUDA device.
# If bfloat16 is supported, 'compute_dtype' is set to 'torch.bfloat16' and 'attn_implementation' is set to 'flash_attention_2'.
if torch.cuda.is_bf16_supported():
  compute_dtype = torch.bfloat16
  attn_implementation = 'flash_attention_2'
# If bfloat16 is not supported, 'compute_dtype' is set to 'torch.float16' and 'attn_implementation' is set to 'sdpa'.
else:
  compute_dtype = torch.float16
  attn_implementation = 'sdpa'

# This line of code is used to print the value of 'attn_implementation', which indicates the chosen attention implementation.
print(attn_implementation)
```

- Download the model

```
# This code block is used to load a pre-trained model and its associated tokenizer from the Hugging Face Model Hub.

# 'model_name' is set to the identifier of the pre-trained model.
model_name = "microsoft/Phi-3-mini-4k-instruct"

# 'AutoTokenizer.from_pretrained' is a method that loads a tokenizer from the Hugging Face Model Hub.
# 'model_id' is passed as an argument to specify which tokenizer to load.
# 'trust_remote_code' is set to True to trust the remote code in the tokenizer files.
# 'add_eos_token' is set to True to add an end-of-sentence token to the tokenizer.
# 'use_fast' is set to True to use the fast version of the tokenizer.
tokenizer = AutoTokenizer.from_pretrained(model_id, trust_remote_code=True, add_eos_token=True, use_fast=True)

# The padding token is set to the unknown token.
tokenizer.pad_token = tokenizer.unk_token

# The ID of the padding token is set to the ID of the unknown token.
tokenizer.pad_token_id = tokenizer.convert_tokens_to_ids(tokenizer.pad_token)

# The padding side is set to 'left', meaning that padding tokens will be added to the left (start) of the sequence.
tokenizer.padding_side = 'left'

# 'AutoModelForCausalLM.from_pretrained' is a method that loads a pre-trained model for causal language modeling from the Hugging Face Model Hub.
# 'model_id' is passed as an argument to specify which model to load.
# 'torch_dtype' is set to the compute data type determined earlier.
# 'trust_remote_code' is set to True to trust the remote code in the model files.
# 'device_map' is passed as an argument to specify the device mapping for distributed training.
# 'attn_implementation' is set to the attention implementation determined earlier.
model = AutoModelForCausalLM.from_pretrained(
          model_id, torch_dtype=compute_dtype, trust_remote_code=True, device_map=device_map,
          attn_implementation=attn_implementation
)
```

- Setup training parameters

```
args = TrainingArguments(
        output_dir="./phi-3-mini-LoRA",
        evaluation_strategy="steps",
        do_eval=True,
        optim="adamw_torch",
        per_device_train_batch_size=8,
        gradient_accumulation_steps=4,
        per_device_eval_batch_size=8,
        log_level="debug",
        save_strategy="epoch",
        logging_steps=100,
        learning_rate=1e-4,
        fp16 = not torch.cuda.is_bf16_supported(),
        bf16 = torch.cuda.is_bf16_supported(),
        eval_steps=100,
        num_train_epochs=3,
        warmup_ratio=0.1,
        lr_scheduler_type="linear",
        report_to="wandb",
        seed=42,
)

peft_config = LoraConfig(
        r=lora_r,
        lora_alpha=lora_alpha,
        lora_dropout=lora_dropout,
        task_type=TaskType.CAUSAL_LM,
        target_modules=target_modules,
)
```

- Login into wandb.ai

```
import wandb

# 'wandb.login()' is a method that logs you into your Weights & Biases account.
# If you're not already logged in, it will prompt you to log in.
# Once you're logged in, you can use Weights & Biases to track and visualize your experiments.
wandb.login()
```

- Set the project

```
project_name = "Phi3-mini-ft-python-code"

# 'wandb.init' is a method that initializes a new Weights & Biases run.
# 'project' is set to 'project_name', meaning that the run will be associated with this project.
# 'name' is set to "phi-3-mini-ft-py-3e", which is the name of the run.
# Each run has a unique name which can be used to identify it in the Weights & Biases dashboard.
wandb.init(project=project_name, name = "phi-3-mini-ft-py-3e")
```

- Setup the trainer

```
trainer = SFTTrainer(
        model=model,
        train_dataset=dataset_chatml['train'],
        eval_dataset=dataset_chatml['test'],
        peft_config=peft_config,
        dataset_text_field="text",
        max_seq_length=512,
        tokenizer=tokenizer,
        args=args,
)
```

- Train the model

```
trainer.train()

# 'trainer.save_model()' is a method that saves the trained model locally.
# The model will be saved in the directory specified by 'output_dir' in the training arguments.
trainer.save_model()
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-1.jpg 'Phi3 128K instruct Model')

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-2.jpg 'Phi3 128K instruct Model')

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-3.jpg 'Phi3 128K instruct Model')

- Save the adapter to Hugging Face

```
trainer.push_to_hub("Balab2021/adapter-phi-3-mini-py_code")
```

- Clean up the temp and merge model

```
# This code block is used to free up GPU memory.

# 'del model' and 'del trainer' are used to delete the 'model' and 'trainer' objects. 
# This removes the references to these objects, allowing Python's garbage collector to free up the memory they were using.

del model
del trainer

# 'import gc' is used to import Python's garbage collector module.
import gc

# 'gc.collect()' is a method that triggers a full garbage collection, which can help to free up memory.
# It's called twice here to ensure that all unreachable objects are collected.
gc.collect()
gc.collect()
```

```
torch.cuda.empty_cache()
gc.collect()
```

- Now to merge the model

```
from peft import AutoPeftModelForCausalLM

# 'AutoPeftModelForCausalLM.from_pretrained' is a method that loads a pre-trained model (adapter model) and its base model.
#  The adapter model is loaded from 'args.output_dir', which is the directory where the trained model was saved.
# 'low_cpu_mem_usage' is set to True, which means that the model will use less CPU memory.
# 'return_dict' is set to True, which means that the model will return a 'ModelOutput' (a named tuple) instead of a plain tuple.
# 'torch_dtype' is set to 'torch.bfloat16', which means that the model will use bfloat16 precision for its computations.
# 'trust_remote_code' is set to True, which means that the model will trust and execute remote code.
# 'device_map' is the device map that will be used by the model.

new_model = AutoPeftModelForCausalLM.from_pretrained(
    args.output_dir,
    low_cpu_mem_usage=True,
    return_dict=True,
    torch_dtype=torch.bfloat16, #torch.float16,
    trust_remote_code=True,
    device_map=device_map,
)

# 'new_model.merge_and_unload' is a method that merges the model and unloads it from memory.
# The merged model is stored in 'merged_model'.

merged_model = new_model.merge_and_unload()

# 'merged_model.save_pretrained' is a method that saves the merged model.
# The model is saved in the directory "merged_model".
# 'trust_remote_code' is set to True, which means that the model will trust and execute remote code.
# 'safe_serialization' is set to True, which means that the model will use safe serialization.

merged_model.save_pretrained("merged_model", trust_remote_code=True, safe_serialization=True)

# 'tokenizer.save_pretrained' is a method that saves the tokenizer.
# The tokenizer is saved in the directory "merged_model".

tokenizer.save_pretrained("merged_model")
```

- push the model to huging face

```
# This code block is used to push the merged model and the tokenizer to the Hugging Face Model Hub.

# 'merged_model.push_to_hub' is a method that pushes the merged model to the Hugging Face Model Hub.
# 'hf_model_repo' is the name of the repository on the Hugging Face Model Hub where the model will be saved.
merged_model.push_to_hub(hf_model_repo)

# 'tokenizer.push_to_hub' is a method that pushes the tokenizer to the Hugging Face Model Hub.
# 'hf_model_repo' is the name of the repository on the Hugging Face Model Hub where the tokenizer will be saved.
tokenizer.push_to_hub(hf_model_repo)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-4.jpg 'Phi3 128K instruct Model')

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-5.jpg 'Phi3 128K instruct Model')

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-6.jpg 'Phi3 128K instruct Model')

- display the repo 

```
hf_model_repo
```

- check repo exist if not create

```
hf_model_repo = 'Balab2021/bbphi3ftv1' if not hf_model_repo else hf_model_repo
```

- load the model

```
# This code block is used to load the model and tokenizer from the Hugging Face Model Hub.

# 'torch' is a library that provides a wide range of functionalities for tensor computations with strong GPU acceleration support.
# 'AutoTokenizer' and 'AutoModelForCausalLM' are classes from the 'transformers' library that provide a tokenizer and a causal language model, respectively.
# 'set_seed' is a function from the 'transformers' library that sets the seed for generating random numbers, which can be used for reproducibility.

import torch
from transformers import AutoTokenizer, AutoModelForCausalLM, set_seed

# 'set_seed(1234)' sets the seed for generating random numbers to 1234.
set_seed(1234)  # For reproducibility

# 'AutoTokenizer.from_pretrained' is a method that loads a pre-trained tokenizer.
# The tokenizer is loaded from 'hf_model_repo', which is the name of the repository on the Hugging Face Model Hub where the tokenizer was saved.
# 'trust_remote_code' is set to True, which means that the tokenizer will trust and execute remote code.

tokenizer = AutoTokenizer.from_pretrained(hf_model_repo,trust_remote_code=True)

# 'AutoModelForCausalLM.from_pretrained' is a method that loads a pre-trained causal language model.
# The model is loaded from 'hf_model_repo', which is the name of the repository on the Hugging Face Model Hub where the model was saved.
# 'trust_remote_code' is set to True, which means that the model will trust and execute remote code.
# 'torch_dtype' is set to "auto", which means that the model will automatically choose the data type for its computations.
# 'device_map' is set to "cuda", which means that the model will use the CUDA device for its computations.

model = AutoModelForCausalLM.from_pretrained(hf_model_repo, trust_remote_code=True, torch_dtype="auto", device_map="cuda")
```

- Load the test data

```
# This code block is used to prepare the dataset for model training.

# 'dataset.map(create_message_column)' applies the 'create_message_column' function to each element in the 'dataset'.
# This function is used to create a new column in the dataset.
dataset_chatml = dataset.map(create_message_column)

# 'dataset_chatml.map(format_dataset_chatml)' applies the 'format_dataset_chatml' function to each element in 'dataset_chatml'.
# This function is used to format the dataset in a way that is suitable for chat ML.
dataset_chatml = dataset_chatml.map(format_dataset_chatml)

# 'dataset_chatml.train_test_split(test_size=0.05, seed=1234)' splits 'dataset_chatml' into a training set and a test set.
# 'test_size=0.05' means that 5% of the data will be used for the test set.
# 'seed=1234' is used for reproducibility.
dataset_chatml = dataset_chatml.train_test_split(test_size=0.05, seed=1234)

# 'dataset_chatml' is printed to the console to inspect its contents.
dataset_chatml
```

```
dataset_chatml['test'][0]
```

- now to inference the model using pipeline template

```
pipe = pipeline("text-generation", model=model, tokenizer=tokenizer)
```

```
# The function returns the first generated response.
# The response is stripped of the prompt and any leading or trailing whitespace.
def test_inference(prompt):
    prompt = pipe.tokenizer.apply_chat_template([{"role": "user", "content": prompt}], tokenize=False, add_generation_prompt=True)
    outputs = pipe(prompt, max_new_tokens=256, do_sample=True, num_beams=1, temperature=0.3, top_k=50, top_p=0.95,
                   max_time= 180) #, eos_token_id=eos_token)
    return outputs[0]['generated_text'][len(prompt):].strip()
```

- Test the model

```
test_inference(dataset_chatml['test'][0]['messages'][0]['content'])
```

- now calculate the rouge score

```
from datasets import load_metric
rouge_metric = load_metric("rouge", trust_remote_code=True)
```

- Calculate metrics

```
def calculate_rogue(row):
    response = test_inference(row['messages'][0]['content'])
    result = rouge_metric.compute(predictions=[response], references=[row['output']], use_stemmer=True)
    result = {key: value.mid.fmeasure * 100 for key, value in result.items()}
    result['response']=response
    return result
```

```
import time
metricas = dataset_chatml['test'].select(range(0,500)).map(calculate_rogue, batched=False)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-7.jpg 'Phi3 128K instruct Model')

- print the metrics

```
import numpy as np
# 'print' is used to print the calculated means to the console.
print("Rouge 1 Mean: ",np.mean(metricas['rouge1']))
print("Rouge 2 Mean: ",np.mean(metricas['rouge2']))
print("Rouge L Mean: ",np.mean(metricas['rougeL']))
print("Rouge Lsum Mean: ",np.mean(metricas['rougeLsum']))
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-8.jpg 'Phi3 128K instruct Model')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-9.jpg 'Phi3 128K instruct Model')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-10.jpg 'Phi3 128K instruct Model')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-11.jpg 'Phi3 128K instruct Model')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hfft-12.jpg 'Phi3 128K instruct Model')