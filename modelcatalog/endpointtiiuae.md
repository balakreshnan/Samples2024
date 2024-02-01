# Azure AI Studio Model catalog deploy Realtime Endpoint PHI - 2

## Introduction

- Realtime Endpoint tiiuae model
- Realtime from Model Catalog UI
- Using Azure AI Studio Model Catalog
- GPU compute needed
- SKU can change based on the model
- For PHI 2 model we need Standard_NC48ads_A100_v4 SKU
  

## Steps

- Go to AI Studio: https://ai.azure.com
- On the left menu select Model Catalog
- Search for tiiuae model
- Click Deploy
- Select Realtime Endpoint
- Select GPU compute
- Select Standard_NC48ads_A100_v4 SKU. Only this SKU is supported for this model or bigger GPU SKU

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-1.jpg 'RagChat')

- list of VM's available is shown in the drop down list

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-2.jpg 'RagChat')

- Give a name to the enpoint
- Select the instance count based on quota available
- if you need more quota, then raise a support ticket
- Enable inference data collection to log metrics
- Click Deploy
- Wait for the deployment to complete, will take few minutes

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-3.jpg 'RagChat')

- here is the details page

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-4.jpg 'RagChat')

- Once deployed, click on the test tab
- Use this input

```
{
  "inputs": "Create 3 ideas on what to do in Germany, Nuremberg. I want to go there in the summer.",
  "parameters": {
    "do_sample": true,
    "top_p": 0.95,
    "temperature": 0.2,
    "top_k": 50,
    "max_new_tokens": 256,
    "repetition_penalty": 1.03,
    "stop": ["
User:", "<|endoftext|>", "</s>"]
  }
}
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-5.jpg 'RagChat')

- Takes about 20 or 30 seconds to get the output. Can vary based on question.
- Model output

```
[
  {
    "0": "Instruct: What is the recipes to make briyani?\nOutput: Ingredients:\n- 1 cup basmati rice\n- 1 cup water\n- 1/2 cup ghee\n- 1 onion, finely chopped\n- 2 cloves garlic, minced\n- 1 tablespoon ginger, grated\n- 1 teaspoon cumin seeds\n- 1 teaspoon cinnamon\n- 1/2 teaspoon turmeric powder\n- 1/2 teaspoon cardamom powder\n- 1/2 teaspoon cloves\n- 1/2 teaspoon coriander powder\n- 1/2 teaspoon gar"
  }
]
```

- Have fun trying more prompts and see the output
- Once you are satisfied with the output, then you can use this endpoint in your application
- CLick Consume tab to see sample code to use in your application
- Now to see the latency click monitor

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointtiiuae-4.jpg 'RagChat')

- For logs check the logs tab