# Hard Hats detection using owlv2 2 model

## Introduction

- Idea to use owlv2 2 model to detect objects
- Using the base model to detect hard hats
- Also see if we can segmentation on hard hats

## Pre-requisites

- Azure account
- Azure subscription
- Azure machine learning workspace
- Using CPU compute for inference
- Model size is close to 630MB

## Steps

- install the required packages

```
%pip install scipy
```

- Load libraries

```
import requests
from PIL import Image
import numpy as np
import torch
from transformers import AutoProcessor, Owlv2ForObjectDetection
from transformers.utils.constants import OPENAI_CLIP_MEAN, OPENAI_CLIP_STD
import requests
import copy
import torch
%matplotlib inline
```

- Check device to use:

```
import torch

# Check if a GPU is available and select the device
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# Display the device being used
print(f"Using device: {device}")

# If using GPU, print GPU details
if device.type == "cuda":
    print(f"GPU device name: {torch.cuda.get_device_name(0)}")
```

- Tried in GPU and CPU works in both.

- Download the model

```
processor = AutoProcessor.from_pretrained("google/owlv2-base-patch16-ensemble")
model = Owlv2ForObjectDetection.from_pretrained("google/owlv2-base-patch16-ensemble")
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/visionfoundation/images/owl2-1.jpg 'RagChat')

- Setup the image to read

```
url = "https://jksafety.com/cdn/shop/articles/ANSIISEA-107_2020_Standard_2_45d803bb-9e50-4f46-921e-3d1c8aae2972.jpg?v=1716934285&width=1000"
image = Image.open(requests.get(url, stream=True).raw)
texts = [["a photo of a hard hats", "a photo of a Safety vests"]]
inputs = processor(text=texts, images=image, return_tensors="pt")
```

- Set torch forward pass

```
# forward pass
with torch.no_grad():
    outputs = model(**inputs)
```

- Normalize the image

```
def get_preprocessed_image(pixel_values):
    pixel_values = pixel_values.squeeze().numpy()
    unnormalized_image = (pixel_values * np.array(OPENAI_CLIP_STD)[:, None, None]) + np.array(OPENAI_CLIP_MEAN)[:, None, None]
    unnormalized_image = (unnormalized_image * 255).astype(np.uint8)
    unnormalized_image = np.moveaxis(unnormalized_image, 0, -1)
    unnormalized_image = Image.fromarray(unnormalized_image)
    return unnormalized_image

unnormalized_image = get_preprocessed_image(inputs.pixel_values)
```

- Set the target sizes
- Process the model

```
target_sizes = torch.Tensor([unnormalized_image.size[::-1]])
# Convert outputs (bounding boxes and class logits) to final bounding boxes and scores
results = processor.post_process_object_detection(
    outputs=outputs, threshold=0.2, target_sizes=target_sizes
)
```

- Display bounding boxes

```
i = 0  # Retrieve predictions for the first image for the corresponding text queries
text = texts[i]
boxes, scores, labels = results[i]["boxes"], results[i]["scores"], results[i]["labels"]

for box, score, label in zip(boxes, scores, labels):
    box = [round(i, 2) for i in box.tolist()]
    print(f"Detected {text[label]} with confidence {round(score.item(), 3)} at location {box}")

```

- Load the library to display the image

```
import matplotlib.pyplot as plt
import matplotlib.patches as patches
from PIL import Image
```

- Display the image with bounding boxes

```
# Initialize a figure
fig, ax = plt.subplots(1)

# Display the image
ax.imshow(image)

# Loop through detected boxes, labels, and scores
for box, score, label in zip(boxes, scores, labels):
    # Convert the box to a format [x_min, y_min, width, height]
    box = [round(i, 2) for i in box.tolist()]
    x_min, y_min, x_max, y_max = box
    width, height = x_max - x_min, y_max - y_min

    # Create a Rectangle patch for the bounding box
    rect = patches.Rectangle((x_min, y_min), width, height, linewidth=2, edgecolor='r', facecolor='none')
    
    # Add the patch to the Axes
    ax.add_patch(rect)
    
    # Annotate with label and confidence score
    plt.text(x_min, y_min, f"{text[label]}: {round(score.item(), 3)}", color='white', 
             fontsize=10, bbox=dict(facecolor='red', alpha=0.5))

# Show the result
plt.show()
```

- Output

![info](https://github.com/balakreshnan/Samples2024/blob/main/visionfoundation/images/owl2-2.jpg 'RagChat')