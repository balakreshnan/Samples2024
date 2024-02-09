# Comparison studies with 2 pdfs using Azure Open AI gpt 4 turbo

## Use Case

- Upload 2 pdfs
- ask question on that can compare information from one pdfs to another
- Using Azure Open AI gpt 4 turbo
- Idea here is to learn to compare 2 pdfs using Azure Open AI gpt 4 turbo
- for example compare quarterly reports of 2 quarters and show what to improve in next quarter

## Requirements

- Azure Subscription
- Azure Machine learning
- Azure Open AI gpt 4 turbo
- Python

## Code

- install libraries

```
%pip install openai
```

```
%pip install python-dotenv
```

```
%pip install PyPDF2
```

- Now load the environment variables

```
import os
import PyPDF2
```

```python
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- now configure open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2023-09-01-preview"
)
```

- now configure the pdfs

```
pdf_dir="./pdfs"
```

- Read the pdfs

```
text = []

for filename in os.listdir(pdf_dir):
    if filename.endswith('.pdf'):
        #pdf_file = open("./pdfs/" + filename, 'rb')
        print(' Filename: ' , filename)
        with open("./pdfs/" + filename, 'rb') as pdf_file:
            pdf_reader = PyPDF2.PdfReader(pdf_file)        
            for i in range(len(pdf_reader.pages)):
                page = pdf_reader.pages[i]
                text.append(page.extract_text())
                print(page)
```

- Combined text

```
combined_text = "\n".join(text)
```

- Formulate the message

```
message_text = [{"role":"system", "content":"Use the provided articles delimited by triple quotes to answer questions. If the answer cannot be found in the articles, write I could not find an answer."}, {"role": "user", "content": f""" {combined_text} \n Question: Compare Quarterly highlights across the two quarters and show us where to improve and put focus for next quarter? """}]
```

- now execute the code

```
response = client.chat.completions.create(
    model="gpt-4-turbo", # model = "deployment_name".
    messages=message_text
)

print(response.choices[0].message.content)
```

- output

```
Based on the information from the articles, let's compare the highlights across the two quarters of the Microsoft Fiscal Year 2024:

**First Quarter (Q1) Highlights (ended September 30, 2023):**
- Revenue was $56.5 billion, a 13% increase (12% in constant currency).
- Operating income was $26.9 billion, a 25% increase (24% in constant currency).
- Net income was $22.3 billion, a 27% increase (26% in constant currency).
- Diluted earnings per share was $2.99, a 27% increase (26% in constant currency).
- Microsoft Cloud revenue was $31.8 billion, up 24% (23% in constant currency) year-over-year.

**Second Quarter (Q2) Highlights (ended December 31, 2023):**
- Revenue was $62.0 billion, an 18% increase (16% in constant currency).
- Operating income was $27.0 billion, a 33% increase (25% non-GAAP, up 23% in constant currency).
- Net income was $21.9 billion, a 33% increase (26% non-GAAP, up 23% in constant currency).
- Diluted earnings per share was $2.93, a 33% increase (26% non-GAAP, up 23% in constant currency).
- Microsoft Cloud revenue was $33.7 billion, up 24% (22% in constant currency) year-over-year.
- The acquisition of Activision Blizzard, Inc. was completed on October 13, 2023.

**Areas to Improve and Focus for Next Quarter:**

1. **Devices Revenue**: In the second quarter, the Devices revenue decreased by 9% (down 10% in constant currency). This is an area that needs attention, and efforts should be made to understand the cause and develop strategies to improve device sales or consider phasing out underperforming products.

2. **Activision Integration**: With the recent acquisition of Activision Blizzard, it's crucial to integrate the business effectively to realize the expected synergies and to maximize the revenue growth from gaming content and services, which saw a considerable increase in Q2.

3. **Cloud Services**: Although the cloud services including Azure are doing well, continuous innovation, customer acquisition, and service improvements are necessary to maintain the growth trajectory and to compete with other cloud providers.

4. **Global Economic Uncertainties**: The broader economic conditions with respect to currency fluctuations and any economic downturn can impact spending on IT products and services. A focus on operational efficiency and cost optimization can help mitigate these factors.

5. **Operational Infrastructure**: There should always be a proactive focus on maintaining an adequate operational infrastructure to prevent outages and data losses which can harm the company's reputation and client trust.

6. **Artificial Intelligence (AI)**: As AI is an area of competitive differentiation for Microsoft, continuing to infuse AI across various products and services can create productivity gains for customers and maintain Microsoft's edge over competitors.

7. **ESG Initiatives**: Progressing on Environmental, Social, and Governance (ESG) initiatives can improve corporate reputation and potentially open up new markets and customer bases who prioritize sustainable and ethical business practices.

By addressing these areas, Microsoft could potentially improve its overall financial and operational performance in subsequent quarters.
```