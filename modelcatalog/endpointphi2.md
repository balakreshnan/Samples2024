# Azure AI Studio Model catalog deploy Realtime Endpoint PHI - 2

## Introduction

- Realtime Endpoint PHI model
- Realtime from Model Catalog UI
- Using Azure AI Studio Model Catalog
- GPU compute needed
- SKU can change based on the model
- For PHI 2 model we need Standard_NC24s_v3 SKU
  

## Steps

- Go to AI Studio: https://ai.azure.com
- On the left menu select Model Catalog
- Search for Mixtral8x7b
- Click Deploy
- Select Realtime Endpoint
- Select GPU compute
- Select Standard_NC96s_v3 SKU. Only this SKU is supported for this model or bigger GPU SKU

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi2-1.jpg 'RagChat')

- list of VM's available is shown in the drop down list

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi2-2.jpg 'RagChat')

- Give a name to the enpoint
- Select the instance count based on quota available
- if you need more quota, then raise a support ticket
- Enable inference data collection to log metrics
- Click Deploy
- Wait for the deployment to complete, will take few minutes

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi2-3.jpg 'RagChat')

- here is the details page

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi2-4.jpg 'RagChat')

- Once deployed, click on the test tab
- Use this input

```
{
	  "input_data": {
	    "input_string": [
	      "Instruct: What is the recipes to make briyani?\nOutput:"
	    ],
	    "parameters": {
	      "top_p": 0.1,
	      "temperature": 0.1,
	      "max_new_tokens": 100,
	      "do_sample": true
	    }
	  }
}
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/phi2-5.jpg 'RagChat')

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

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/mixtral8x7b-6.jpg 'RagChat')

- For logs check the logs tab