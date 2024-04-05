# Fine tuning Decision Tree and Process

## Introduction

- Why should we do fine tuning?
- What is the process of fine tuning?
- How would i take a decision to fine tune?
- Is there a value in fine tuning for business use case?
- What are the steps involved in fine tuning?
- Only high level steps are discussed here.
- These are just my thoughts and can change based on organizations requirements
- Use this as guidance and not as a rule book

## Fine tuning Decision Making and Process

### High Level Process and decision making

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/finetuneprocess1.jpg 'RagChat')

- First figure out if fine tuning is needed.
- If we can use RAG and achieve the outcome then no need for fine tuning.
- Only use fine tuning if you want to add more your own context to the model.
- Fine tuning can be used to respond with organization data and context.
- Fine tuning is a cumbersome process and needs lot of data and time.
- Collecting data and labeling data is a time consuming process.
- Human in loop is needed to validate the results.
- Human also have to avoid bias in the data.
- Training infrastructure like GPU compute is hard to get and expensive
- Optimized fine tuning for scale will be another challenge.
- Data collection and creation can be automated but needs lots of validation by human.
- Fine tuning is done based on use case and task inside.
- We can also use small and large lanugage models which has multiple tasks inbuilt to fine tune.
- Use data cache to save training and validation, test data sets.
- Make sure there is human evaluation in the loop to data set creation results before it goes to training.
- For fine tuning select the model based on tasks and how effective to use.
- Try with small language model first and then go to large language model.
- After fine tuning, evaluate the model with test data set.
- Also evaluate the model for responsible ai and bias.
- For training there are multiple way to test LoRa,QloRa, DoRa.
- Also pick use GPU Nvidia libraries like NCCL to speed up training.
- This is speed up Pytorch, Tensorflow training.
- Once the results are good, create a leaderboard and update the information.
- If results are acceptable, use LLMOps to deploy to environment.
- Save the model to registry to use in production.
- Saving in registry also allows to share with other in the same organization.
- The model consumers might be user or applications built on top of it.
- It is absolutly necessary to manage the fine tuning life cycle management.
- Security and privacy is also important in the fine tuning process.
- Be transparent in the process and results.

### Deeper look at teams and their skills needed

![info](https://github.com/balakreshnan/Samples2024/blob/main/finetuning/images/finetuneprocess2.jpg 'RagChat')

- Here is my view of what is needed in the team.
- Might be missing few also.
- i have defined the functionality
- Skills needed can be defined based on how the organization is structured.
- The team should have a good mix of skills.
- The team should have a good mix of experience.
- The team should have a good mix of domain knowledge.
- Domain experience can be subject management expertise or industry expertise as needed.
- Need for testing which is evaluting model and it's relevance is important.