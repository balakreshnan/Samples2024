# LLM Application RAG Architecture (RAG - Retrieval Augmented Generation)

## Simple way to build Generation AI Application

### 1. Introduction

- Build a Chat style generation ai application using LLM (Long Language Model) and RAG (Retrieval Augmented Generation) architecture.
- LLM is a language model that can generate text of arbitrary length.
- RAG is a retrieval augmented generation model that can generate text by referring to a document.
- Most effective and economical way to build a generation ai application.
- Please choose use cases that can be solved by generation ai application.
- Given LLM are language model, anything to generate text are the best target use cases
- Knowledge mining or chatting with lots of documents or information in text form are the best target use cases for RAG.
- To simplify i have split the application in 2 process
- 1st process is to generate embeddings for documents and create the LLM application and evaluate and test the LLM application.
- 2nd process is to deploy to web application framework for users to consume using chat UI.

```
Note: This is to show how to build a LLM application using RAG architecture. 
      we can use different services or products to achieve the same results.
```

## Architecture

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/Ragchat.jpg 'RagChat')

### 2. LLM Application

- First and foremost collect the documents for the use case.
- Research and and find the best way to chunk the document for example page, or chunk size with leading and trailing sentences.
- Use the Azure open ai ada model version 2 to create the embeddings for the documents.
- Once all the embeddings are created store them in a AI vector index or database.
- This is an iterative process and you can try different chunk size and leading and trailing sentences to get the best results.
- Now create the LLM application using prompt flow.
- Prompt flow will allows us to create step by step sequence to build LLM application
- Make sure try with different prompts and parameters to get the best results.
- Please have some questions to ask based on the documents.
- Also have expected answers created for the questions.
- Would be better if you can create a separate a dataset for the questions and answers to evaluate the LLM application.
- Depending on the use case, you might want to provide more context or other parameters for evaluation
- Evaluation is part of the development process and it is very important to get the best results.
- Can also evaluate with other LLM available in huggingdface model hub or other Open AI models.
- Create a evaluation score board to track the progress of the different LLM models.
- If you want to better results iterative with different chnking strategies and prompt engineering.
- Once we know if the LLM application is working as expected, we can move to the next step.
- You can deploy the model usually as web service or web application or API.
- Front end can be web Chat UI or Mobile UI.
- We can also download the LLM application and export it and then build LLM Ops using Azure DevOps or Github Actions.
- LLM Ops can be deployed using CLI so that we can use any Deployment tools.
- This is one way to deploy to different environments like Dev, Test, UAT and Prod.
- There is also single click deployment from UI to deploy to Azure App service as Web site.
- If you want to do enterprise deployment, either create the flow using code or export the code and create deployments.
- We can either deploy the entire end to end including development to testing and production.
- Or we can deploy the final application with documents to testing and production.
- Of course apply the best practicses like Security, Scale, Monitoring, Logging, Alerting, etc.
- Scale is very important for LLM application as it is very compute intensive.
- Scale your application based on workload requirements for the use case.
- Plan to monitor the application for future enhancements and improvements.
- Include more documents and improve prompts for better results.
- Get user feedback on the responses and save that in database for further analysis.
- Save user meta data information and other preference for providing better experience.
- You can measure the metrics like relevance, coherence, fluency, etc.

```
Note: Default model is general knowledge based, so will be challenging for industry specific use cases.
      We can build domain specific models or fine tune the model for better results.
```

- Fine tuning will be next step to improve the LLM application.
- We can use Azure Machine learning to fine tune open source models like LLama 2, GPT3, etc.
- Is an iterative process and also time consuming
- Data collection and data preparation is very important for fine tuning.
- Creating data set is where most time will be spent.
- Remaining process can be automated using Azure Machine learning.
- What to fine tune and how to will change based on use case.
- Do you want to change all the weights or only some weights.
- Depending on the above point, will cost more time and money.
- These models to require GPU compute, mostly A100 with 8 GPU minimum for the larger models.
- Yes we can also use Small language models to consume or evaluate the LLM application.