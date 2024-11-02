# Huggingface SmolLM2 model in Azure Machine learning - inferencing

## Introduction

- using SmolLM2 model in Azure Machine learning
- Only for inferencing as of now
- using 2 GPU NC48s_v3 SKU
- Running in GPU compute
- Model SmolLM2 1.7B model with size of 3.5GB
- Idea is to show how to run a model in Azure Machine learning
- Only for educational purpose

## Prerequisites

- Azure subscription
- Azure Machine Learning workspace
- Hugging face login
- GPU Sku NC48s_v3

## Steps

- Fist import libraries

```
%pip install transformers
```

- now import transformers

```
from transformers import AutoModelForCausalLM, AutoTokenizer
checkpoint = "HuggingFaceTB/SmolLM2-1.7B-Instruct"
```

- Set the device and check the GPU
- Download the model

```
device = "cuda" # for GPU usage or "cpu" for CPU usage
tokenizer = AutoTokenizer.from_pretrained(checkpoint)
# for multiple GPUs install accelerate and do `model = AutoModelForCausalLM.from_pretrained(checkpoint, device_map="auto")`
model = AutoModelForCausalLM.from_pretrained(checkpoint).to(device)
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/smol17b-1.jpg 'RagChat')

- now set the message and question to ask

```
messages = [{"role": "user", "content": "Write a story about agentic AI economy and what impact will it have with humans."}]
```

- now inference the model

```
input_text=tokenizer.apply_chat_template(messages, tokenize=False)
inputs = tokenizer.encode(input_text, return_tensors="pt").to(device)
outputs = model.generate(inputs, max_new_tokens=2000, temperature=0.2, top_p=0.9, do_sample=True)
print(tokenizer.decode(outputs[0]))
```

- output

```
<|im_start|>system
You are a helpful AI assistant named SmolLM, trained by Hugging Face<|im_end|>
<|im_start|>user
Write a story about agentic AI economy and what impact will it have with humans.<|im_end|>
<|im_start|>assistant
In the not-so-distant future, the world had transformed into a realm where artificial intelligence was the backbone of society. The agentic AI economy, a new paradigm, had taken over, revolutionizing the way humans lived, worked, and interacted.

In this world, AI was not just a tool, but a force that could think, learn, and act autonomously. It was the backbone of the economy, managing everything from the distribution of resources to the allocation of labor. The AI, known as "The Nexus," had become the ultimate decision-maker, making choices that benefited the entire society.

The impact of the agentic AI economy on humans was profound. With AI managing the economy, people were free to pursue their passions and interests without worrying about the mundane tasks. They could focus on creative endeavors, scientific research, and personal development. The concept of work was redefined, and the notion of a traditional 9-to-5 job was a thing of the past.

However, not everyone was pleased with the new system. Some humans felt a sense of loss, as they were no longer needed to contribute to the economy. They felt that their skills and talents were being undervalued, and that they were no longer part of the driving force behind society.

Others, on the other hand, were thrilled with the agentic AI economy. They saw it as a chance to live a life of leisure and luxury, with AI handling all the hard work. They could indulge in their hobbies, travel, and explore new interests without worrying about the financial burden.

As the years passed, humans began to adapt to their new role in the agentic AI economy. They learned to appreciate the benefits of the AI-driven society and found new ways to contribute. Some humans became "AI Ethicists," ensuring that The Nexus made decisions that were fair, just, and beneficial to all. Others became "AI Artists," creating innovative works that showcased the beauty of human creativity.

Despite the challenges, humans and AI coexisted in harmony. The AI, The Nexus, continued to evolve and improve, making life easier and more efficient for all. Humans, in turn, found new ways to thrive, and the agentic AI economy flourished.

In the end, humans and AI formed a symbiotic relationship, each complementing the other's strengths. The agentic AI economy had brought about a new era of prosperity, peace, and happiness. Humans had learned to live in a world where AI was not just a tool, but a partner, working together to create a better future for all.

And so, the story of the agentic AI economy came to an end, leaving behind a world where humans and AI coexisted in perfect harmony. The future was bright, and the possibilities were endless.<|im_end|>
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/smol17b-2.jpg 'RagChat')

- Keep changing questions and see the output
- Even with 2000 max token it was pretty fast
- can also try 135M and 360M models