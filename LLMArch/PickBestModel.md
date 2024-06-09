# Pick the Best model to use for use case

## Introduction

- Based on the use case, we can evaluate multiple LLM models to find the best model for our use case.
- Could be open source, closed source, or proprietary models.
- Only conuming the models, not training them.
- We can also use fine tuned models if available.

## Pre-requisites

- Azure Subscription
- Azure AI Studio
- Azure Open AI
- Model Catalog and deploy models
- Other models deployed as REST API to consume

## Architecture

- How do we choose which is the best LLM model we can use for our use case?
- We can evaluate multiple models and pick the best one.
- We need to have a ground truth data set for set of questions or chat messages.
- Once we have the dataset ready, we can evaluate the models against the dataset.
- Make sure the models we want to use based on which category, deploy them in Azure AI Studio as REST API.
- Every model we choose might have different hardware requirements, so we need to make sure we have the right hardware to deploy the model.
- Below is the flow diagram to pick the best model for our use case.

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/bestllmpickinferencing1.jpg 'RagChat')

## Steps

- First we need understand the use case and the data we have.
- Create a validation dataset with ground truth data.
- Each question will get an answer from the different models we choose.
- Then will be compared against the ground truth data.
- Metrics are calculated for each row inference.
- Based on the metrics, we can choose the best model for our use case.
- Make sure the parameters used like temperature, max_tokens, top_p, etc are same for all models.
- we can also change the parameters and evaluate the models again to see if the model is better with different parameters.
- Once we have all the models evaluated and metrics calculated, Store them to future reference.
- Identify the best model based on the metrics and deploy as REST API.
- Now we have the best model for our use case.