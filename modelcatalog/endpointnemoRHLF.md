# Azure AI Model Catalog - Endpoint Nemo RHLF

## Introduction

- Deploy an Nemo endpoint in Azure AI Studio
- Using GPU SKU - Standard_NC24ads_A100_v4
- Using Managed Endpoint

## Use Case

- To deploy an Nemo model
- Only for inferencing purpose
- Ability to deploy rest and interface with other python apps
- Using Model ID: Nemotron-3-8B-Chat-RLHF

## Steps

- Go to Azure AI Studio
- Location is WEST US3

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointnemoRHLF-1.jpg 'RagChat')

- List of computes
- Select the compute
- Click Deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointnemoRHLF-2.jpg 'RagChat')

- After deployment here is the details

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointnemoRHLF-3.jpg 'RagChat')

- Now you can test the endpoint
- https://learn.microsoft.com/en-us/azure/machine-learning/how-to-deploy-with-triton?view=azureml-api-2&amp%3Btabs=azure-cli%2Cendpoint&tabs=azure-cli%2Cendpoint#test-the-endpoint

- Check the monitor section

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointnemoRHLF-4.jpg 'RagChat')