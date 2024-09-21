# Hard Hats detection using Florence 3 model

## Introduction

- Idea to use florence 2 model to detect objects
- Using the base model to detect hard hats
- Also see if we can segmentation on hard hats

## Pre-requisites

- Azure account
- Azure subscription
- Azure machine learning workspace
- Using GPU compute for inference
- Model size is close to 1.5GB

## Steps

- install the required packages

```
from transformers import AutoProcessor, AutoModelForCausalLM
from PIL import Image
import requests
import copy
import torch
%matplotlib inline
```

- Import the models

```
model_id = 'microsoft/Florence-2-large'
model = AutoModelForCausalLM.from_pretrained(model_id, trust_remote_code=True, torch_dtype='auto').eval().cuda()
processor = AutoProcessor.from_pretrained(model_id, trust_remote_code=True)
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/visionfoundation/images/florence2-1.jpg 'RagChat')

- function to process the model for answers

```
def run_example(task_prompt, text_input=None):
    if text_input is None:
        prompt = task_prompt
    else:
        prompt = task_prompt + text_input
    inputs = processor(text=prompt, images=image, return_tensors="pt").to('cuda', torch.float16)
    generated_ids = model.generate(
      input_ids=inputs["input_ids"].cuda(),
      pixel_values=inputs["pixel_values"].cuda(),
      max_new_tokens=1024,
      early_stopping=False,
      do_sample=False,
      num_beams=3,
    )
    generated_text = processor.batch_decode(generated_ids, skip_special_tokens=False)[0]
    parsed_answer = processor.post_process_generation(
        generated_text,
        task=task_prompt,
        image_size=(image.width, image.height)
    )

    return parsed_answer
```

- Load the image

```
url = "https://static.vecteezy.com/system/resources/thumbnails/006/268/897/small_2x/safety-uniform-workers-and-industrial-engineers-with-hardhat-use-laptop-computer-to-check-and-control-machines-three-professionals-work-in-paper-manufacturing-factory-maintain-production-equipment-photo.jpg"
image = Image.open(requests.get(url, stream=True).raw)
```

```
image
```

- i am using public images
- Lets now to caption

```
task_prompt = '<CAPTION>'
run_example(task_prompt)
```

- Detailed caption

```
task_prompt = '<DETAILED_CAPTION>'
run_example(task_prompt)
```

- now object detection

```
task_prompt = '<OD>'
results = run_example(task_prompt)
print(results)
```

- object detection plotting for output

```
import matplotlib.pyplot as plt
import matplotlib.patches as patches
def plot_bbox(image, data):
   # Create a figure and axes
    fig, ax = plt.subplots()

    # Display the image
    ax.imshow(image)

    # Plot each bounding box
    for bbox, label in zip(data['bboxes'], data['labels']):
        # Unpack the bounding box coordinates
        x1, y1, x2, y2 = bbox
        # Create a Rectangle patch
        rect = patches.Rectangle((x1, y1), x2-x1, y2-y1, linewidth=1, edgecolor='r', facecolor='none')
        # Add the rectangle to the Axes
        ax.add_patch(rect)
        # Annotate the label
        plt.text(x1, y1, label, color='white', fontsize=8, bbox=dict(facecolor='red', alpha=0.5))

    # Remove the axis ticks and labels
    ax.axis('off')

    # Show the plot
    plt.show()
```

- disply the image

```
plot_bbox(image, results['<OD>'])
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/visionfoundation/images/florence2-2.jpg 'RagChat')

- now doing segmentation

```
task_prompt = '<REFERRING_EXPRESSION_SEGMENTATION>'
results = run_example(task_prompt, text_input="hard hats")
print(results)
```

- Prep the image to show segmentation

```
from PIL import Image, ImageDraw, ImageFont
import random
import numpy as np
colormap = ['blue','orange','green','purple','brown','pink','gray','olive','cyan','red',
            'lime','indigo','violet','aqua','magenta','coral','gold','tan','skyblue']
def draw_polygons(image, prediction, fill_mask=False):
    """
    Draws segmentation masks with polygons on an image.

    Parameters:
    - image_path: Path to the image file.
    - prediction: Dictionary containing 'polygons' and 'labels' keys.
                  'polygons' is a list of lists, each containing vertices of a polygon.
                  'labels' is a list of labels corresponding to each polygon.
    - fill_mask: Boolean indicating whether to fill the polygons with color.
    """
    # Load the image

    draw = ImageDraw.Draw(image)


    # Set up scale factor if needed (use 1 if not scaling)
    scale = 1

    # Iterate over polygons and labels
    for polygons, label in zip(prediction['polygons'], prediction['labels']):
        color = random.choice(colormap)
        fill_color = random.choice(colormap) if fill_mask else None

        for _polygon in polygons:
            _polygon = np.array(_polygon).reshape(-1, 2)
            if len(_polygon) < 3:
                print('Invalid polygon:', _polygon)
                continue

            _polygon = (_polygon * scale).reshape(-1).tolist()

            # Draw the polygon
            if fill_mask:
                draw.polygon(_polygon, outline=color, fill=fill_color)
            else:
                draw.polygon(_polygon, outline=color)

            # Draw the label text
            draw.text((_polygon[0] + 8, _polygon[1] + 2), label, fill=color)

    # Save or display the image
    #image.show()  # Display the image
    display(image)
```

- now display the output

```
output_image = copy.deepcopy(image)
draw_polygons(output_image, results['<REFERRING_EXPRESSION_SEGMENTATION>'], fill_mask=True)
```

- output

![info](https://github.com/balakreshnan/Samples2024/blob/main/visionfoundation/images/florence2-3.jpg 'RagChat')