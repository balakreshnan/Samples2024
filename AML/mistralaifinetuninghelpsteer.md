# Azure Machine Learning fine tuning Mistralai model with Helpsteer data

## Requirements

- Azure Machine Learning workspace
- Azure Machine Learning compute cluster
- Download helpsteerTM dataset from: https://huggingface.co/datasets/nvidia/HelpSteer
- Make sure you have GPU enabled compute cluster with 8 A100 GPU machines

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/mistralai4.jpg, 'Mistralai')

## Steps

- Login to Azure Machine Learning studio
- Create a new compute cluster
- Select GPU 8 A100 machines and select the number of nodes needed
- can range from 0 to 2 or 10 depending on how much quota you have
- Go to Model Catalog
- Select mistralai-Mistral-7B-v01
- Click fine tune

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/mistralai1.jpg 'Mistralai')

- Select the training dataset
- First upload the helpsteerTM dataset in upload screen

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/mistralai1-1.jpg 'Mistralai')

- Click Next

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/mistralai2.jpg 'Mistralai')

- Now do the same for validation dataset

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/mistralai3.jpg 'Mistralai')

- Now click Submit and wait for the training to complete
- Can take hours to complete