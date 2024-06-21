# Deploying LLaMa 3 chat and text in Azure ML and Azure AI Studio

## Introduction

- How to deploy LLaMa 3 model
- using GPU compute
- i am using SKU - Standard_NC24ads_A100_v4
- Can deploy only where GPU is available
- Using Azure Machine Learning
- Or you can also use Azure AI Studio
- Only to show how to deploy using the UI

## Pre-requisites

- Azure Machine learning or Azure AI Studio
- Azure subscription
- GPU compute instance
- Standard_NC24ads_A100_v4 cores available to deploy

## Steps

### Code

- Go to Azure machine learning and click on model catalog
- Select Meta on the right side catalog
- Select the LLaMa 3 model you want to deploy
- Click Deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/llama3chat1.jpg 'RagChat')

- Select the VM Size

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/llama3chat2.jpg 'RagChat')

- Wait for the model to deploy
- will take about 5 or 10 minutes

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/llama3chat3.jpg 'RagChat')

- Monitor section

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/llama3chat4.jpg 'RagChat')

- Logs

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/llama3chat5.jpg 'RagChat')