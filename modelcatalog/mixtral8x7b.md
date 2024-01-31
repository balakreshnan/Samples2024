# Realtime deployment of Mixtral8x7b in Azure AI Studio Model Catalog

## Introduction

- Deploy Model from Model catalog
- Using Open Source model
- Using Azure AI Studio Model Catalog
- Mixtral8x7b is a open source model
- Using Mixtral8x7b model from Model Catalog
- Deploy in GPU compute
- GPU SKU needed: Standard_NC6s_v3

## Steps

- Go to AI Studio: https://ai.azure.com
- On the left menu select Model Catalog
- Search for Mixtral8x7b
- Click Deploy
- Select Realtime Endpoint
- Select GPU compute
- Select Standard_NC96s_v3 SKU. Only this SKU is supported for this model or bigger GPU SKU

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-2.jpg 'RagChat')

- list of VM's available is shown in the drop down list

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-3.jpg 'RagChat')

- Give a name to the enpoint
- Select the instance count based on quota available
- if you need more quota, then raise a support ticket
- Enable inference data collection to log metrics
- Click Deploy
- Wait for the deployment to complete, will take few minutes

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-1.jpg 'RagChat')

- Once deployed, click on the test tab
- Use this input

```
{
    "input_data": {
        "input_string": [
            "Show me places to visit in London?",
            "Do you have Hyderabad Briyani recipes?"
        ],
        "parameters": {
            "max_new_tokens": 100,
            "do_sample": true,
            "return_full_text": false
        }
    }
}
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-4.jpg 'RagChat')

- Takes about 20 or 30 seconds to get the output. Can vary based on question.
- Model output

```
[
  {
    "0": "London Pass\n\nAlso on this page:\n\n## Create A London Tour Guide\n\nYou’re planning your own tour of London so create your own London Tour Guide.\n\nI’m going to give you a lot of really useful tips that will enable you to have the best possible time while in London.\n\nThis content will be free to you. I’m not going to hide that.\n\nThe payback for me is that I sell the best",
    "1": "Best we have is Bombay Briyani\n\n1 Like\n\nLong time!\n\nThere are lots of variants. Puri and rasam on the side is one you don’t get at restaurants (not that we go any more since the pandemic).\n\nI usually only make briyani lamb though.\n\nGreat to see your replies.\n\nWe have recently started making Hyderbadi Biryani and other regional specialties"
  }
]
```

- Have fun trying more prompts and see the output
- Once you are satisfied with the output, then you can use this endpoint in your application
- CLick Consume tab to see sample code to use in your application
- Now to see the latency click monitor

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-5.jpg 'RagChat')

- For logs check the logs tab