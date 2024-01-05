# Auto fine tuning Multiple LLM and Pick the best one

## Introduction

- List LLM models
- List SLM models
- Ability to formulate the data set from external source
- We can also outsource data set creation to external vendors or crowd source
- Ability to run parallel model to fine tune
- ABility to select modes, and compute for the model
- Save the model output in metrics registry
- Have a algorithm to pick the best fine tune model based on coherence, similarity, groundess, relevancy, fluency
- We can also use other metrics as new ones are available

## Design

![info](https://github.com/balakreshnan/Samples2024/blob/main/LLMArch/images/autotuningdesign1.jpg 'RagChat')

- Pick the use case first
- Filter the models based on use case
- Work on the data set, depends on what type like instruct, chat or summarization etc
- Ability to select multiple LLM or SLM models
- Pick from list or use all type scenario
- Ability to select compute for the model
- Ability to see if we have quota for training
- Ability to predict how long will it take to fine tune
- Also fine tune model with evaluation and store the information
- Run through all the metrics output and find the best one
- Ability to run Responsible AI on the model