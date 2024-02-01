# Azure AI Studio Model catalog deploy Realtime Endpoint DeciLM7B instruct - 2

## Introduction

- Realtime Endpoint DeciLM7B model
- Realtime from Model Catalog UI
- Using Azure AI Studio Model Catalog
- GPU compute needed
- SKU can change based on the model
- For PHI 2 model we need Standard_NC12s_v3 SKU
  

## Steps

- Go to AI Studio: https://ai.azure.com
- On the left menu select Model Catalog
- Search for DeciLM7B model
- Click Deploy
- Select Realtime Endpoint
- Select GPU compute
- Select Standard_NC12s_v3 SKU. Only this SKU is supported for this model or bigger GPU SKU

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-1.jpg 'RagChat')

- list of VM's available is shown in the drop down list

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-2.jpg 'RagChat')

- Give a name to the enpoint
- Select the instance count based on quota available
- if you need more quota, then raise a support ticket
- Enable inference data collection to log metrics
- Click Deploy
- Wait for the deployment to complete, will take few minutes

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-3.jpg 'RagChat')

- here is the details page

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-4.jpg 'RagChat')

- Once deployed, click on the test tab
- Use this input

```
{
  "input_data": {
    "input_string": [
      "How do I make the most delicious briyani the world has ever tasted?"
    ],
    "parameters": {
      "top_p": 0.95,
      "temperature": 0.6,
      "max_new_tokens": 500,
      "do_sample": true
    }
  }
}
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-5.jpg 'RagChat')

- Takes about 20 or 30 seconds to get the output. Can vary based on question.
- Model output

```
[
  {
    "0": "How do I make the most delicious briyani the world has ever tasted?\n\nAnswer: To make the most delicious biryani the world has ever tasted, you need to follow these steps:\n\n1. Start with a high-quality rice, such as basmati rice. Wash the rice thoroughly under running water to remove any impurities.\n\n2. Marinate the meat (chicken, mutton, or any other meat of your choice) in yogurt, ginger-garlic paste, and spices like cumin, coriander, turmeric, garam masala, and chili powder. Leave it aside for at least an hour to marinate.\n\n3. In a separate pan, fry the onions until they turn golden brown. Add the marinated meat and cook until it's well-cooked.\n\n4. In a large pot, heat oil and fry the whole spices like cinnamon, cardamom, and cloves. Add the fried onions and the marinated meat. Cook until the meat is well-coated with the spices.\n\n5. In another pot, boil the rice until it's half-cooked. Drain the rice and mix it with the marinated meat. Add the saffron strands, ghee, and salt. Mix well and let it cool.\n\n6. In a large pot, layer the rice and meat mixture, followed by the fried onions and spices, and then the remaining rice mixture. Garnish with fried onions and saffron strands.\n\n7. Cover the pot with a lid and cook on low heat for about 10-15 minutes, or until the biryani is well-cooked and the rice is fluffy.\n\n8. Serve hot with raita, mint chutney, or any other chutney of your choice.\n\nBy following these steps, you can make the most delicious biryani the world has ever tasted. Enjoy!"
  }
]
```

- Have fun trying more prompts and see the output
- Once you are satisfied with the output, then you can use this endpoint in your application
- CLick Consume tab to see sample code to use in your application
- Now to see the latency click monitor

![info](https://github.com/balakreshnan/Samples2024/blob/main/modelcatalog/images/endpointdeciLM7b-6.jpg 'RagChat')

- For logs check the logs tab