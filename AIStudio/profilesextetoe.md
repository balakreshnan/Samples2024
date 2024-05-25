# End to End Gen AI Application using Azure AI Studio

## Introduction

- build a end to end Gen ai application using azure ai studio
- Assuming we already have a vector index with dataset
- Using existing vector index
- Sample data pulled from LinkedIn profiles
- This is just an example not real data

## Pre-requisites

- Azure Subscription
- Azure AI Studio
- Azure Open AI GPT 4o in East us or west us 3 region
- Azure Open AI GPT 4o key
- Azure AI Search
- Existing vector search
- Sample data loaded in AI Search

## Steps

### Create a connection to existing index

- Go to Settings
- Click Add connections
- Select the vector AI Search to add connection
- Now we create a connection to existing index
- Click Create Index

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector1.jpg 'RagChat')

- Select AI Search

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector2.jpg 'RagChat')

- Select the existing AI search connection
- Select the existing Index created before

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector3.jpg 'RagChat')

- Add Vector capability if needed as optional. Only if vector is not created before

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector4.jpg 'RagChat')

- Create the same index Name

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector5.jpg 'RagChat')

- Now map the fields to the existing index (only optional if needed)

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector6.jpg 'RagChat')

- Then click create
- We are done with creating the connection to existing index
- Next is to create the AI application using prompt flow


### Create the AI application

- Go to Prompt Flow
- Click Create
- Clone multi turn chat with your own data template
- Template has best flow to get the optimal results

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector7.jpg 'RagChat')

- give a name
- Now start the compute to use for prompt flow

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector8.jpg 'RagChat')

- Wait for the compute to start
- Then scroll down to see the prompt flow
- Let's go from top to bottom
- Each box is a unit of execution
- Let's config Azure open ai connection to get semantic search query
- We are using GPT 4o model

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector9.jpg 'RagChat')

- Scroll down to lookup in the flow
- Select the existing vector index we created above
- Map the fields to the existing index

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector10.jpg 'RagChat')

- after all the setup lookup should like this

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector11.jpg 'RagChat')

- Go down to the last prompt flow prompt execution
- Choose the open ai model to use
- We are using GPT 4o

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector12.jpg 'RagChat')

- Now Click Save in the right top of the screen
- Now click Chat to test the AI application
- Type in the question and see the response
- in execution here is the screen

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector13.jpg 'RagChat')

- once execution is done you can see the response

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector14.jpg 'RagChat')

- Click View trace to see graphical execution and time took for each graph or execution block

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector15.jpg 'RagChat')

- Now on the right top there is a evaluate button

### Evaluate the AI application

- Click Evaluate
- Create a new evaluation

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector16.jpg 'RagChat')

- Select question answer with context
- Select a existing dataset
- I manually created a JSONL file for chat data set
- If you don't have a dataset please creaet one based on your use case
- Image will show the column names in the dataset

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector17.jpg 'RagChat')

- Now Select the evaluation metrics

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector18.jpg 'RagChat')

- provide the azure open ai deployment to use

- Now select the Safety metrics

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector19.jpg 'RagChat')

- select Medium for evaluation
- Change based on your use case
- Also see the data set columns to map to the evaluation
- Click Evaluate
- Wait for the evaluation to complete
- Once done you can see the evaluation results
- You can see the evaluation results in the screen

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector28.jpg 'RagChat')

- Check Performance and Quality metrics

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector20.jpg 'RagChat')

- Check the Risk and Safety Metrics

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector21.jpg 'RagChat')

- Now the above process goes in iteration until you get the optimal results
- Once done you can deploy the AI application

### Deploy the AI application

- Go back to prompt flow and select the one created
- On Top menu bar there is a option for Deploy
- Click Deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector22.jpg 'RagChat')

- Give a Name and select the compute to deploy
- usually 3 nodes for production deployment
- Select your security method to authenticate

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector23.jpg 'RagChat')

- Create tags as needed

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector24.jpg 'RagChat')

- Now select the Azure open ai deployment to use
- i am using GPT 4o global deployment
- We can use regional deployment as well

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector25.jpg 'RagChat')

- Review the deployment and click Deploy

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector26.jpg 'RagChat')

- Wait for the REST API to complete

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector27.jpg 'RagChat')

- Wait for the deployment to complete

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector29.jpg 'RagChat')

- once done you can see the deployment details

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector30.jpg 'RagChat')

- Test the deployment using the REST API

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector31.jpg 'RagChat')

- Find the monitor and logs

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector32.jpg 'RagChat')

![info](https://github.com/balakreshnan/Samples2024/blob/main/AIStudio/images/aivector33.jpg 'RagChat')

### Conclusion

- We have successfully created a end to end Gen AI application using Azure AI Studio
- We did evaluation and deployment
- Able to test in development
- Also able to test the production deployment