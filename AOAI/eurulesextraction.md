# Extract rules from Government regulations

## Introduction

- using REGULATION OF THE EUROPEAN PARLIAMENT AND OF THE COUNCIL
- Extract rules from Government regulations
- Idea here is extract data privacy rules from the regulations
- The rules are extracted from the regulations and further processed to validate rules
- Take the rules and then validate against the question asked
- Content: https://eur-lex.europa.eu/resource.html?uri=cellar:6d8cdc8b-63f7-11e8-ab9c-01aa75ed71a1.0001.02/DOC_1&format=PDF

## Pre-requisites

- Azure Subscription
- Azure machine learning service
- Azure Open AI Service
- Python 3.10 or greater
- GPT 4 turbo 128K Token model

## Steps/Code

- Load environment variables

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Load open ai libraries and environment variables

```
import openai 

openai.api_type = "azure"
openai.api_key = config["AZURE_OPENAI_KEY"]
openai.api_base = config["AZURE_OPENAI_ENDPOINT"]
openai.api_version = config["AZURE_OPENAI_ENDPOINT"]
openai.base_url = config["AZURE_OPENAI_ENDPOINT"]
```

- Setup the open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = config["AZURE_OPENAI_ENDPOINT"], 
  api_key=config["AZURE_OPENAI_KEY"],  
  api_version="2024-02-01"
)
```

- load pypdf libraries

```
import os
import PyPDF2
```

- Setup file name and text variable to store content

```
filename = "cellar_6d8cdc8b-63f7-11e8-ab9c-01aa75ed71a1.0001.02_DOC_1.pdf"
text = []
```

- Open the pdf file and read the content

```
with open("./" + filename, 'rb') as pdf_file:
    pdf_reader = PyPDF2.PdfReader(pdf_file)        
    for i in range(len(pdf_reader.pages)):
        page = pdf_reader.pages[i]
        text.append(page.extract_text())
        #print(page)
```

- Now combined all the text into a single string

```
combined_text = "\n".join(text)
```

- now setup the rules to extract from the text

```
message_text = [{"role":"system", "content":"Use the provided articles delimited by triple quotes to answer questions. Only respond from the sources content provided."}, {"role": "user", "content": f""" {combined_text} \n Rules: Extract all the rules provided in the document. """}]
```

- now create the open ai request

```
response = client.chat.completions.create(
    model="gpt-4-turbo", # model = "deployment_name".
    messages=message_text
)

print(response.choices[0].message.content)
```

- The output will be the rules extracted from the document

```
Regulation (EU) No 1380/2013:

- The objectives of the common fisheries policy (CFP) and the requirements for fisheries control and enforcement are set out in Articles 2 and 36 of that Regulation.
- The successful implementation of the CFP depends on the effective and up-to-date control and enforcement system.

Council Regulation (EC) No 1224/2009:

- Provides for monitoring centers, tracking of fishing vessels, reporting obligations, and other controls to ensure compliance with the CFP.
- Defines specific terms like 'rules of the common fisheries policy,' 'fishing licence,' 'vessel position data,' 'lot,' and 'recreational fisheries.'
- Requires Member States to track all fishing vessels, including those less than 12 meters in length.
- Requires electronic reporting and recording of logbooks, transhipment declarations, and landing declarations.
- Introduces a mandatory inspection report system.
- Establishes a point system for serious infringements and sanctions for non-compliance.

Council Regulation (EC) No 768/2005:

- Establishes a European Fisheries Control Agency (EFCA) to ensure a high, uniform, and effective level of control and compliance with the CFP.

Regulation (EC) No 1005/2008:

- Establishes a system to prevent, deter and eliminate illegal, unreported and unregulated (IUU) fishing.
- Requires the use of catch certificates for imported fishery products.

Regulation (EU) No 2017/2403:

- Provides for the management of external fishing fleets.

Regulation (EU) 2016/679 (General Data Protection Regulation - GDPR):

- Applies to processing of personal data by the Union institutions.

Regulation (EU) No 182/2011:

- Establishes rules for control by Member States of the Commission's exercise of implementing powers.

Council Regulation (EC) No 1967/2006:

- Concerns management measures for sustainable exploitation of fishery resources in the Mediterranean Sea.

Regulation (EU) No 2016/1139:

- Establishes a multiannual plan for stocks of cod, herring, and sprat in the Baltic Sea.

Regulation (EU) 2017/1004:

- Establishes a Union framework for the collection, management, and use of data in the fisheries sector.

Regulation (EU) No 1379/2013:

- Establishes the common organization of the markets in fishery and aquaculture products.

Regulation (EU) No 1379/2013 and Regulation (EU) No 1224/2009:

- Set out provisions related to control and enforcement of the CFP, official controls, traceability, and market standards for fishery and aquaculture products.

The document provides amendments to align the enforcement of CFP rules with newer policies, improve data reliability, simplify the legal framework, and introduce modern technologies for better control and compliance. It also details participatory roles for various groups, personal data protection rules, delegation and implementation of powers, and budgetary implications without changing the maximum funding allocated in operational programs for the period 2014-2020. Additionally, there are implementation plans for monitoring, evaluation, and reporting, along with an overview of specific provisions proposed for regulation amendments.
```

- Now lets save the above content as variable and then validate against the question asked

```
rulescnt = response.choices[0].message.content
```

- Now setup the question to validate against the rules

```
message_text1 = [{"role":"system", "content":"Rules are provided and your job to validate and see if the question asked is within rules of engagement provided. Only respond from the sources content provided."}, {"role": "user", "content": f""" {rulescnt} \n Does Fishing for salmon in baltic sea is legal? """}]
```

- now create the open ai request

```
response1 = client.chat.completions.create(
    model="gpt-4-turbo", # model = "deployment_name".
    messages=message_text1
)

print(response1.choices[0].message.content)
```

- The output will be the validation of the question asked against the rules

```
Based on the source content provided, Regulation (EU) No 2016/1139 establishes a multiannual plan for the management of stocks of cod, herring, and sprat in the Baltic Sea. This indicates that there is a framework regulating some types of fishing in the Baltic Sea. However, the specific legality of fishing for salmon in the Baltic Sea is not directly addressed in the provided excerpts.

To determine the current legality of fishing for salmon in the Baltic Sea, one would need to refer to the specific measures within that regulation and any other relevant local or EU laws, such as those concerning species conservation statuses, quotas, seasonal restrictions, or protected areas. Additionally, national regulations of the Baltic Sea bordering countries would also play a role.

As the provided sources do not directly answer the question about the legality of fishing for salmon in the Baltic Sea, further research on current, specific regulations regarding salmon fishing in that area would be necessary.
```

- Sole purpose is to show how to extract rules from the regulations and then validate against the question asked
- This doesn't guarantee the accuracy of the rules extracted
- Work with Prompt engineering to extract the rules accurately