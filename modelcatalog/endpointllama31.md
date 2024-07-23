# Deploying LLaMa 3.1 8B in Azure ML and Azure AI Studio

## Introduction

- How to deploy pLLaMa 3.1 model
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

- Select the model you want to deploy
- I am using LLaMa 3.1 8B model

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-1.jpg 'RagChat')

- Provide a endpoint name

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-2.jpg 'RagChat')

- then click deploy and wait for model to deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-3.jpg 'RagChat')

- Check the progress

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-4.jpg 'RagChat')

- once completed check the details page of endpoint

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-5.jpg 'RagChat')

- Test the model to make sure it is working

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-6.jpg 'RagChat')

- For monitoring and logging click on the monitoring tab

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointllama31-7.jpg 'RagChat')

- Have fun