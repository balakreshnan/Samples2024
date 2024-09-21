# Florence 2 Fine tuning for images

## Introduction

- Idea to use florence 2 model to detect objects
- Fine tune for certain industry specific objects
- this allows us to customize to industry needs
- Intent to show how to fine tune the model for specific objects
- It's not a production system nor solves a real world use case.

## pre-requisites

- Azure account
- Azure subscription
- Azure machine learning workspace
- Using GPU compute for finetuning
- finetuning model size is close to 1.5GB
  

## Steps

- install the required packages

```
%pip install -q datasets flash_attn timm einops
```

- Following this article to fine tune the model

```
https://huggingface.co/blog/finetune-florence2
```

- Thank you to the author

- load sample huggingface data sets for fine tuning with imaes

```
from datasets import load_dataset

data = load_dataset("HuggingFaceM4/DocumentVQA")
```

- Download base model to fine tune

```
from transformers import AutoModelForCausalLM, AutoProcessor
import torch

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

model = AutoModelForCausalLM.from_pretrained("microsoft/Florence-2-base-ft", trust_remote_code=True, revision='refs/pr/6').to(device)
processor = AutoProcessor.from_pretrained("microsoft/Florence-2-base-ft", trust_remote_code=True, revision='refs/pr/6')
```

- clear cache

```
torch.cuda.empty_cache()
```

- fun the model

```
# Function to run the model on an example
def run_example(task_prompt, text_input, image):
    prompt = task_prompt + text_input

    # Ensure the image is in RGB mode
    if image.mode != "RGB":
        image = image.convert("RGB")

    inputs = processor(text=prompt, images=image, return_tensors="pt").to(device)
    generated_ids = model.generate(
        input_ids=inputs["input_ids"],
        pixel_values=inputs["pixel_values"],
        max_new_tokens=1024,
        num_beams=3
    )
    generated_text = processor.batch_decode(generated_ids, skip_special_tokens=False)[0]
    parsed_answer = processor.post_process_generation(generated_text, task=task_prompt, image_size=(image.width, image.height))
    return parsed_answer
```

- display sample images

```
for idx in range(3):
  print(run_example("DocVQA", 'What do you see in this image?', data['train'][idx]['image']))
  display(data['train'][idx]['image'].resize([350, 350]))
```

- Data set class

```
from torch.utils.data import Dataset

class DocVQADataset(Dataset):
    def __init__(self, data):
        self.data = data

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        example = self.data[idx]
        question = "<DocVQA>" + example['question']
        first_answer = example['answers'][0]
        image = example['image']
        if image.mode != "RGB":
            image = image.convert("RGB")
        return question, first_answer, image
```

- Split data sets

```
import os
from torch.utils.data import DataLoader
from tqdm import tqdm
from transformers import (AdamW, AutoProcessor, get_scheduler)

def collate_fn(batch):
    questions, answers, images = zip(*batch)
    inputs = processor(text=list(questions), images=list(images), return_tensors="pt", padding=True).to(device)
    return inputs, answers

# Create datasets
train_dataset = DocVQADataset(data['train'])
val_dataset = DocVQADataset(data['validation'])

# Create DataLoader
batch_size = 6
num_workers = 0

train_loader = DataLoader(train_dataset, batch_size=batch_size, collate_fn=collate_fn, num_workers=num_workers, shuffle=True)
val_loader = DataLoader(val_dataset, batch_size=batch_size, collate_fn=collate_fn, num_workers=num_workers)
```

- Now Train function to fine tune

```
def train_model(train_loader, val_loader, model, processor, epochs=10, lr=1e-6):
    optimizer = AdamW(model.parameters(), lr=lr)
    num_training_steps = epochs * len(train_loader)
    lr_scheduler = get_scheduler(
        name="linear",
        optimizer=optimizer,
        num_warmup_steps=0,
        num_training_steps=num_training_steps,
    )

    for epoch in range(epochs):
        model.train()
        train_loss = 0
        i = -1
        for batch in tqdm(train_loader, desc=f"Training Epoch {epoch + 1}/{epochs}"):
            i += 1
            inputs, answers = batch

            input_ids = inputs["input_ids"]
            pixel_values = inputs["pixel_values"]
            labels = processor.tokenizer(text=answers, return_tensors="pt", padding=True, return_token_type_ids=False).input_ids.to(device)

            outputs = model(input_ids=input_ids, pixel_values=pixel_values, labels=labels)
            loss = outputs.loss

            loss.backward()
            optimizer.step()
            lr_scheduler.step()
            optimizer.zero_grad()

            train_loss += loss.item()

        avg_train_loss = train_loss / len(train_loader)
        print(f"Average Training Loss: {avg_train_loss}")

        # Validation phase
        model.eval()
        val_loss = 0
        with torch.no_grad():
            for batch in tqdm(val_loader, desc=f"Validation Epoch {epoch + 1}/{epochs}"):
                inputs, answers = batch

                input_ids = inputs["input_ids"]
                pixel_values = inputs["pixel_values"]
                labels = processor.tokenizer(text=answers, return_tensors="pt", padding=True, return_token_type_ids=False).input_ids.to(device)

                outputs = model(input_ids=input_ids, pixel_values=pixel_values, labels=labels)
                loss = outputs.loss

                val_loss += loss.item()

        avg_val_loss = val_loss / len(val_loader)
        print(f"Average Validation Loss: {avg_val_loss}")

        # Save model checkpoint
        output_dir = f"./model_checkpoints/epoch_{epoch+1}"
        os.makedirs(output_dir, exist_ok=True)
        model.save_pretrained(output_dir)
        processor.save_pretrained(output_dir)
```

- Log into huggingface to upload the model

```
from huggingface_hub import notebook_login

notebook_login()
```

- Set vision parameters

```
for param in model.vision_tower.parameters():
  param.is_trainable = False
```

- Train the model

```
train_model(train_loader, val_loader, model, processor, epochs=1)
```

- Wait for 3 hours for 1 epoch to complete
- for 2 epochs it will take 6 hours

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/florence2-1.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/florence2-4.jpg 'RagChat')

- now push to huggingface

```
model.push_to_hub("Balab2021/Florence-2-FT-DocVQA")
processor.push_to_hub("Balab2021/Florence-2-FT-DocVQA")
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/florence2-5.jpg 'RagChat')

- go to https://huggingface.co/Balab2021/Florence-2-FT-DocVQA to see the model

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/florence2-6.jpg 'RagChat')