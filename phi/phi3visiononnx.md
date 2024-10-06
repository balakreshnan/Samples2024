# How to Run Phi3 Vision on ONNX in python

## Introduction

- Idea to use phi3 vision model to detect objects
- See how we can run in local GPU
- Need to have CUDA installed
- Ability to run phi3 in local edge with GPU
- It is also possible CPU/GPU
- Ask any questions to the image

## Pre-requisites

- CUDA installed - https://developer.nvidia.com/cuda-12-2-0-download-archive
- NVIDIA GPU
- cu-DNN installed - https://developer.nvidia.com/cudnn-9-3-0-download-archive?target_os=Windows
- Running in Windows 11 surface book 3
- Python 3.12
- Make sure Path is set for CUDA and cu-DNN
- In case if cu-DNN is not recognized, copy the dll from cu-DNN path to cuda path bin and lib folders
- Model is available - https://huggingface.co/microsoft/Phi-3-vision-128k-instruct-onnx-cuda
- Tutorial i am following - https://onnxruntime.ai/docs/genai/tutorials/phi3-v.html#run-with-nvidia-cuda
- NVIDIA RTX A2000 GPU

## Code

- Install the required packages

```
pip install onnxruntime-genai-cuda
```

- Import the models

```
import argparse
import os
import readline
import glob

import onnxruntime_genai as og
import time
```

- Here is the actual code to run the model

```
def _complete(text, state):
    return (glob.glob(text + "*") + [None])[state]


def run(args: argparse.Namespace):
    print("Loading model...")
    model = og.Model(args.model_path)
    processor = model.create_multimodal_processor()
    tokenizer_stream = processor.create_stream()

    while True:
        readline.set_completer_delims(" \t\n;")
        readline.parse_and_bind("tab: complete")
        readline.set_completer(_complete)
        image_paths = [
            image_path.strip()
            for image_path in input(
                "Image Path (comma separated; leave empty if no image): "
            ).split(",")
        ]
        image_paths = [image_path for image_path in image_paths if len(image_path)]
        print(image_paths)

        images = None
        prompt = "<|user|>\n"
        if len(image_paths) == 0:
            print("No image provided")
        else:
            print("Loading images...")
            for i, image_path in enumerate(image_paths):
                if not os.path.exists(image_path):
                    raise FileNotFoundError(f"Image file not found: {image_path}")
                prompt += f"<|image_{i+1}|>\n"

            images = og.Images.open(*image_paths)
            print("Images loaded")

        text = input("Prompt: ")
        prompt += f"{text}<|end|>\n<|assistant|>\n"
        print("Processing images and prompt...")
        inputs = processor(prompt, images=images)

        starttime = time.time()

        print("Generating response...")
        params = og.GeneratorParams(model)
        params.set_inputs(inputs)
        params.set_search_options(max_length=7680)

        generator = og.Generator(model, params)

        while not generator.is_done():
            generator.compute_logits()
            generator.generate_next_token()

            new_token = generator.get_next_tokens()[0]
            print(tokenizer_stream.decode(new_token), end="", flush=True)

        endtime = time.time()
        #print('Time taken: ', endtime - starttime)
        print(f"Time taken: {endtime - starttime:.2f} seconds")

        for _ in range(3):
            print()

        # Delete the generator to free the captured graph before creating another one
        del generator
```

- Run the model

```
# python phi3vgpu.py -m cuda-int4-rtn-block-32 
# C:\Code\gradioapps\mfgappsv1\images\DunnesStoresImmuneClosedCups450gLabel.jpg
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "-m", "--model_path", type=str, required=True, help="Path to the model"
    )
    args = parser.parse_args()
    run(args)
```

## run the model

```
python phi3vgpu.py -m cuda-int4-rtn-block-32
```

- use this path for image file as input

```
C:\Code\gradioapps\mfgappsv1\images\highvisibility_blog_1200x628_8-1024x536.jpg
```

- Prompt

```
can you find hardhats, safety vest, goggles
```

```
Generating response...
Yes, the image shows a person wearing a hard hat and safety vest, which are commonly used for construction work to protect against potential hazards. The person is also wearing safety goggles, which are used to protect the eyes from debris, dust, or other potential hazards. These items are essential for ensuring the safety and well-being of construction workers on the job site.Time taken: 207.39 seconds
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/phi/images/phi3-vision-1.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/phi/images/phi3-vision-2.jpg 'RagChat')