# Deploying Phi 3 in Azure ML and Azure AI Studio

## Introduction

- How to deploy phi 3 model
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

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-3.jpg 'RagChat')

- Select the model you want to deploy
- I am using Phi-3-mini-128k-instruct model

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-4.jpg 'RagChat')

- Click Deploy
- Select Realtime endpoint
- First choose the SKU Standard_NC24ads_A100_v4
- Select the number of compute instance based on what environment

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-5.jpg 'RagChat')

- Make instance count to 1
- Then click deploy and wait for model to deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-1.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-2.jpg 'RagChat')

- Now click Test and try some sample questions

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-7.jpg 'RagChat')

- Check the monitoring and logs

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-8.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi3-9.jpg 'RagChat')