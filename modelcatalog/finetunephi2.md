# Azure Machine learning Model Catalog - Fine Tune PHI 2 model

## Introduction

- Fine tune PHI 2 model
- with custom dataset
- using Azure Machine learning model catalog
- GPU compute needed
- SKU can change based on the model
- For PHI 2 model we need Standard_NC24ads_A100_v4 (24 cores, 220 GB RAM, 64 GB disk) SKU
- If validation step fails please open the deigner and clone and delete the validation step and run again
- I am using 1 node for training.
- If you need more quota, then raise a support ticket

## Use Case

- Only to adjust the output of the model to your domain
- Using Microsoft PHI-2 - Small Lanugage Model
- Create a training and validation dataset
- Prepare a ground truth dataset

## Steps

- Go to Azure Machine learning
- Click Model Catalog
- Click FineTune

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-2.jpg 'RagChat')

- Now load the dataset
- Select Data

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-3.jpg 'RagChat')

- Upload the dataset or re use existing dataset
- i uploaded a dataset

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-4.jpg 'RagChat')

- Select column names for prompt and responses

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-5.jpg 'RagChat')

- Now create compute
- Select the compute cluster or use existing

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-6.jpg 'RagChat')

- Click Finish
- Now wait for the experiment to complete

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/finetunephi2-1.jpg 'RagChat')

- it took close to 8 hours with 1 GPU instance