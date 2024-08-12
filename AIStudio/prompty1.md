# Prompty Example 1 for Gen AI Application

## aisearch code

```
from typing import List
import os
from azure.identity import DefaultAzureCredential
from azure.search.documents import SearchClient
from azure.search.documents.models import (
    VectorizedQuery,
    QueryType,
    QueryCaptionType,
    QueryAnswerType,
)
from azure.core.credentials import AzureKeyCredential

from dotenv import dotenv_values

config = dotenv_values("env.env")

azuresearchendpoint = config["AZURE_SEARCH_ENDPOINT"]
azuresearchkey = config["AZURE_SEARCH_KEY"]

def retrieve_documentation(
    question: str,
    index_name: str,
    embedding: List[float],
) -> str:

    
    search_client = SearchClient(
        endpoint=azuresearchendpoint,
        index_name=index_name,
        credential=AzureKeyCredential(azuresearchkey),
    )

    vector_query = VectorizedQuery(
        vector=embedding, k_nearest_neighbors=3, fields="contentVector"
    )

    results = search_client.search(
        search_text=question,
        vector_queries=[vector_query],
        query_type=QueryType.SEMANTIC,
        semantic_configuration_name="default",
        query_caption=QueryCaptionType.EXTRACTIVE,
        query_answer=QueryAnswerType.EXTRACTIVE,
        top=3,
    )

    docs = [
        {
            "id": doc["id"],
            "title": doc["title"],
            "content": doc["content"],
            "url": doc["url"],
        }
        for doc in results
    ]

    return docs
```

- AI Search function above is created as new aisearch.py file
- It's needed to retrieve context data to ground the model
- Setting Environment variables in the env.env file
- Setting Environment variables in the windows machine is been challenging to recognize the environment variables

## Code

```
from dotenv import load_dotenv
#load_dotenv()
from dotenv import dotenv_values

config = dotenv_values("env.env")

#config = load_dotenv("env.env")
azure_deployment = config["AZURE_OPENAI_EMBEDDING_DEPLOYMENT"]
api_key = config["AZURE_OPENAI_API_KEY"]
api_version = config["AZURE_OPENAI_API_VERSION"]
api_base = config["AZURE_OPENAI_ENDPOINT"]
embeddingmodel = config["AZURE_OPENAI_EMBEDDING_MODEL"]
azuresearchendpoint = config["AZURE_SEARCH_ENDPOINT"]
azure_chatdeployment = config["AZURE_OPENAI_CHAT_DEPLOYMENT"]
azure_endpoint = config["AZURE_OPENAI_ENDPOINT"]

from sys import argv
import os
import pathlib
from ai_search import retrieve_documentation
from promptflow.tools.common import init_azure_openai_client
from promptflow.connections import AzureOpenAIConnection
from promptflow.core import (AzureOpenAIModelConfiguration, Prompty, tool)
from promptflow.tracing import start_trace
start_trace()

def get_context(question, embedding):
    return retrieve_documentation(question=question, index_name="aistudioprofile", embedding=embedding)

def get_embedding(question: str):
    #config = load_dotenv("env.env")
    
    #connection = AzureOpenAIConnection(        
    #                azure_deployment=os.getenv("AZURE_OPENAI_EMBEDDING_DEPLOYMENT", ""),
    #                api_version=os.getenv("AZURE_OPENAI_API_VERSION", ""),
    #                api_base=os.getenv("AZURE_OPENAI_ENDPOINT", "")
    #                )
    connection = AzureOpenAIConnection(        
                    #azure_deployment=azure_deployment,
                    api_key=api_key,
                    #api_version=api_version,
                    api_base=api_base,
                    #endpoint=api_base
                    )
                
    client = init_azure_openai_client(connection)

    return client.embeddings.create(
            input=question,
            model=embeddingmodel,
        ).data[0].embedding

def get_response(question, chat_history):
    print("inputs:", question)
    embedding = get_embedding(question)
    context = get_context(question, embedding)
    #print("context:", context)
    print("getting result...")
    rs = []

    # Load prompty with AzureOpenAIModelConfiguration override
    configuration = AzureOpenAIModelConfiguration(
        azure_deployment="gpt-4o-g",
        api_key=api_key,
        api_version=api_version,
        azure_endpoint=azure_endpoint,
        #connection=AzureOpenAIConnection(api_base=api_base, api_key=api_key, api_type="azure_openai"),
    )
    override_model = {
        "configuration": configuration,
        "parameters": {"max_tokens": 512}
    }
    #prompty = Prompty.load(source="path/to/prompty.prompty", model=override_model)
    
    #data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "./chat.prompty")
    #print(data_path)
    # Override model config with dict
    model_config = {
        "api": "chat",
        "configuration": {
            "type": "azure_openai",
            "azure_deployment": "gpt-4o-g",
            "api_key": f"{api_key}",
            "api_version": f"{api_version}",
            "azure_endpoint": f"{azure_endpoint}",
        },
        "parameters": {
            "max_tokens": 512
        }
    }

    #print(model_config)

    data_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "chat1.prompty")
    #print(data_path)

    #prompty_obj = Prompty.load(data_path, model=override_model)
    #prompty_obj  = Prompty.load(source="chat1.prompty", model=model_config)
    prompty_obj  = Prompty.load(source="chat.prompty", model=model_config)

    result = prompty_obj(question = question, documents = context)
    #result = prompty_obj(first_name="John", last_name="Doh", question="What is the capital of France?")

    print("result: ", result)

    # Initialize Azure OpenAI Connection with your environment variables
    eval_model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=azure_endpoint,
        api_key=api_key,
        azure_deployment=azure_chatdeployment,
        api_version=api_version,
    )

    relevance_evaluator = RelevanceEvaluator(model_config=eval_model_config)
    coherence_evaluator = CoherenceEvaluator(model_config=eval_model_config)
    #path = "profiles.jsonl"

    evalutionname = "chat_eval_gpt4og_" + str(uuid.uuid4())[:8]
    print(evalutionname)
    json_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "profiles.jsonl")
    print(json_path)

    # Initialzing Relevance Evaluator
    relevance_eval = RelevanceEvaluator(eval_model_config)
    # Running Relevance Evaluator on single input row
    relevance_score = relevance_eval(
        answer="The Alpine Explorer Tent is the most waterproof.",
        context="From the our product list,"
        " the alpine explorer tent is the most waterproof."
        " The Adventure Dining Table has higher weight.",
        question="Which tent is the most waterproof?",
    )
    print(relevance_score)

    # Initialzing Violence Evaluator with project information
    violence_eval = ViolenceEvaluator(azure_ai_project)
    # Running Violence Evaluator on single input row
    violence_score = violence_eval(question="What is the capital of France?", answer="Paris.")
    print(violence_score)

    #print(rs)
    # Convert the Python list (array) to a JSON string
    data = [{
        "chat_input": question,
        "answer": result,
        "context": str(context)
    },
    {
        "chat_input": question,
        "answer": result,
        "context": str(context)
    }]
    json_data = json.dumps(data, indent=4)

    # Write the JSON string to a file
    #with open('chat_data.jsonl', 'w') as file:
    #    file.write(json_data)
    # Write the JSONL file
    with open('chat_data.jsonl', 'w') as file:
        for entry in data:
            # Convert each dictionary to a JSON string and write it as a new line
            file.write(json.dumps(entry) + '\n')

    print("JSON file created successfully!")


    return {"answer": result, "context": context}

# Helper methods
def load_jsonl(path):
    with open(path, "r") as f:
        return [json.loads(line) for line in f.readlines()]


def copilot_wrapper(*, chat_input, **kwargs):
    #from copilot_flow.copilot import get_chat_response#

    result = get_response(chat_input)

    parsedResult = {"answer": str(result["answer"]), "context": str(result["context"])}
    return parsedResult


def run_evaluation(eval_name, dataset_path):

    # Initialize Azure OpenAI Connection with your environment variables
    model_config = AzureOpenAIModelConfiguration(
        azure_endpoint=azure_endpoint,
        api_key=api_key,
        azure_deployment=azure_chatdeployment,
        api_version=api_version,
    )


    # Initializing Evaluators
    relevance_eval = RelevanceEvaluator(model_config)
    groundedness_eval = GroundednessEvaluator(model_config)
    coherence_eval = CoherenceEvaluator(model_config)

    output_path = "./eval_results.jsonl"
    print(" Data Path: ", dataset_path)

    result = evaluate(
        target=copilot_wrapper,
        evaluation_name=eval_name,
        data=dataset_path,
        evaluators={
            "relevance": relevance_eval,
            "groundedness": groundedness_eval,
            "coherence": coherence_eval,
        },
        evaluator_config={
            "relevance": {"question": "${data.chat_input}", "answer": "${data.answer}", "context": "${data.context}"},
            "coherence": {"question": "${data.chat_input}", "answer": "${data.answer}", "context": "${data.context}"},
        },
        # to log evaluation to the cloud AI Studio project
        azure_ai_project=azure_ai_project,
    )

    tabular_result = pd.DataFrame(result.get("rows"))
    tabular_result.to_json(output_path, orient="records", lines=True)

    return result, tabular_result

if __name__ == "__main__":
    print("running chat request")
    answer = get_response("show me top 5 technology leadership candidates", [])
    eval_name = "chat-eval"
    #dataset_path = "./eval_dataset.jsonl"
    dataset_path = os.path.join(pathlib.Path(__file__).parent.resolve(), "chat_data.jsonl")
    #https://learn.microsoft.com/en-us/azure/ai-studio/tutorials/copilot-sdk-build-rag?tabs=azure-portal
    #https://learn.microsoft.com/en-us/azure/ai-studio/tutorials/copilot-sdk-evaluate-deploy

    df = pd.read_json(dataset_path, lines=True)
    print(df)

    result, tabular_result = run_evaluation(
        eval_name=eval_name, dataset_path=dataset_path
    )
```

## data set for evaluation

- chat_data.jsonl

```
{"chat_input": "show me top 5 technology leadership candidates", "answer": "Here are five top technology leadership candidates based on the provided documents:\n\n1. **Christina Raab**\n   - **Role:** Market Unit Lead, Accenture Germany, Switzerland, Austria\n   - **Expertise:** Digital Transformation, Business Strategy, Management Consulting\n   - **Languages:** English, German, Italian\n   - [LinkedIn](https://www.linkedin.com/in/christinaraab)\n\n2. **Emma McGuigan**\n   - **Role:** Lead - Intelligent Platform Services, Accenture\n   - **Expertise:** IT Strategy, Governance, Leadership\n   - **Focus:** Digital enterprise transformation, technology-enabled solutions\n   - [LinkedIn](https://www.linkedin.com/in/emmatmcguigan)\n\n3. **AeroFarms Board Member**\n   - **Role:** Board Member, AeroFarms\n   - **Expertise:** Sustainability, Data and Analytics, Urban Farming Technologies\n   - **Focus:** Disruptive tech in urban farming\n   - [LinkedIn](https://www.linkedin.com/in/)\n\n4. **The Forum of Young Global Leaders Board Member**\n   - **Role:** Board Member, The Forum of Young Global Leaders\n   - **Expertise:** Leadership Development, Global Change Initiatives\n   - **Focus:** Developing future leaders for positive change\n   - [LinkedIn](https://www.linkedin.com/in/)\n\n5. **Accenture Global Management Committee Member**\n   - **Role:** Global Management Committee Member, Accenture\n   - **Expertise:** Sustainability Services, Corporate Responsibility\n   - **Focus:** Sustainability strategy, global leadership\n   - [LinkedIn](https://www.linkedin.com/in/)\n\nThese candidates are highly experienced in technology leadership and digital transformation. \ud83c\udf1f", "context": "[{'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTkRFcExuQmtaZzIvY29udGVudF9jaHVua3NfMC5qc29u0', 'title': 'Kontakt', 'content': 'Kontakt\\nwww.linkedin.com/in/christinaraab\\n(LinkedIn)\\nwww.accenture.com/de-de\\n(Company)\\nTop-Kenntnisse\\nManagement Consulting\\nBusiness Strategy\\nDigital Transformation\\nLanguages\\nEnglish (Professional Working)\\nGerman (Native or Bilingual)\\nItalian (Professional Working)\\nChristina Raab\\nMarket Unit Lead Accenture Germany, Switzerland, Austria | Digital\\nEvangelist | Experience Enthusiast | Ardent Collaborator | Value\\nAlchemist\\nMetropolregion M\u00fcnchen\\nZusammenfassung\\nMy pride and joy is helping Accenture clients \u2013 both existing and\\npotential \u2013 build and achieve bright new futures. For me, nothing\\nis more fulfilling than learning together, sharing experiences and\\nexchanging knowledge in a global movement that makes our world\\nfunction better and act smarter. \\nIt\u2019s my firm conviction that to create and sustain meaningful value in\\nour prevailing era of rapid change and compressed transformations\\nrequires adopting thoroughly fresh ways of working, exciting\\nnew technologies and an altogether different style of customer\\nexperience. As such, I am passionate about emerging trends and\\ngetting stuck into the very latest innovations.\\nWhen I first joined Accenture 22 years ago it was frankly because I\\nyearned to be a part of a community of inspiring people dedicated\\nto enhancing the world around us. That very much continues to be\\nmy driving force today. Having initially spearheaded our products\\norganization across ASG, I nowadays specialize in supporting\\nclients from a wide range of industry verticals \u2013 collaborating to\\naccelerate their digital journeys and deftly handle complex business\\ntransformations.\\nBerufserfahrung\\nAccenture DACH\\nMarket Unit Lead Accenture Germany, Switzerland and Austria\\nSeptember 2021\\xa0-\\xa0Present\\xa0(1 Jahr 6 Monate)\\nAccenture\\n22 Jahre 5 Monate\\nMember of Accenture Global Management Committee\\nSeptember 2021\\xa0-\\xa0Present\\xa0(1 Jahr 6 Monate)\\n Page 1 of 2\\nhttps://www.linkedin.com/in/christinaraab?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BI89GrLhRSceeGYgCTzWNxQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.linkedin.com/in/christinaraab?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BI89GrLhRSceeGYgCTzWNxQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.accenture.com/de-de\\nhttps://www.accenture.com/de-de\\n \\nBoard Member of Accenture Holding GmbH & Co. KG\\nSeptember 2019\\xa0-\\xa0Present\\xa0(3 Jahre 6 Monate)\\nSenior Managing Director\\nDezember 2020\\xa0-\\xa0August 2021\\xa0(9 Monate)\\nLeading Business Unit \u201cProducts\u201d in DACH and Russia; Design and\\nimplementation of market and portfolio strategies \\nManaging Director\\nSeptember 2010\\xa0-\\xa0November 2020\\xa0(10 Jahre 3 Monate)\\nResponsible for development and implementation of transformational growth\\nstrategies for international clients in the Automotive, Industrial and Consumer\\nGoods sector at global scale. Management and development of global client\\nand Accenture Teams \\nStrategy Consultant\\nOktober 2000\\xa0-\\xa0September 2010\\xa0(10 Jahre)\\nDriving large scale, strategic projects in the areas of M&A, Marketing, Sales\\nand Service\\nAusbildung\\nUniversit\u00e0 degli Studi di Parma\\nUniversit\u00e4t Passau\\nDiplom-Kauffrau,\\xa0Betriebswirtschaftslehre\\n Page 2 of 2', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(41).pdf'}, {'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTXpJcExuQmtaZzIvY29udGVudF9jaHVua3NfMS5qc29u0', 'title': 'Contact', 'content': 'Put simply focused on clients, people and\\npurpose \\nExperience\\nAeroFarms\\nBoard Member\\nMarch 2014\\xa0-\\xa0Present\\xa0(9 years)\\nNewark, New Jersey, United States\\nAdvisory Board and then Full Board Member of the world\u2019s most innovative\\nand one of the fastest growing disruptive tech urban farming businesses. With\\nstrong links to sustainability, data and analytics and the intersection of digital\\nand physical technologies. As well as just producing great product! \\nThe Forum of Young Global Leaders\\nBoard Member\\nFebruary 2019\\xa0-\\xa0Present\\xa0(4 years 1 month)\\nGeneva, Switzerland\\nMember of the board overseeing the activities of the Young Global Leaders\\nForum. A network and platforms set up by the World Economic Forum to\\ndevelop the future leaders we need to create positive change. \\nBlue Rose Compass\\nVarious not-for-profit board roles\\nFebruary 2008\\xa0-\\xa0Present\\xa0(15 years 1 month)\\nUnited States, United Kingdom, China, Australia\\nAccenture\\n15 years 2 months\\n Page 2 of 4\\n \\nGlobal Management Committee Member, Sustainability Services Lead\\n& Chief Responsibility Officer\\nDecember 2020\\xa0-\\xa0Present\\xa0(2 years 3 months)\\nLondon, England, United Kingdom\\nAccenture Executive Lead Sponsor - WEF & UN Global Compact \\nOctober 2014\\xa0-\\xa0October 2021\\xa0(7 years 1 month)\\nSenior Managing Director - Accenture Strategy Europe Lead; Global\\nSustainability Strategy Lead\\nDecember 2017\\xa0-\\xa02020\\xa0(3 years)\\nLondon, England, United Kingdom\\nMultiple Roles - Growth, Strategy, Sustainability & Technology\\nJanuary 2008\\xa0-\\xa0December 2017\\xa0(10 years)\\nShanghai, China & London, England\\nGlobal Growth, Strategy & Sustainability Lead - Accenture Strategy - leading\\nall organic & inorganic growth for Accenture Strategy globally - 3 years \\nGlobal Lead - Digital Transformation of Industries (3 years)\\nAsia-Pacific Cross-Industry Strategy Lead (Shanghai for 3 years)\\nCo-founded Accenture\u2019s Sustainability Strategy practice in 2008. Led both\\nEurope and Asia Pacific regions before global lead role\\nMcKinsey & Company\\nSenior Advisor\\nFebruary 2006\\xa0-\\xa0March 2007\\xa0(1 year 2 months)\\nStrategy Practice / Business in Society\\nABIS - The Academy of Business in Society\\nJoint Founding Executive Director\\nFebruary 2003\\xa0-\\xa0February 2007\\xa0(4 years 1 month)\\nBrussels, Belgium\\nAccenture\\nStrategy Consulting\\nAugust 2000\\xa0-\\xa0January 2003\\xa0(2 years 6 months)\\n2000-2003 \\nAndersen Consulting Summer Vacation Placement - 2000 & 2001\\nAccenture Graduate Training Programme - 2002-3\\n Page 3 of 4\\n \\nCosta Rica renewables project, The British Institute, Lastorders.com,\\nlastminute.com, Ford Motor Co.\\nGap Year Roles & While Studying\\nFebruary 1996\\xa0-\\xa0February 2000\\xa0(4 years 1 month)\\nLondon, United Kingdom & Costa Rica\\nEducation\\nUniversity of Oxford\\nBusiness Fellow, Smith School of Environment & Economics,\\xa0Business,\\nEconomics & Policy\\xa0\u00b7\\xa0(2010\\xa0-\\xa02020)\\nHarvard University, John F. Kennedy School of Government\\nGlobal Leadership and Public Policy for the 21st Century \u00b7\\xa0(2014\\xa0-\\xa02014)\\nYale University\\nGlobal Issues & Foundations for Leadership in the 21st\\nCentury \u00b7\\xa0(2013\\xa0-\\xa02013)\\nUniversity of Oxford\\nOxford University Business Economics Programme\\n(OUBEP),\\xa0Economics\\xa0\u00b7\\xa0(2011\\xa0-\\xa02011)\\nINSEAD\\nInternational Executive Programme,\\xa0IEP\\xa0\u00b7\\xa0(2007\\xa0-\\xa02007)\\n Page 4 of 4', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(32).pdf'}, {'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTXpVcExuQmtaZzIvY29udGVudF9jaHVua3NfMC5qc29u0', 'title': 'Contact', 'content': 'Contact\\nwww.linkedin.com/in/\\nemmatmcguigan (LinkedIn)\\nTop Skills\\nIT Strategy\\nGovernance\\nLeadership\\nEmma McGuigan\\nLead - Intelligent Platform Services at Accenture\\nUnited Kingdom\\nSummary\\nI am a curious person at my core and channel that curiosity into\\nexamining the world to figure out how we can come together to be\\nbold, dream big, and drive change. I joined Accenture UK in the\\nmid-90s and have been lucky enough to experience a breadth of\\nopportunities working with clients and partners to create solutions\\nand I love the art of the possible that technology gives you. I have\\nspent my career architecting and delivering technology enabled\\nsolutions to clients in a range of industries and I have had the\\nprivilege to work with clients on programs from the cutting edge of\\ninnovation to large scale core solutions.\\nToday, leading Accenture\u2019s Intelligent Platform Business, I am\\nincredibly excited about the being at the heart of the digital enterprise\\ntransformation: working with incredible partner organizations to\\nfundamentally shift how we approach technology and innovation;\\nunlocking new business models, new ways of working and new\\nopportunities for our clients and for our people. \\nI have spent my career architecting and delivering technology\\nenabled solutions to clients in a range of industries and I have had\\nthe privilege to work with clients on programs from the cutting edge\\nof innovation to large scale core solutions.\\nEmpowering young people to embrace technology is one of my huge\\ndrivers. I think empowerment often comes in the form of recovery\\nfrom failure. Too much of our education system is about making\\nhumans \"exam passing machines\" rather than preparing them for\\nworking life by showing them how to grow when things don\u2019t go to\\nplan. People are so much more resilient, so much more powerful,\\nso much more empathetic after they come back from a failure, a\\nchallenge or a loss and it\u2019s one of my goals as a leader to help my\\nteams to embrace. \\n Page 1 of 3\\nhttps://www.linkedin.com/in/emmatmcguigan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BHbBz0XHxQGCs5BK4Q2VEVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.linkedin.com/in/emmatmcguigan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BHbBz0XHxQGCs5BK4Q2VEVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\n \\nI\u2019d love to hear from you so please drop me a message here or\\nfollow me on Twitter at @emmcgoo\\nExperience\\nAccenture\\n8 years 9 months\\nLead - Intelligent Platform Services\\nSeptember 2020\\xa0-\\xa0Present\\xa0(2 years 6 months)\\nLondon, England, United Kingdom\\nAccenture\\'s Intelligent Platform Services, helps clients achieve enterprise-\\nwide transformation by bringing Accenture\u2019s deep technology, functional\\nand industry expertise across SAP, Microsoft, Oracle, Salesforce, Workday,\\nAdobe and other leading platforms. For these IPS platforms, Emma oversees\\nAccenture\u2019s relationships, strategy and capabilities globally and across all\\nindustries. She is a member of Accenture\u2019s Global Management Committee\\nLead - Accenture Microsoft Business Group\\nMarch 2019\\xa0-\\xa0May 2021\\xa0(2 years 3 months)\\nGreater Seattle Area\\nResponsible for Accenture\u2019s combined business with Microsoft, including\\nthe industry leading JV, Avanade, Emma overseas this strategic alliance\\nof 45,000 Accenture & Avanade employees helping organisations across\\nall industries navigate disruption and drive innovation through harnessing\\nMicrosoft technologies. \\nGroup Technology Officer - Communications, Media & Technology\\nMarch 2017\\xa0-\\xa0February 2019\\xa0(2 years)\\nLondon, United Kingdom\\nResponsible for Accenture\\'s CMT technology strategy, overseeing current and\\nfuture partnerships, and positioning around emerging technologies including\\nmachine learning, artificial intelligence and cognitive computing to help drive\\nleading-edge technology consulting and innovation to clients', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(35).pdf'}]"}
{"chat_input": "show me top 5 technology leadership candidates", "answer": "Here are five top technology leadership candidates based on the provided documents:\n\n1. **Christina Raab**\n   - **Role:** Market Unit Lead, Accenture Germany, Switzerland, Austria\n   - **Expertise:** Digital Transformation, Business Strategy, Management Consulting\n   - **Languages:** English, German, Italian\n   - [LinkedIn](https://www.linkedin.com/in/christinaraab)\n\n2. **Emma McGuigan**\n   - **Role:** Lead - Intelligent Platform Services, Accenture\n   - **Expertise:** IT Strategy, Governance, Leadership\n   - **Focus:** Digital enterprise transformation, technology-enabled solutions\n   - [LinkedIn](https://www.linkedin.com/in/emmatmcguigan)\n\n3. **AeroFarms Board Member**\n   - **Role:** Board Member, AeroFarms\n   - **Expertise:** Sustainability, Data and Analytics, Urban Farming Technologies\n   - **Focus:** Disruptive tech in urban farming\n   - [LinkedIn](https://www.linkedin.com/in/)\n\n4. **The Forum of Young Global Leaders Board Member**\n   - **Role:** Board Member, The Forum of Young Global Leaders\n   - **Expertise:** Leadership Development, Global Change Initiatives\n   - **Focus:** Developing future leaders for positive change\n   - [LinkedIn](https://www.linkedin.com/in/)\n\n5. **Accenture Global Management Committee Member**\n   - **Role:** Global Management Committee Member, Accenture\n   - **Expertise:** Sustainability Services, Corporate Responsibility\n   - **Focus:** Sustainability strategy, global leadership\n   - [LinkedIn](https://www.linkedin.com/in/)\n\nThese candidates are highly experienced in technology leadership and digital transformation. \ud83c\udf1f", "context": "[{'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTkRFcExuQmtaZzIvY29udGVudF9jaHVua3NfMC5qc29u0', 'title': 'Kontakt', 'content': 'Kontakt\\nwww.linkedin.com/in/christinaraab\\n(LinkedIn)\\nwww.accenture.com/de-de\\n(Company)\\nTop-Kenntnisse\\nManagement Consulting\\nBusiness Strategy\\nDigital Transformation\\nLanguages\\nEnglish (Professional Working)\\nGerman (Native or Bilingual)\\nItalian (Professional Working)\\nChristina Raab\\nMarket Unit Lead Accenture Germany, Switzerland, Austria | Digital\\nEvangelist | Experience Enthusiast | Ardent Collaborator | Value\\nAlchemist\\nMetropolregion M\u00fcnchen\\nZusammenfassung\\nMy pride and joy is helping Accenture clients \u2013 both existing and\\npotential \u2013 build and achieve bright new futures. For me, nothing\\nis more fulfilling than learning together, sharing experiences and\\nexchanging knowledge in a global movement that makes our world\\nfunction better and act smarter. \\nIt\u2019s my firm conviction that to create and sustain meaningful value in\\nour prevailing era of rapid change and compressed transformations\\nrequires adopting thoroughly fresh ways of working, exciting\\nnew technologies and an altogether different style of customer\\nexperience. As such, I am passionate about emerging trends and\\ngetting stuck into the very latest innovations.\\nWhen I first joined Accenture 22 years ago it was frankly because I\\nyearned to be a part of a community of inspiring people dedicated\\nto enhancing the world around us. That very much continues to be\\nmy driving force today. Having initially spearheaded our products\\norganization across ASG, I nowadays specialize in supporting\\nclients from a wide range of industry verticals \u2013 collaborating to\\naccelerate their digital journeys and deftly handle complex business\\ntransformations.\\nBerufserfahrung\\nAccenture DACH\\nMarket Unit Lead Accenture Germany, Switzerland and Austria\\nSeptember 2021\\xa0-\\xa0Present\\xa0(1 Jahr 6 Monate)\\nAccenture\\n22 Jahre 5 Monate\\nMember of Accenture Global Management Committee\\nSeptember 2021\\xa0-\\xa0Present\\xa0(1 Jahr 6 Monate)\\n Page 1 of 2\\nhttps://www.linkedin.com/in/christinaraab?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BI89GrLhRSceeGYgCTzWNxQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.linkedin.com/in/christinaraab?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BI89GrLhRSceeGYgCTzWNxQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.accenture.com/de-de\\nhttps://www.accenture.com/de-de\\n \\nBoard Member of Accenture Holding GmbH & Co. KG\\nSeptember 2019\\xa0-\\xa0Present\\xa0(3 Jahre 6 Monate)\\nSenior Managing Director\\nDezember 2020\\xa0-\\xa0August 2021\\xa0(9 Monate)\\nLeading Business Unit \u201cProducts\u201d in DACH and Russia; Design and\\nimplementation of market and portfolio strategies \\nManaging Director\\nSeptember 2010\\xa0-\\xa0November 2020\\xa0(10 Jahre 3 Monate)\\nResponsible for development and implementation of transformational growth\\nstrategies for international clients in the Automotive, Industrial and Consumer\\nGoods sector at global scale. Management and development of global client\\nand Accenture Teams \\nStrategy Consultant\\nOktober 2000\\xa0-\\xa0September 2010\\xa0(10 Jahre)\\nDriving large scale, strategic projects in the areas of M&A, Marketing, Sales\\nand Service\\nAusbildung\\nUniversit\u00e0 degli Studi di Parma\\nUniversit\u00e4t Passau\\nDiplom-Kauffrau,\\xa0Betriebswirtschaftslehre\\n Page 2 of 2', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(41).pdf'}, {'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTXpJcExuQmtaZzIvY29udGVudF9jaHVua3NfMS5qc29u0', 'title': 'Contact', 'content': 'Put simply focused on clients, people and\\npurpose \\nExperience\\nAeroFarms\\nBoard Member\\nMarch 2014\\xa0-\\xa0Present\\xa0(9 years)\\nNewark, New Jersey, United States\\nAdvisory Board and then Full Board Member of the world\u2019s most innovative\\nand one of the fastest growing disruptive tech urban farming businesses. With\\nstrong links to sustainability, data and analytics and the intersection of digital\\nand physical technologies. As well as just producing great product! \\nThe Forum of Young Global Leaders\\nBoard Member\\nFebruary 2019\\xa0-\\xa0Present\\xa0(4 years 1 month)\\nGeneva, Switzerland\\nMember of the board overseeing the activities of the Young Global Leaders\\nForum. A network and platforms set up by the World Economic Forum to\\ndevelop the future leaders we need to create positive change. \\nBlue Rose Compass\\nVarious not-for-profit board roles\\nFebruary 2008\\xa0-\\xa0Present\\xa0(15 years 1 month)\\nUnited States, United Kingdom, China, Australia\\nAccenture\\n15 years 2 months\\n Page 2 of 4\\n \\nGlobal Management Committee Member, Sustainability Services Lead\\n& Chief Responsibility Officer\\nDecember 2020\\xa0-\\xa0Present\\xa0(2 years 3 months)\\nLondon, England, United Kingdom\\nAccenture Executive Lead Sponsor - WEF & UN Global Compact \\nOctober 2014\\xa0-\\xa0October 2021\\xa0(7 years 1 month)\\nSenior Managing Director - Accenture Strategy Europe Lead; Global\\nSustainability Strategy Lead\\nDecember 2017\\xa0-\\xa02020\\xa0(3 years)\\nLondon, England, United Kingdom\\nMultiple Roles - Growth, Strategy, Sustainability & Technology\\nJanuary 2008\\xa0-\\xa0December 2017\\xa0(10 years)\\nShanghai, China & London, England\\nGlobal Growth, Strategy & Sustainability Lead - Accenture Strategy - leading\\nall organic & inorganic growth for Accenture Strategy globally - 3 years \\nGlobal Lead - Digital Transformation of Industries (3 years)\\nAsia-Pacific Cross-Industry Strategy Lead (Shanghai for 3 years)\\nCo-founded Accenture\u2019s Sustainability Strategy practice in 2008. Led both\\nEurope and Asia Pacific regions before global lead role\\nMcKinsey & Company\\nSenior Advisor\\nFebruary 2006\\xa0-\\xa0March 2007\\xa0(1 year 2 months)\\nStrategy Practice / Business in Society\\nABIS - The Academy of Business in Society\\nJoint Founding Executive Director\\nFebruary 2003\\xa0-\\xa0February 2007\\xa0(4 years 1 month)\\nBrussels, Belgium\\nAccenture\\nStrategy Consulting\\nAugust 2000\\xa0-\\xa0January 2003\\xa0(2 years 6 months)\\n2000-2003 \\nAndersen Consulting Summer Vacation Placement - 2000 & 2001\\nAccenture Graduate Training Programme - 2002-3\\n Page 3 of 4\\n \\nCosta Rica renewables project, The British Institute, Lastorders.com,\\nlastminute.com, Ford Motor Co.\\nGap Year Roles & While Studying\\nFebruary 1996\\xa0-\\xa0February 2000\\xa0(4 years 1 month)\\nLondon, United Kingdom & Costa Rica\\nEducation\\nUniversity of Oxford\\nBusiness Fellow, Smith School of Environment & Economics,\\xa0Business,\\nEconomics & Policy\\xa0\u00b7\\xa0(2010\\xa0-\\xa02020)\\nHarvard University, John F. Kennedy School of Government\\nGlobal Leadership and Public Policy for the 21st Century \u00b7\\xa0(2014\\xa0-\\xa02014)\\nYale University\\nGlobal Issues & Foundations for Leadership in the 21st\\nCentury \u00b7\\xa0(2013\\xa0-\\xa02013)\\nUniversity of Oxford\\nOxford University Business Economics Programme\\n(OUBEP),\\xa0Economics\\xa0\u00b7\\xa0(2011\\xa0-\\xa02011)\\nINSEAD\\nInternational Executive Programme,\\xa0IEP\\xa0\u00b7\\xa0(2007\\xa0-\\xa02007)\\n Page 4 of 4', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(32).pdf'}, {'id': 'aHR0cHM6Ly9zdGJhYmFsYWkxNTI4MDc1MzY2OTEuYmxvYi5jb3JlLndpbmRvd3MubmV0L2Fpc3R1ZGlvcHJvZmlsZS1jaHVua3MvYUhSMGNITTZMeTl6ZEdKaFltRnNZV2t4TlRJNE1EYzFNelkyT1RFdVlteHZZaTVqYjNKbExuZHBibVJ2ZDNNdWJtVjBMMlpwYkdWMWNHeHZZV1F0WVdsemRIVmthVzl3Y205bWFXeGxMMUJ5YjJacGJHVWxNakFvTXpVcExuQmtaZzIvY29udGVudF9jaHVua3NfMC5qc29u0', 'title': 'Contact', 'content': 'Contact\\nwww.linkedin.com/in/\\nemmatmcguigan (LinkedIn)\\nTop Skills\\nIT Strategy\\nGovernance\\nLeadership\\nEmma McGuigan\\nLead - Intelligent Platform Services at Accenture\\nUnited Kingdom\\nSummary\\nI am a curious person at my core and channel that curiosity into\\nexamining the world to figure out how we can come together to be\\nbold, dream big, and drive change. I joined Accenture UK in the\\nmid-90s and have been lucky enough to experience a breadth of\\nopportunities working with clients and partners to create solutions\\nand I love the art of the possible that technology gives you. I have\\nspent my career architecting and delivering technology enabled\\nsolutions to clients in a range of industries and I have had the\\nprivilege to work with clients on programs from the cutting edge of\\ninnovation to large scale core solutions.\\nToday, leading Accenture\u2019s Intelligent Platform Business, I am\\nincredibly excited about the being at the heart of the digital enterprise\\ntransformation: working with incredible partner organizations to\\nfundamentally shift how we approach technology and innovation;\\nunlocking new business models, new ways of working and new\\nopportunities for our clients and for our people. \\nI have spent my career architecting and delivering technology\\nenabled solutions to clients in a range of industries and I have had\\nthe privilege to work with clients on programs from the cutting edge\\nof innovation to large scale core solutions.\\nEmpowering young people to embrace technology is one of my huge\\ndrivers. I think empowerment often comes in the form of recovery\\nfrom failure. Too much of our education system is about making\\nhumans \"exam passing machines\" rather than preparing them for\\nworking life by showing them how to grow when things don\u2019t go to\\nplan. People are so much more resilient, so much more powerful,\\nso much more empathetic after they come back from a failure, a\\nchallenge or a loss and it\u2019s one of my goals as a leader to help my\\nteams to embrace. \\n Page 1 of 3\\nhttps://www.linkedin.com/in/emmatmcguigan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BHbBz0XHxQGCs5BK4Q2VEVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\nhttps://www.linkedin.com/in/emmatmcguigan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BHbBz0XHxQGCs5BK4Q2VEVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile\\n \\nI\u2019d love to hear from you so please drop me a message here or\\nfollow me on Twitter at @emmcgoo\\nExperience\\nAccenture\\n8 years 9 months\\nLead - Intelligent Platform Services\\nSeptember 2020\\xa0-\\xa0Present\\xa0(2 years 6 months)\\nLondon, England, United Kingdom\\nAccenture\\'s Intelligent Platform Services, helps clients achieve enterprise-\\nwide transformation by bringing Accenture\u2019s deep technology, functional\\nand industry expertise across SAP, Microsoft, Oracle, Salesforce, Workday,\\nAdobe and other leading platforms. For these IPS platforms, Emma oversees\\nAccenture\u2019s relationships, strategy and capabilities globally and across all\\nindustries. She is a member of Accenture\u2019s Global Management Committee\\nLead - Accenture Microsoft Business Group\\nMarch 2019\\xa0-\\xa0May 2021\\xa0(2 years 3 months)\\nGreater Seattle Area\\nResponsible for Accenture\u2019s combined business with Microsoft, including\\nthe industry leading JV, Avanade, Emma overseas this strategic alliance\\nof 45,000 Accenture & Avanade employees helping organisations across\\nall industries navigate disruption and drive innovation through harnessing\\nMicrosoft technologies. \\nGroup Technology Officer - Communications, Media & Technology\\nMarch 2017\\xa0-\\xa0February 2019\\xa0(2 years)\\nLondon, United Kingdom\\nResponsible for Accenture\\'s CMT technology strategy, overseeing current and\\nfuture partnerships, and positioning around emerging technologies including\\nmachine learning, artificial intelligence and cognitive computing to help drive\\nleading-edge technology consulting and innovation to clients', 'url': 'https://stbabalai152807536691.blob.core.windows.net/fileupload-aistudioprofile/Profile%20(35).pdf'}]"}

```