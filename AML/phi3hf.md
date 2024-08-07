# PHI 3 128K using Hugging Face in Azure Machine Learning - Run Local

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

- Provide the Hugging Face API key

```
import torch 
from transformers import AutoModelForCausalLM, AutoTokenizer, pipeline 
```

- Download the model meta data

```
torch.random.manual_seed(0) 
model = AutoModelForCausalLM.from_pretrained( 
    "microsoft/Phi-3-mini-128k-instruct",  
    device_map="cuda",  
    torch_dtype="auto",  
    trust_remote_code=True,  
) 
```

- Download the tokenizer

```
tokenizer = AutoTokenizer.from_pretrained("microsoft/Phi-3-mini-128k-instruct") 
```

- now set the messages to generate

```
messages = [ 
    {"role": "system", "content": "You are a helpful AI assistant."}, 
    {"role": "user", "content": "Can you create a Statement of work for a Gen AI Solution?"}, 
] 
```

- now generate the response

```
generation_args = { 
    "max_new_tokens": 1000, 
    "return_full_text": False, 
    "temperature": 0.0, 
    "do_sample": False, 
} 

output = pipe(messages, **generation_args) 
print(output[0]['generated_text']) 
```

- Here is the output

```
 Title: Statement of Work for Gen AI Solution Implementation

1.0 Introduction

This Statement of Work (SOW) outlines the requirements, scope, deliverables, and timeline for the implementation of a Generative Artificial Intelligence (Gen AI) solution. The purpose of this project is to develop a customized Gen AI solution that meets the specific needs of our organization.

2.0 Project Scope

2.1 Project Objectives

- To develop a Gen AI solution that can generate high-quality, customized content for our organization.
- To integrate the Gen AI solution with our existing systems and infrastructure.
- To ensure the Gen AI solution is scalable, secure, and compliant with industry standards and regulations.

2.2 Deliverables

- A fully functional Gen AI solution that meets the project objectives.
- A comprehensive documentation package that includes system architecture, design specifications, and user manuals.
- A training program for end-users to effectively utilize the Gen AI solution.

2.3 Project Milestones

- Project initiation and requirements gathering: 2 weeks
- System design and architecture: 4 weeks
- Development and testing: 8 weeks
- Integration and deployment: 4 weeks
- Training and user acceptance testing: 2 weeks
- Project closure and documentation: 2 weeks

3.0 Project Team

3.1 Project Manager

- Responsible for overseeing the project, ensuring that it meets the objectives and timeline.
- Coordinates with the project team, stakeholders, and vendors.
- Manages project risks and issues.

3.2 Gen AI Solution Architect

- Responsible for designing the system architecture and ensuring that the Gen AI solution meets the project objectives.
- Works closely with the development team to ensure that the solution is scalable, secure, and compliant with industry standards and regulations.

3.3 Gen AI Solution Developer

- Responsible for developing the Gen AI solution, including the algorithms, models, and data pipelines.
- Works closely with the Gen AI Solution Architect to ensure that the solution meets the project objectives.

3.4 Gen AI Solution Tester

- Responsible for testing the Gen AI solution to ensure that it meets the project objectives and is free of defects.
- Works closely with the Gen AI Solution Developer to identify and fix any issues.

3.5 Gen AI Solution Trainer

- Responsible for developing and delivering a training program for end-users to effectively utilize the Gen AI solution.
- Works closely with the Gen AI Solution Developer to understand the solution's capabilities and limitations.

4.0 Project Deliverables

4.1 Gen AI Solution

- A fully functional Gen AI solution that meets the project objectives.
- The solution should be able to generate high-quality, customized content for our organization.

4.2 System Architecture and Design Documentation

- A comprehensive documentation package that includes system architecture, design specifications, and user manuals.

4.3 Training Program

- A training program for end-users to effectively utilize the Gen AI solution.

4.4 Project Closure and Documentation

- A project closure report that includes a summary of the project, lessons learned, and recommendations for future projects.

5.0 Project Timeline

- Project initiation and requirements gathering: 2 weeks
- System design and architecture: 4 weeks
- Development and testing: 8 weeks
- Integration and deployment: 4 weeks
- Training and user acceptance testing: 2 weeks
- Project closure and documentation: 2 weeks

6.0 Budget

The total budget for this project is $XXX,XXX. The budget will be allocated as follows:

- System design and architecture: $XXX,XXX
- Development and testing: $XXX,XXX
- Integration and deployment: $XXX,XXX
- Training and user acceptance testing: $XXX,XXX
- Project closure and documentation: $XXX,XXX

7.0 Acceptance Criteria

- The Gen AI solution meets the project objectives and is fully functional.
- The system architecture and design documentation are comprehensive and meet industry standards and regulations.
- The training program is effective and enables end-users to effectively util
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/phi3hf-1.jpg 'Phi3 128K instruct Model')