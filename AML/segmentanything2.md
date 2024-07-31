# Meta Segment Anything V2 in Azure Machine Learning

## Introduction

- Run SAM v2 in Azure Machine Learning
- how can we install the model and run in compute instance
- Using GPU machine

## Requirements

- Azure Subscription
- Azure Machine Learning Workspace
- Azure Machine Learning Compute Instance
- Standard_NC96ads_A100_v4 (96 cores, 880 GB RAM, 256 GB disk)
- Create a compute instance with the above configuration
- Create a virtual environment with python 3.11

## Steps

- Create virtual environment for python 3.11

```
conda create --name py311
conda activate py311
conda install python==3.11
conda install ipykernel
python -m ipykernel install --user --name py311 --display-name "Python (py311)"
```

- Create a compute instance with the above configuration
- Start the instance
- Go to Notebook and open terminal
- Set the right compute instance in the top right corner
- i am using py311 which is python 3.11
- with GPU compute instance
- Lets go to meta segmantation anything v2
- fork the repo and clone it from: https://github.com/facebookresearch/segment-anything-2
- Here is mine: https://github.com/balakreshnan/segment-anything-2
- In case if i need to make changes i can change in my fork instead the original repo
- now go to terminal and install the required packages
- Go to terminal
- type conda env list

```
conda env list
```

- then type conda activate py311

```
conda activate py311
```

- py311 is the virtual environment name i used
- Now clone the repo

```
git clone https://github.com/balakreshnan/segment-anything-2.git
```

- Go to the folder

```
cd segment-anything-2
```

- now install the library

```
pip install -e .
```

- now install the demo

```
pip install -e ".[demo]"
```

- Now download the model

```
cd checkpoints
./download_ckpts.sh
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/samv2-1.jpg 'RagChat')

- now we can run the demo in notebooks folder

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/samv2-2.jpg 'RagChat')

- Keep executing each cell and see the output
- i ran all the cells and here are some samples at the end of the notebook

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/samv2-3.jpg 'RagChat')
![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/samv2-4.jpg 'RagChat')

- This is how we can run SAM v2 in Azure Machine Learning