# Azure Open AI Assitant with Rag AI Search

## Use Case

- Access internal documents using embeddings to ground with the user's knowledge
- ask questions to get answers from the documents
- Using Azure Open AI gpt 4 turbo
- Idea here is to learn to show how rag can be implemented

## Requirements

- Azure Subscription
- Azure Machine learning
- Azure Open AI gpt 4 turbo
- Python

## Code

- install libraries

```
%pip install azure-search-documents
```

```
%pip install openai
```

```
%pip install python-dotenv
```

- Now load the environment variables

```
import openai
import json
```

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Configure search

```
search_endpoint = config["AZURE_SEARCH_SERVICE_ENDPOINT"]
index_name = config["AZURE_SEARCH_INDEX_NAME"]
key = config["AZURE_SEARCH_API_KEY"]
```

- Configure open ai client

```
import os
from openai import AzureOpenAI

client = AzureOpenAI(
  azure_endpoint = "https://xxxxx.openai.azure.com/", 
  api_key="xxxxxxxxxxxxxxxxxxxxxxx",  
  api_version="2024-02-15-preview"
)
```

- search client

```
from azure.core.credentials import AzureKeyCredential
from azure.search.documents import SearchClient


search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
```

- search

```
from typing import Optional

def searchai(searchtext):
    searchcontent= ""
    search_client = SearchClient(search_endpoint, index_name, AzureKeyCredential(key))
    results = search_client.search(search_text="CEO Role")
    for result in results:
        print("{}: {})".format(result["url"], result["content"]))
        searchcontent += f"{result['content']} : {result['url']}"
    return json.dumps(searchcontent)
```

```
import json

def show_json(obj):
    display(json.loads(obj.model_dump_json()))
```

- model name

```
model_name = f'gpt-4-turbo'
```

- define poll and pull

```
def poll_run_till_completion(
    client: AzureOpenAI,
    thread_id: str,
    run_id: str,
    available_functions: dict,
    verbose: bool,
    max_steps: int = 10,
    wait: int = 3,
) -> None:
    """
    Poll a run until it is completed or failed or exceeds a certain number of iterations (MAX_STEPS)
    with a preset wait in between polls

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param run_id: Run ID
    @param assistant_id: Assistant ID
    @param verbose: Print verbose output
    @param max_steps: Maximum number of steps to poll
    @param wait: Wait time in seconds between polls

    """

    if (client is None and thread_id is None) or run_id is None:
        print("Client, Thread ID and Run ID are required.")
        return
    try:
        cnt = 0
        while cnt < max_steps:
            run = client.beta.threads.runs.retrieve(thread_id=thread_id, run_id=run_id)
            if verbose:
                print("Poll {}: {}".format(cnt, run.status))
            cnt += 1
            if run.status == "requires_action":
                tool_responses = []
                if (
                    run.required_action.type == "submit_tool_outputs"
                    and run.required_action.submit_tool_outputs.tool_calls is not None
                ):
                    tool_calls = run.required_action.submit_tool_outputs.tool_calls

                    for call in tool_calls:
                        if call.type == "function":
                            if call.function.name not in available_functions:
                                raise Exception("Function requested by the model does not exist")
                            function_to_call = available_functions[call.function.name]
                            tool_response = function_to_call(**json.loads(call.function.arguments))
                            tool_responses.append({"tool_call_id": call.id, "output": tool_response})

                run = client.beta.threads.runs.submit_tool_outputs(
                    thread_id=thread_id, run_id=run.id, tool_outputs=tool_responses
                )
            if run.status == "failed":
                print("Run failed.")
                break
            if run.status == "completed":
                break
            time.sleep(wait)

    except Exception as e:
        print(e)
```

- Create message

```
def create_message(
    client: AzureOpenAI,
    thread_id: str,
    role: str = "",
    content: str = "",
    file_ids: Optional[list] = None,
    metadata: Optional[dict] = None,
    message_id: Optional[str] = None,
) -> any:
    """
    Create a message in a thread using the client.

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param role: Message role (user or assistant)
    @param content: Message content
    @param file_ids: Message file IDs
    @param metadata: Message metadata
    @param message_id: Message ID
    @return: Message object

    """
    if metadata is None:
        metadata = {}
    if file_ids is None:
        file_ids = []

    if client is None:
        print("Client parameter is required.")
        return None

    if thread_id is None:
        print("Thread ID is required.")
        return None

    try:
        if message_id is not None:
            return client.beta.threads.messages.retrieve(thread_id=thread_id, message_id=message_id)

        if file_ids is not None and len(file_ids) > 0 and metadata is not None and len(metadata) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, file_ids=file_ids, metadata=metadata
            )

        if file_ids is not None and len(file_ids) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, file_ids=file_ids
            )

        if metadata is not None and len(metadata) > 0:
            return client.beta.threads.messages.create(
                thread_id=thread_id, role=role, content=content, metadata=metadata
            )

        return client.beta.threads.messages.create(thread_id=thread_id, role=role, content=content)

    except Exception as e:
        print(e)
        return None
```

- retrieve message

```
def retrieve_and_print_messages(
    client: AzureOpenAI, thread_id: str, verbose: bool, out_dir: Optional[str] = None
) -> any:
    """
    Retrieve a list of messages in a thread and print it out with the query and response

    @param client: OpenAI client
    @param thread_id: Thread ID
    @param verbose: Print verbose output
    @param out_dir: Output directory to save images
    @return: Messages object

    """

    if client is None and thread_id is None:
        print("Client and Thread ID are required.")
        return None
    try:
        messages = client.beta.threads.messages.list(thread_id=thread_id)
        display_role = {"user": "User query", "assistant": "Assistant response"}

        prev_role = None

        if verbose:
            print("\n\nCONVERSATION:")
        for md in reversed(messages.data):
            if prev_role == "assistant" and md.role == "user" and verbose:
                print("------ \n")

            for mc in md.content:
                # Check if valid text field is present in the mc object
                if mc.type == "text":
                    txt_val = mc.text.value
                # Check if valid image field is present in the mc object
                elif mc.type == "image_file":
                    image_data = client.files.content(mc.image_file.file_id)
                    if out_dir is not None:
                        out_dir_path = Path(out_dir)
                        if out_dir_path.exists():
                            image_path = out_dir_path / (mc.image_file.file_id + ".png")
                            with image_path.open("wb") as f:
                                f.write(image_data.read())

                if verbose:
                    if prev_role == md.role:
                        print(txt_val)
                    else:
                        print("{}:\n{}".format(display_role[md.role], txt_val))
            prev_role = md.role
        return messages
    except Exception as e:
        print(e)
        return None
```

- now ask questions

```
import time

name = "aisearch-assistant"
instructions = """You are an assistant designed to help people answer questions.

You have access to query the web using azure cognitive ai Search. You should call ai search whenever a question requires up to date information or could benefit from profiles data.
"""

message = {"role": "user", "content": "List top 5 candidates for CEO role with details?"}


tools = [
    {
        "type": "function",
        "function": {
            "name": "searchai",
            "description": "Searches AI Search to get up-to-date information from the web.",
            "parameters": {
                "type": "object",
                "properties": {
                    "searchtext": {
                        "type": "string",
                        "description": "The search query",
                    }
                },
                "required": ["searchtext"],
            },
        },
    }
]

available_functions = {"searchai": searchai}
verbose_output = True

#client = AzureOpenAI(api_key=aoai_api_key, api_version=api_version, azure_endpoint=azure_endpoint)

assistant = client.beta.assistants.create(
    name=name, description="", instructions=instructions, tools=tools, model=model_name
)

thread = client.beta.threads.create()

create_message(client, thread.id, message["role"], message["content"])


run = client.beta.threads.runs.create(thread_id=thread.id, assistant_id=assistant.id, instructions=instructions)
poll_run_till_completion(
    client=client, thread_id=thread.id, run_id=run.id, available_functions=available_functions, verbose=verbose_output
)
messages = retrieve_and_print_messages(client=client, thread_id=thread.id, verbose=verbose_output)

  
messages = client.beta.threads.messages.list(
  thread_id=thread.id)
print(messages.data[0].content[0].text.value)
```

- output

```
Poll 0: queued
Poll 1: in_progress
Poll 2: requires_action
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(24).pdf: Contact
www.linkedin.com/in/lauragurski
(LinkedIn)
Top Skills
Management Consulting
Market Entry
Organizational Design
Languages
French
English (Native or Bilingual)
Honors-Awards
Top 25 Most Influential Consultants
Publications
Early Push for Sales Undercuts
Black Friday (quoted)
Data analytics needs CIO/CMO
cooperation
Big Data Not Living Up To Its
Promise? Change the Way You
Work
There's a myth circulating that
millennials don't want to own
Retailers lose billions when crooks
exploit return policies (interview)
Laura Gurski
Lead - Client Portfolio and Office of the Chair and CEO
Oak Park, Illinois, United States
Summary
Laura Gurski leads Accenture's Office of the Chair & CEO. In this
role, Laura is responsible for working with Chair & CEO, Julie
Sweet, in creating and executing her client engagement strategy,
collaborating with ecosystem relationship leaders, representing her
internally and externally, and overseeing all functions of her office. In
addition, Laura has leadership responsibility for client relationships,
including those in the portfolio of the Chair & CEO. She is also a
member of Accenture's Global Management Committee.
Before assuming her current role in 2021, Laura led multiple
functional and global industry practices, primarily in consumer goods
and retail. Most recently, she served as the North America lead of
Accenture's Consumer Goods and Services practice, and previously
was the global lead. In these roles she was responsible for setting
the strategy, managing the executive team, overseeing thought
leadership and helping Accenture clients achieve their business
objectives. Previously, she was the global lead for the Customer
and Channels consulting practice, overseeing the development
and delivery of marketing, customer service, commerce and sales
transformation services to clients in the automotive, consumer
goods, industrial, life sciences, retail and travel industries.
Prior to joining Accenture in 2017, Laura spent more than 20
years at A.T. Kearney, including leading its global services and
industry practices and overseeing 3,500 consultants, reporting
to the Managing Partner. During her tenure she also led the
company's global Consumer Products & Retail practice, where
she focused on driving profitability through growth strategy and
organization transformation. Laura also held senior positions in
brand management, merchant and product marketing before joining
A.T. Kearney.
Laura is a member of the Chicago Network and the Executives' Club
of Chicago. She holds a Bachelor of Science from Northern Illinois
University and an MBA from the Kellogg School of Management at
Northwestern University. She has been featured as a commentator
in leading media outlets such as CNBC, CNN Money, National Public
 Page 1 of 5
https://www.linkedin.com/in/lauragurski?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BMP2a6NovS2%2BtfbcVfLRDIg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/lauragurski?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BMP2a6NovS2%2BtfbcVfLRDIg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Radio and The New York Times. Laura has been recognized by
Consulting Magazine as one of the Top 25 industry consultants.
Experience
Accenture
6 years 1 month
Senior Managing Director
February 2017 - Present (6 years 1 month)
Greater Chicago Area
Lead - Client Portfolio and Office of the Chair & CEO
September 2021 - July 2022 (11 months)
Laura Gurski leads Accenture's Office of the Chair & CEO. In this role, Laura
is responsible for working with Chair & CEO, Julie Sweet, in creating and
executing her client engagement strategy, collaborating with ecosystem
relationship leaders, representing her internally and externally, and overseeing
all functions of her office. In addition, Laura has leadership responsibility for
client relationships, including those in the portfolio of the Chair & CEO. She is
also a member of Accenture's Global Management Committee)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(40).pdf: Contact
www.linkedin.com/in/
davidmichparker (LinkedIn)
Top Skills
Business Process Improvement
Business Transformation
Management Consulting
David Parker
Global Financial Services Industry Practice Chair at Accenture
New York, New York, United States
Summary
I am a trusted CEO and C-Suite advisor and consulting business
leader within the financial services industry. I have 30+ years
of experience at Accenture in shaping and driving business
transformation at some of the world’s largest Financial Services
institutions. I am the Global Chair for Accenture's Financial Services
Industry Practices; in this role I have responsibility for how we
serve our clients and grow our Financial Services business through
the development of Industry Solutions, Partnerships, Thought
Leadership and Talent. I also represent the Financial Services
Industry on Accenture's Global Management Committee.
Experience
Accenture
Senior Managing Director
1992 - Present (31 years)
1992-Present - Senior Managing Director, Global Financial Services Industry
Practices Chair 
• I am the Global Chair for Accenture's Financial Services Industry Practices;
in this role I have responsibility for how we serve our clients and grow our
Financial Services business through the development of Industry Solutions,
Partnerships, Thought Leadership and Talent. I also represent the Financial
Services Industry on Accenture's Global Management Committee
• I was previously the Lead for our Northeast US Financial Services business;
in this role, I was responsible for all aspects of Accenture's business at
Banking, Insurance and Capital Markets clients in the Northeast region in
North America
• I previously managed the relationship with a large Global Banking Group,
oversaw a portfolio of retail and commercial banking clients in the UK and was
also responsible for establishing Accenture’s partnership with a major cloud
platform partner within Financial Services
• Prior roles include Client Account Lead and Service Delivery Executive
 Page 1 of 2
https://www.linkedin.com/in/davidmichparker?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BtL3gJAO%2FSz69JxSqoL1Thw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/davidmichparker?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BtL3gJAO%2FSz69JxSqoL1Thw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Education
University of York
 · (1988 - 1991)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(9).pdf: I started as the Business Development lead
for HP BPO and worked with the Sales Lead for NA in helping and growing
the business with a team supporting Sales and Solutioning. I did this role for
six months and then moved as the Delivery lead for HP BPO clients in India.
Both these roles were pivotal as I had moved to a new industry and learned
 Page 4 of 5
 
the workings of the BPO Industry both from the Sales side and the Delivery
side.
Standard Chartered Bank
Head, Branch Banking, South
October 2001 - November 2005 (4 years 2 months)
Bangalore
I had two roles in the four years with Standard Chartered Bank. I was leading
the Branch Banking Business for the South of India for Standard Chartered
Bank and was responsible for the P&L for the region and managing all clients,
employees with 11 branches. Prior to that I was the Credit cards Service head
for the bank leading all customer services for the Cards book of business. Poth
these roles were pivotal in my career as I grew from a product role to a P&L
role and a large people management role.
Citi India
Product Manager, Citigold
January 1993 - September 2001 (8 years 9 months)
I played different roles with Citibank in the 8 years that I spent with them. I
started as a Customer Services Associate, went on to being a Wealth Manager
and my last role was a Product Manager for the High Networth portfolio for the
bank - CitiGold. I learned the entire Retail Banking Operations and specialized
in Wealth Management.
Education
University of Calcutta
M.B.A, Marketing · (1990 - 1992)
Harvard Business School
Women's Leadership Forum, Organizational Leadership · (2011)
 Page 5 of 5)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(9).pdf: Leading in crisis, managing the health, family and hospital
situations to ensuring the delivery for client operations has changed my
leadership perspective. Today, I am working on the talent strategy for the
workforce of the future keeping in mind that employee needs have changed
significantly and yet client needs remain the same and get increasingly
complex.
Managing Director and Global Transformation Lead, Accenture
Operations 
March 2017 - June 2018 (1 year 4 months)
Bengaluru Area, India
 Page 2 of 5
 
In this role, I was leading our Global Solution Development team, Global
Transitions & Global Capability teams for Accenture Operations. In this role,
my responsibilities were in three different areas of the business. 
In my role leading Solution Development, I was responsible for all solutions
that were being created for our new clients. There were two clear objectives.
Firstly to improve the win rate and our sold margins and secondly to build and
strengthen the overall solution development team by bringing in practitioners.
In this role, my learnings were on end to end crafting transformation solutions
for clients that would enable them meet their business objectives.
In my role as a Global Transition lead - the key responsibility was delivering a
seamless year one experience and upfront transformation.
As the Capability lead, my responsibility was to strengthen our offerings build
improved benchmarking and start building industry depth in the different
offerings.
In all three roles my responsibility was towards driving growth and enhancing
our depth in our offerings and capabilities.
Accenture
Managing Director & Service Delivery Lead, Accenture Operations,
Philippines
March 2013 - 2017 (4 years)
I was responsible for Leading Accenture Operations Delivery in the
Philippines, that was the second largest operations center for Accenture. My
responsibilities were around end to end client experience and transformation,
driving profitable growth ,talent strategy and development and the location
strategy for the Philippines.
One of my key focus areas was to develop local leaders. 
This was a role in which I truly learned operating in a new culture and the
importance of knowing and understanding people and the local culture in-order
to lead. My achievements were in different areas, we grew a new center in
Ilocos, we grew the team by more than 60% .
Mphasis
2 years 10 months
Senior Vice President & Head, M&A, MphasiS.
March 2012 - March 2013 (1 year 1 month)
Bangalore.
In this role I was leading M&A for mPhasiS and working with the CEO & the
CFO on the acquisition strategy. We went through multiple searches and
finally short selected and conducted due diligence, completed valuation and
 Page 3 of 5
 
the entire acquisition process of Digital Risk, a Mortgage analytics and Risk
management company in less than a year.
Senior VP & Head Global Alliance, HP
June 2010 - March 2012 (1 year 10 months)
My role was Vice President and Head of HP Alliance for mPhasis. My key
responsibilities were to strengthen the mPhasis HP relationship, where HP was
the biggest shareholder and the single largest client for mPhasis and I was
tasked to carry out a difficult negotiation and MSA renewal.
This has been one of the most challenging assignments I have had and I have
learned absolutely new skill sets because of this role. I built my negotiation
skills along with the strategic thinking skills in this role and we completed the
MSA renewal successfully.
Target
VP Business Services
December 2007 - June 2010 (2 years 7 months)
Bengaluru, Karnataka, India
My role was Vice President, Business Services for Target India. Target had set
up its own Shared Services in India that was focused primary on Technology. I
joined the company to grow the Business Services ( all the Retail operations )
and I started working with all the Headquarters teams in bringing in functions
like Merchandising. Marketing, Store Operations, Distribution, Finance and
HR. It was an extremely interesting journey of creating teams and building
skills that the company had never done outside the United States. In this role,
I spent a week in a store at Target and another week at a Distribution center
that deepened my knowledge on Store and Distribution Center operations.
This role also helped me truly understand the operations of a Shared Service
center and also learn how to create an extension of the headquarters in an
offshore location.
HP
Director-Operations
January 2006 - November 2007 (1 year 11 months)
I did two roles while I was with HP)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(51).pdf: Contact
www.linkedin.com/in/john-f-walsh
(LinkedIn)
www.accenture.com (Company)
www.accenture.com/us-en/careers
(Other)
vlocity.com/ (Other)
Top Skills
Management Consulting
Business Transformation
Outsourcing
Languages
Spanish
John Walsh
Chief Sales Officer at Accenture
Los Altos, California, United States
Summary
I believe the best leaders have a vision and lead from the front,
inspiring others to follow. This is what I try to do every day as
Accenture’s Chief Sales Officer, where I am responsible for creating
value and driving sales growth with many of the world’s largest
and most innovative companies. In Fiscal Year 2020, we delivered
a record $49.6 billion in new bookings—a double-digit year-over-
year increase. Accenture has always been a momentum business,
powered by the people who work here to deliver the best results for
our clients on a worldwide scale. I am proud to have played a role in
helping to build that momentum.
Prior to my current role, I served as Group Chief Executive Officer
of Accenture’s $10B Global Communications, Media & Technology
(CMT) business which also afforded me the unique opportunity
to work closely with the special group companies who are both
important ecosystem partners and key clients of Accenture. As
a member of Accenture’s Executive and Global Management
Committee, I work closely with our CEO and other senior leadership
to set our strategy by starting with our clients’ needs.
Outside of Accenture I serve on the IFS Board and have served
as director on several boards, including for Vlocity, a cloud unicorn
acquired by Salesforce for $1.3 billion in February 2020, and
Adchemy, acquired by Walmart.com in 2014. I live in Silicon Valley
and am married with four daughters who simultaneously entertain
me, challenge me and annoy me, in a good way. I also enjoy
surfing, skiing and endurance sports.
Experience
Accenture
36 years 6 months
Chief Sales Officer
 Page 1 of 2
https://www.linkedin.com/in/john-f-walsh?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B0GFLJJ%2BgSe2ttL58pOznPQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/john-f-walsh?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B0GFLJJ%2BgSe2ttL58pOznPQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
http://www.accenture.com
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/careers
https://vlocity.com/
 
September 2020 - Present (2 years 6 months)
Accenture is a global professional services company with leading capabilities
in digital, cloud and security. Combining unmatched experience and
specialized skills across more than 40 industries, we offer Strategy and
Consulting, Interactive, Technology and Operations services — all powered by
the world’s largest network of Advanced Technology and Intelligent Operations
centers. Our 674,000 people deliver on the promise of technology and human
ingenuity every day, serving clients in more than 120 countries. We embrace
the power of change to create value and shared success for our clients,
people, shareholders, partners and communities.
Group Chief Executive — Communications, Media & Technology
September 1986 - September 2020 (34 years 1 month)
San Francisco Bay Area
Adchemy
Board Member
July 2013 - Present (9 years 8 months)
Menlo Park California
Churchill Club
Member Board of Directors
2012 - Present (11 years)
San Jose California
Vlocity
Board of Directors
January 2018 - Present (5 years 2 months)
Education
Iowa State University
Business Administration, Marketing, Finance · (1982 - 1986)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(33).pdf: Contatta
www.linkedin.com/in/mauro-macchi
(LinkedIn)
www.accenture.com/us-en/about/
leadership/mauro-macchi (Other)
www.accenture.com/us-en/about/
consulting-index (Company)
fondazioneaccenture.it/ (Other)
Competenze principali
Business Transformation
Digital Strategy
Business Strategy
Languages
English (Full Professional)
French (Professional Working)
Mauro Macchi
CEO Accenture Italy - Market Unit Lead Italy, Central Europe,
Greece
Milano, Lombardia, Italia
Riepilogo
To create business value, you must embrace technology; fully and
actively. As a champion of technology-enabled change, I show
clients how to combine innovation, strategy and Accenture’s end-to-
end capabilities in order to achieve the results they expect and more.
Throughout my 30-year career at Accenture, I relied on this
approach to help some of Europe’s largest banks, execute major
transformation programs, showing them how technology can make
a difference in all facets of banking. This vision was also front and
center when I helped establish Accenture’s Strategy practice in Italy. 
As a member of the board of Accenture Foundation in Italy, I am
proud to share my experiences and knowledge to help nonprofits
develop and accelerate digital transformation programs that will
create positive, sustainable change in people’s lives and contribute
to the well-being of the broader community. 
Developing the human potential is a constant motivator. So, whether
it’s helping my wife choose new contemporary artists to feature
in her gallery or encouraging my teams to try new skills, I believe
creative diversity is the foundation for continued engagement, trust,
and progress. That’s why I passionately advocate for inclusion
and mentorship initiatives and consider them essential for any
organization aiming to build technology capabilities, secure talent,
and drive innovation. 
Looking for the right experts to drive meaningful impact for your
business? Connect with me on LinkedIn.
Esperienza
Accenture
 Page 1 of 3
https://www.linkedin.com/in/mauro-macchi?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B7YWnpxLITJa64sTGz3QAEw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/mauro-macchi?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B7YWnpxLITJa64sTGz3QAEw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com/us-en/about/leadership/mauro-macchi
https://www.accenture.com/us-en/about/leadership/mauro-macchi
https://www.accenture.com/us-en/about/consulting-index
https://www.accenture.com/us-en/about/consulting-index
https://fondazioneaccenture.it/
 
16 anni 2 mesi
CEO Accenture Italy - Market Unit Lead Italy, Central Europe, Greece
settembre 2021 - Present (1 anno 6 mesi)
Milan, Lombardy, Italy
Senior Managing Director - Accenture Strategy & Consulting Lead for
Europe
marzo 2020 - settembre 2021 (1 anno 7 mesi)
Milano, Lombardia, Italia
I lead Strategy & Consulting Services for the European market. In this role,
I'm responsible for setting the overall vision and strategy for our group, for
investment and consulting asset development, for sales and for bringing our
distinctive end-to-end services to our clients across Europe)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(5).pdf: Contact
www.linkedin.com/in/fabio-benasso
(LinkedIn)
Top Skills
Management Consulting
Business Transformation
IT Strategy
Fabio Benasso
Advisor to Accenture Chair & CEO - President Accenture Italy
Milan, Lombardy, Italy
Experience
Accenture Italia
38 years 8 months
Advisor to Accenture Chair & CEO - President Accenture Italy
September 2021 - Present (1 year 6 months)
Milan, Lombardy, Italy
President & CEO Accenture Italy - ICEG GU Senior Managing Director
March 2020 - August 2021 (1 year 6 months)
Milan, Lombardy, Italy
Geographic Unit Senior Managing Director - Italy, Greece, Central &
Estern Europe, Middle East
September 2006 - March 2020 (13 years 7 months)
Milan
Managing Director Products - Italy, Greece, Central & Estern Europe,
Middle East
September 2003 - August 2006 (3 years)
Partner
September 1997 - August 2003 (6 years)
Management Consultant
July 1984 - September 1997 (13 years 3 months)
Management Consulting Division
Confindustria
9 years 10 months
Componente del Gruppo Tecnico Investitori e investimenti esteri - ABIE
August 2020 - Present (2 years 7 months)
Membro del Gruppo Tecnico “Il Digitale per la competitività del Sistema
Industriale”
July 2022 - Present (8 months)
Member of the General Council
May 2013 - Present (9 years 10 months)
 Page 1 of 4
https://www.linkedin.com/in/fabio-benasso?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BGzviRmhhSkyGITBszEpkGg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(44).pdf: Senior Managing Director, Global CEO Transformation: Strategy &
Consulting Lead 
November 2020 - Present (2 years 4 months)
Global
Responsible for:
• Directing a global team that help’s Accenture’s clients and their workforces
drive innovation by transforming CEO and Leadership mindsets, talent, and
cultures
• Driving and evolving Accenture’s CEO agenda and leadership through
research to deliver breakthrough competitive insights to the C-suite
• Delivering Inclusion & Diversity solutions to the C-suite to achieve equity in
their workforce
USA Field Hockey
Independent Board Member
January 2021 - Present (2 years 2 months)
United States
 Page 2 of 4
 
Apple
Vice President Inclusion & Diversity
2017 - June 2020 (3 years)
Cupertino, CA
GRAIL, Inc.
Interim Head Of Human Resources
2017 - 2017 (less than a year)
Menlo Park, California
Deloitte
National Managing Principal
2001 - 2017 (16 years)
United States
Joined Deloitte as a direct entry Principal in New York City and helped develop
the firm’s business and service offerings in the areas of: strategy, leadership,
analytics. talent management, organizational design, M&A, sales force
effectiveness and technology. Served a broad range of international clients
in the areas of financial services, technology, consumer products, retail, life
sciences, government and other sectors. Held managing partner roles in
industry and consulting in the west region of the United States.
The Sausalito Group
Vice President
1998 - 2001 (3 years)
Management Consultant: Strategy, Leadership Development, Organizational
Effectiveness, Business Development.
Obik
Vice President
1995 - August 1998 (3 years)
New Jersey, United States
Start-up company focused on education and corporate organizational
development. Managed the development of software products and clients.
Manchester Partners International
Vice President
1992 - August 1995 (3 years)
Executive Leadership Development.
 Page 3 of 4
 
Education
New York University
Doctor of Philosophy (Ph.D.), Clinical Social Work · (1995)
Rutgers University
Master's Degree, Social Work · (1992)
Loyola University Maryland
Bachelor of Arts - BA · (1986)
 Page 4 of 4)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(11).pdf: Contact
www.linkedin.com/in/
jamescrowley123 (LinkedIn)
www.accenture.com/ (Other)
www.accenture.com/us-en/careers
(Other)
Top Skills
Management Consulting
Business Transformation
Strategy
James Crowley
Global Products Industry Practices Chair | Global Management
Committee | Accenture
Philadelphia, Pennsylvania, United States
Summary
I am the Global Products Industry Practices Chair, with a focus on
seven industries: consumer goods & services, retail, industrial, life
sciences, aerospace & defense, travel & hospitality and automotive. I
am a member of Accenture’s Global Management Committee.
Prior to this role, I led the Industry Networks for North America. In
this role I was responsible for 19 Industries, across all Services and
Market Units. Before that, I served as the Products Group Lead
in the Northeast Market Unit, responsible for all of Accenture's
business at consumer goods & services, retail, industrial, life
sciences, travel & hospitality, and automotive clients. Prior to that, I
led our Strategy business globally for Products. With more than 26
years of experience, I have worked internationally serving clients in
North America, Europe and Growth Markets.
I am passionate about solving challenges and unlocking new
value at the intersection of business and technology. I have
significant experience advising clients on business strategies,
customer strategies, marketing & sales, competitiveness, digital
value realization, innovation, M&A and large-scale business
transformation. 
I graduated from the Wharton School at the University of
Pennsylvania and hold a Bachelor of Science in Economics with
Concentrations in Finance and Strategic Management. I serve on the
Oversight Board for the Libraries at the University of Pennsylvania.
Outside of work you can find me spending time with my family,
playing golf, reading a good book or enjoying time with friends.
Experience
Accenture
 Page 1 of 3
https://www.linkedin.com/in/jamescrowley123?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bs3oo%2B7yBTe6XJduHwwp4mA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/jamescrowley123?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bs3oo%2B7yBTe6XJduHwwp4mA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com/
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/careers
 
26 years 7 months
Global Products Industry Practices Chair | Global Management
Committee | Accenture
August 2022 - Present (7 months)
United States
Senior Managing Director | North America Industry Networks Lead
March 2021 - October 2022 (1 year 8 months)
Philadelphia, Pennsylvania, United States
Senior Managing Director | Northeast Products Business Lead
February 2019 - October 2022 (3 years 9 months)
Philadelphia, Pennsylvania, United States
I lead the Northeast Business within Products at Accenture. In this role, I
have responsibility for all aspects of Accenture's business at Life Sciences,
Consumer, Retail, Travel, Mobility and Industrial clients in the Northeast region
in North America.
Senior Managing Director | Global Products and Life Sciences Strategy
Lead
May 2014 - January 2020 (5 years 9 months)
Philadelphia, Pennsylvania, United States
I lead the Global Life Sciences Industry Lead within Accenture Strategy. In
this role, I focus on helping C-suite executives solve challenges and realize
opportunities at the intersection of business and technology to unlock new
value. I have over 23 years of global consulting experience working with
biopharmaceutical clients across all aspects of their business.
Managing Director | Life Sciences, Portfolio Lead
September 2011 - September 2014 (3 years 1 month)
Philadelphia, United States
I am a portfolio lead in our Life Sciences practice. In this role, I am responsible
for Accenture's relationship and business across a portfolio of global
biopharmaceutical clients)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(8).pdf: Contact
www.linkedin.com/in/
richardpclarkcpa (LinkedIn)
Top Skills
Strategy
Business Transformation
Board of Directors
Certifications
Certified Public Accountant
Richard Clark
Enterprise Transformation & Finance Executive / Board Director /
ESG / Public Company Global Financial Management & Accounting
Boston, Massachusetts, United States
Summary
Enterprise Transformation and Finance Executive with global
expertise in ESG, Finance, Accounting and Investor Relations for
a consulting and services company with $60B+ in revenue. Lead
controllership across 120 countries and play critical role in 130+
M&A transactions/integrations representing $10B. CFO with P&L
responsibility for $4B business unit.
Extensive experience in Digital Transformation, M&A, SEC
Compliance, US GAAP, Compensation Design and Investor
Targeting/Development.
Board Director with expertise in ESG reporting and governance.
Audit Committee Chair and strong experience working with
Finance, Audit and Compensation Committees. Advise on strategy,
compliance and risk. Skilled building a proxy board, defining areas
of responsibility, governance and risk management, and redesigning
executive compensation.
Advocate for Diversity, Equity & Inclusion with a focus on gender and
LGBT.
Experience
Accenture
39 years
Chief Transformation Officer
January 2021 - Present (2 years 2 months)
- Lead comprehensive digital transformation across all areas of the business
to enable an integrated data-driven global enterprise; member of Global
Management Committee reporting to CEO
Chief Accounting Officer
January 2013 - September 2022 (9 years 9 months)
 Page 1 of 3
https://www.linkedin.com/in/richardpclarkcpa?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BT9S3ueMmRAa6b7insQo9%2FQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/richardpclarkcpa?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BT9S3ueMmRAa6b7insQo9%2FQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
- Spearhead development and implementation of ESG processes, governance
and metrics across all areas of operations; introduce integrated reporting for
ESG
- Advise on complex transactions and lead integration of 130 acquisitions,
$10B of capital and two carve-out audits for divestitures
- Direct Global Controllership team of 1,700 across representing 120 countries
- Oversight of SEC Compliance, Annual Audit, Annual Report and Proxy
Filings as well as accounting functions, policies and reporting
Corporate Controller
2010 - 2021 (11 years)
- Led Global Controllership overseeing accounting policy and US GAAP
compliance, systems and internal controls
- Partnered with HR to design $3B compensation program for company’s top
5,000 leaders globally
- Managed acquisition and divestiture accounting and valuation
- Financial oversight for India and the Philippines (2014 - 2021) managing
financial operations, planning and analysis, tax and cost management for the
largest regions (300,000 employees)
Senior Managing Director, Investor Relations
2006 - 2010 (4 years)
Directed communications with shareholders, financial community and
shareholder governance groups during time when stock price increased and
CAGR increased 13%.
Finance Director, Communications and High Tech
2001 - 2006 (5 years)
CFO with P&L oversight of $4B global business unit (largest in company);
achieved 7% revenue growth; member of senior leadership team driving global
strategy.
Global finance roles in Communication, Chemicals, Natural Resources,
Energy & Utilities
1994 - 2001 (7 years)
Director of Practice Management, Human Resources & Operations
1991 - 1994 (3 years)
Public accounting roles in Audit & Tax Consulting (Arthur Andersen,
LLP)
1984 - 1991 (7 years)
 Page 2 of 3
 
Accenture Federal Services
11 years
Board Observer
2017 - Present (6 years)
Attend Board and Audit Committee meetings at request of proxy board)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(20).pdf: Contact
www.linkedin.com/in/leonardoframil
(LinkedIn)
Top Skills
Management Consulting
Business Process Design
IT Strategy
Languages
English (Native or Bilingual)
Spanish (Professional Working)
Portuguese (Native or Bilingual)
Leonardo Framil
CEO – Accenture, Growth Markets
São Paulo e Região
Summary
First and foremost a dedicated father (to Julia) and husband (to
Alessandra), I work around barriers in navigating three very different
worlds – the geographic, the exclusive, and the inclusive. 
As Accenture's CEO for Growth Markets, spanning Asia Pacific,
Africa, the Middle East, and Latin America, I work with our 430,000
people who are dedicated to the mission of transforming companies,
developing talent, and employing technology, knowledge, and
creativity to create more value for our clients, people, shareholders,
partners and communities.
I also live this purpose with the spirit of someone who fell in love with
this company 30 years ago and has worked on the evolution of large
financial institutions and the transformation of corporations in Asia,
Europe, and North America.
I have shared my passion for the Accenture culture with more
than ten companies acquired in Latin America the past six years
and thousands of new people joining us. The Accenture I first
encountered, with 1,000 employees in Latin America, now has
44,000 people in the region, nearly half of them women.
Every day this company opens up new worlds with different cultures,
realities, and profound professional and humanitarian challenges. I
serve on the American Chamber of Commerce board and councils of
entities dedicated to technology. In parallel, I devote much energy to
working with my peers, entrepreneurs, executives, and CEOs of non-
governmental entities interested in fighting inequality.
I am obsessed with learning and grateful for what they taught at
Universidade Federal Fluminense (UFF), the Brazilian Institute of
Capital Markets (IBMEC), Insead, IMD, Singularity University, and
the University of Chicago.
 Page 1 of 2
https://www.linkedin.com/in/leonardoframil?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BBaYOMwVnQ1GhFdGqIdz2oQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/leonardoframil?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BBaYOMwVnQ1GhFdGqIdz2oQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Beyond what I learn in my role leading Growth Markets and as a
member of Accenture’s Global Management Committee, I absorb
knowledge living life in the offices, favelas, and at home with the two
women who inspire me daily.
Experience
Accenture
Chief Executive Officer, Growth Markets – Asia Pacific, Africa, Middle
East, Latin America
September 2022 - Present (6 months)
Accenture Brasil
24 years
CEO - Brazil and Latin America
January 2016 - September 2022 (6 years 9 months)
São Paulo Area, Brazil
Senior Managing Director - Latam Geo Lead, Financial Services
October 1998 - January 2016 (17 years 4 months)
São Paulo Area, Brazil
Banco BBM
Product Manager
April 1997 - October 1998 (1 year 7 months)
Accenture Brasil
Consultant
March 1992 - March 1997 (5 years 1 month)
São Paulo Area, Brazil
Education
Universidade Federal Fluminense
Engenharia de telecomunicações, Engenharia de
Telecomunicações · (1986 - 1991)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(66).pdf: Contact
www.linkedin.com/in/
christopheryoung4 (LinkedIn)
Top Skills
SaaS
Product Development
Management
Christopher Young
Executive Vice President Business Development, Strategy and
Ventures at Microsoft
Santa Clara, California, United States
Experience
Microsoft
Executive Vice President Business Development
November 2020 - Present (2 years 4 months)
San Francisco Bay Area
American Express
Member Board Of Directors
April 2018 - Present (4 years 11 months)
Snap Inc.
Member Board Of Directors
October 2016 - November 2020 (4 years 2 months)
McAfee
CEO
April 2017 - February 2020 (2 years 11 months)
San Francisco Bay Area
Intel Corporation
SVP and GM, Intel Security Group
October 2014 - April 2017 (2 years 7 months)
Santa Clara, CA
Intel Security protects the cloud and smart, connected computing devices
worldwide. The organization also executes the world’s most advanced
malware research and curates an unrivaled repository of threat intelligence.
Ninety percent of Fortune 100 firms and two-thirds of the Global 2000 rely on
Intel Security technology to protect mission-critical digital systems and data.
More than 175 million consumers use Intel Security products to keep their
families safe online. 
Rapid7
Board Director
January 2011 - August 2016 (5 years 8 months)
 Page 1 of 2
https://www.linkedin.com/in/christopheryoung4?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bim%2FazkNWS7W0LoPzDX%2BkWA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/christopheryoung4?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bim%2FazkNWS7W0LoPzDX%2BkWA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Cisco
SVP, Security & Government Group
2011 - September 2014 (3 years)
VMware
SVP & GM End User Computing
2010 - 2011 (1 year)
RSA, The Security Division of EMC
SVP, RSA Products
December 2004 - November 2010 (6 years)
AOL
Vice President, Safety & Security Premium Services
2003 - 2004 (1 year)
Cyveillance
President, COO & Co-founder
March 1997 - January 2000 (2 years 11 months)
Mercer Management Consulting
Associate
1994 - 1997 (3 years)
Education
Harvard Business School
Princeton University
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(52).pdf: Contact
www.linkedin.com/in/mahesh-
zurale-a6860a1 (LinkedIn)
Top Skills
IT Strategy
Solution Architecture
Software Project Management
Mahesh Zurale
Senior Managing Director, Lead – Accenture’s Advanced Technology
Centers in India (ATCI)
India
Summary
Mahesh Zurale is a senior technology leader at Accenture, with
nearly 30 years of experience in the global IT industry. In his current
role as the lead for Accenture’s Advanced Technology Centers
in India (ATCI), he has the overall responsibility for technology
delivery and driving the innovation at scale agenda at ATCI. He is
also responsible for talent development, propagating a culture of
innovation, and championing the inclusion and diversity agenda
across ATCI.
Mahesh joined Accenture in 2006 as the delivery lead for one of
Accenture’s largest Healthcare clients. In 2010, he became the
global solution architecture lead for Technology, responsible for
driving profitable growth through highly differentiated and competitive
solutions. He has been working with sales, delivery, platform and
offering teams to create offerings as well as solution strategies that
deliver superior value to Accenture’s clients. 
Prior to Accenture, he was the Chief Operating Officer at Datamatics
Technologies Ltd. He started his career as an entrepreneur, co-
founding FuturisTech Systems, a product company focussed on
developing solutions for Government and corporate clients.
Mahesh is a passionate leader focussed on driving the inclusion
and diversity agenda across ATCI, and supports Accenture's goal to
achieve gender-balanced workforce globally by 2025. 
He completed his Bachelor of Engineering in Electrical Engineering
from the University of Mumbai, India, and holds Master of Science in
Computer Engineering as well as Master of Business Administration
from The University of Texas at Austin. He lives in Pune, India, with
his wife and two daughters.
 Page 1 of 2
https://www.linkedin.com/in/mahesh-zurale-a6860a1?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BImd07SsXT12KrjVFnM8%2Fnw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/mahesh-zurale-a6860a1?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BImd07SsXT12KrjVFnM8%2Fnw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Experience
Accenture
Senior Managing Director, Lead – Accenture’s Advanced Technology
Centers in India (ATCI)
May 2006 - Present (16 years 10 months)
Pune
I lead the Accenture Advanced Technology Centers in India where our
incredibly talented people build highly innovative technology solutions for
world’s leading companies every day! Prior to my current role, I was the Global
Solution Architecture Lead for Accenture Technology.
Datamatics Technologies Ltd.
COO
September 2004 - May 2006 (1 year 9 months)
University of Texas at Austin
Student
1987 - 1991 (4 years)
Education
Texas McCombs School of Business
MBA · (1989 - 1991)
The University of Texas at Austin
MS, Computer Engineering · (1987 - 1989)
VJTI
BE, Electrical Engineering · (1983 - 1987)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(64).pdf: Contact
www.linkedin.com/in/satyanadella
(LinkedIn)
www.microsoft.com/ceo
(Company)
Satya Nadella
Chairman and CEO at Microsoft
Redmond, Washington, United States
Summary
As chairman and CEO of Microsoft, I define my mission and that of
my company as empowering every person and every organization on
the planet to achieve more.
Experience
Microsoft
Chairman and CEO
February 2014 - Present (9 years 1 month)
The Business Council U.S.
Chairman
2021 - Present (2 years)
University of Chicago
Member Board Of Trustees
2018 - Present (5 years)
Starbucks
Board Member
2017 - Present (6 years)
Fred Hutch
Board Member
2016 - 2022 (6 years)
Education
The University of Chicago Booth School of Business
 · (1994 - 1996)
Manipal Institute of Technology, Manipal
Bachelor’s Degree, Electrical Engineering
 Page 1 of 2
https://www.linkedin.com/in/satyanadella?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BSfYLNaXNRZqNStacPYr1Lw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/satyanadella?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BSfYLNaXNRZqNStacPYr1Lw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
http://www.microsoft.com/ceo
http://www.microsoft.com/ceo
 
University of Wisconsin-Milwaukee
Master’s Degree, Computer Science
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(24).pdf: Before assuming her current role in 2021, Laura led multiple functional and
global industry practices, primarily in consumer goods and retail. Most recently,
she served as the North America lead of Accenture's Consumer Goods and
Services practice, and previously was the global lead. In these roles she was
responsible for setting the strategy, managing the executive team, overseeing
thought leadership and helping Accenture clients achieve their business
objectives. Previously, she was the global lead for the Customer and Channels
consulting practice, overseeing the development and delivery of marketing,
customer service, commerce and sales transformation services to clients in
the automotive, consumer goods, industrial, life sciences, retail and travel
industries.
Prior to joining Accenture in 2017, Laura spent more than 20 years at A.T.
Kearney, including leading its global services and industry practices and
overseeing 3,500 consultants, reporting to the Managing Partner. During
her tenure she also led the company's global Consumer Products & Retail
practice, where she focused on driving profitability through growth strategy
and organization transformation. Laura also held senior positions in brand
management, merchant and product marketing before joining A.T. Kearney.
Senior Managing Director
 Page 2 of 5
 
August 2018 - August 2021 (3 years 1 month)
Laura Gurski is a Senior Managing Director with Accenture. She was the
Global and NA Market industry practices leader for Accenture’s Consumer
Goods and Services practice. She has responsibility for setting the strategy for
the industry, overseeing the thought leadership and serving Accenture clients. 
Her experience comprises 25 years serving global companies to drive
transformation to enable growth. Her primary focus has been in consumer
packaged goods (fast moving and durable) and retail industries. She serves
the top global companies.
Senior Managing Director Global Customer & Channels
February 2017 - August 2018 (1 year 7 months)
Laura Gurski lead the Customer and Channels consulting practice for the
automotive, consumer goods, industrial, life sciences, retail and travel
industries. 
As the head of Accenture’s Customer and Channels Consulting Practice for
Products, Ms. Gurski oversaw the development and delivery of marketing,
customer service, commerce and sales transformation services. 
She helped clients define their digital strategies, make insight-driven decisions
and deliver the right experience, the right content, the right brand and the right
products to their customers, in a new era of hyper-personalization. 
GP Strategies Corporation
Board Member
April 2015 - February 2017 (1 year 11 months)
Columbia, Maryland
Laura served on an outside Board for GP Strategies as a Director and member
of the Audit Committee. GP Strategies (NYSE: GPX) is a global performance
improvement company committed to providing flexible, customized learning
and development solutions that align with business, cultural and regional
requirements. 
A.T. Kearney
22 years
Partner
1995 - January 2017 (22 years)
 Page 3 of 5
 
• Build and manage advisory relationships with C-level executives and
leadership teams across Fortune 200 client organizations with a greater
focus in consumer industries and retail. Specific expertise in market entry,
corporate strategy, marketing effectiveness, organization transformation and
cost reduction
• Serve as senior client relationship partner and lead large client pursuits for
multiple global and north American Consumer and Retail clients. 
Some notable engagements:
• Advised CEO with the development and implementation of a broad based
organization transformation for a global retailer (US$50B) to align capabilities
and investments to new strategy. 
• Served as the CFO advisor of a global agri-business to re-align the finance,
IT and HR organizations and implement outsourcing. 
• Advised CEO and COO of large national retailer to structure a significant cost
reduction transformation including COGs, SG&A, procurement and real estate
yielding 10-12% cost improvement.
• Other notable areas for global companies include global commercial
management/pricing for consumer goods, M&A due diligence and merger
integration across industries, organization transformation and market entry.
Partner and Global Practice Lead
January 2014 - January 2016 (2 years 1 month)
Laura Gurski was a partner with the global management consulting firm, A.T.
Kearney. She was the global practices leader, where she has responsibility
for the firm’s global industry and service practices)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(21).pdf: Contact
www.linkedin.com/in/
drbhaskarghosh (LinkedIn)
www.accenture.com/us-en/
about/leadership/bhaskar-ghosh
(Company)
Top Skills
Change Management
Business Process Improvement
Business Analysis
Bhaskar Ghosh
Chief Strategy Officer at Accenture
Bengaluru, Karnataka, India
Summary
Dr. Bhaskar Ghosh is Accenture’s chief strategy officer, responsible
for the company’s strategy and investments, including ventures
and acquisitions. He also oversees the development of all assets
and offerings across Accenture’s Services. In addition, Bhaskar
is management sponsor for Accenture’s Responsible Business &
Sustainability Services and Industry X business, which includes
digital manufacturing and intelligent products and platforms.. He is a
member of Accenture’s Global Management Committee.
Bhaskar most recently served as advisor to Julie Sweet, Accenture’s
chief executive officer, on critical areas including growth and
investment strategy.
Previously, Bhaskar was group CEO of Accenture Technology
Services, directing strategy and investments, and leading platforms,
products and global technology delivery. In this role, he focused
on enabling enterprises to drive growth through innovative
technology services, especially through the adoption of New IT,
spanning strategy, technologies, architectures, platforms, methods,
organization and operating models.
Under Bhaskar’s leadership, Accenture Technology Services made
outstanding progress to rapidly rotate to the New and enable clients
to lead in the New. Over 200,000 Accenture Technology people have
been trained around the world in New IT, including automation, Agile
development and intelligent platforms, and Accenture strengthened
its offerings around data, cloud, security and Liquid Application
Management. He has personally provided leadership to cutting-
edge technology solutions, including the technology automation
platform Accenture myWizard®, Accenture’s cloud assessment and
migration platform - MyNav, and the value-led ERP implementation
platform - Accenture myConcerto. As an innovator, Bhaskar has
been awarded six patents in the area of software engineering and
platform development.
 Page 1 of 3
https://www.linkedin.com/in/drbhaskarghosh?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BmLVAzs%2BBSBSbia3ybrY3Tw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/drbhaskarghosh?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BmLVAzs%2BBSBSbia3ybrY3Tw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com/us-en/about/leadership/bhaskar-ghosh
https://www.accenture.com/us-en/about/leadership/bhaskar-ghosh
https://www.accenture.com/us-en/about/leadership/bhaskar-ghosh
 
Bhaskar serves as an Independent Director on the Board of
Directors of Housing Development Finance Corporation Limited
(HDFC) and is Chairman of its IT Strategy committee and a member
of its Risk, Audit & Governance committee.
Before joining Accenture in 2003, Bhaskar was vice president and
global head of IT infrastructure management services at Infosys.
Earlier in his career, he held several senior positions with Philips in
India, in their consumer electronics business. 
Bhaskar has a bachelor of science degree and a master’s degree
in business administration from Calcutta University and a Ph.D. in
business management from Utkal University in India. 
Bhaskar is currently based in Bangalore, India.
Experience
Accenture
19 years 3 months
Chief Strategy Officer
October 2020 - Present (2 years 5 months)
Bengaluru, Karnataka, India
Dr. Bhaskar Ghosh is Accenture’s chief strategy officer, responsible for the
company’s strategy and investments, including ventures and acquisitions)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(32).pdf: Contact
www.linkedin.com/in/peter-lacy
(LinkedIn)
Top Skills
Management Consulting
Sustainability
Strategy
Languages
French (Limited Working)
English (Native or Bilingual)
Mandarin (Elementary)
Spanish (Professional Working)
Publications
GSK, Accenture & Oxford Climate
Change & Health - Framing the
Issue
Towards a New Era of Sustainability
in the Energy Industry - CEO Study
Series
A New Era of Sustainability in the
Automotive Industry - CEO Study
Series
UN Global Compact - Accenture
CEO Study "A New Era of
Sustainability"
Barclays & Accenture Carbon
Capital - Financing the Low Carbon
Economy
Peter Lacy
Accenture Global Sustainability Services Lead; Chief Responsibility
Officer, Global Mgmt Committee Member, plus Non-Exec at
AeroFarms & WEF YGL Board. Exec Sponsor WEF.
United Kingdom
Summary
Accenture is the world’s largest integrated consulting and technology
services firm listed on the New York Stock Exchange (ACN) with
674,000 people operating in 120 countries. With a market cap over
$200 billion & revenues in excess of $50 billion. As a leadership
team we focus on blending human ingenuity & technology to drive
positive change and 360 value for shareholders and stakeholders
www.accenture.com 
My career at Accenture and beyond has been focused on value
and impact for clients through growth and innovative strategy,
sustainability, technology and solving real world problems at
speed and scale. I currently lead all client services, external
activities as well as all internal activities on Responsible Business
and Sustainability. Lead for global relationships such as WEF,
United Nations etc. I also lead Accenture's technology ecosystem
partnerships with partners such as SAP, Salesforce, Microsoft,
Arabesque etc in this area. 
I am a non-executive director / board member of Aerofarms an
award-winning vertical farming, sustainability and technology
platform and urban agriculture business. Non-executive director /
board member of the WEF's Young Global Leaders Forum. Advisory
board member of various not for profits including Elevate, a charity
working to create education and entrepreneurial opportunities for
young women who are refugees from conflict zones; and, Thirst, a
charity working to raise awareness and provide education on global
water scarcity challenges and solutions. 
I am a twice best selling co-author in 9 languages with two books
on sustainability, strategy, new business models and disruptive
technologies. “Waste to Wealth” in 2015 - https://www.amazon.com/
Waste-Wealth-Circular-Economy-Advantage/dp/1349580392 & “The
 Page 1 of 4
https://www.linkedin.com/in/peter-lacy?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bg3mk4R6cR0iKUeu9TDlywQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/peter-lacy?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Bg3mk4R6cR0iKUeu9TDlywQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Circular Economy Handbook” in 2020 https://www.amazon.co.uk/
Circular-Economy-Handbook-Realizing-Advantage/dp/1349959677 
As well as my leadership roles I’ve spent my career focused on two
things. Growth (inorganic and organic), innovation and disruptive
technologies. And sustainability/responsible business. With everyone
from the UN Secretary General, prime ministers and the boards,
Chairs and CEOs of the worlds largest companies to grass roots not
for profits. 
Currently based in the UK. Previously lived & worked in Shanghai,
Brussels and San Jose)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(43).pdf: Contact
www.linkedin.com/in/ellyn-
shook-1b51127 (LinkedIn)
Top Skills
Organizational Design
HR Transformation
Talent Management
Ellyn Shook
Creating the most truly human company in the digital age.
New York, New York, United States
Summary
Ellyn Shook is Accenture’s chief leadership & human resources
officer, responsible for helping the 699,000 people of Accenture
succeed both professionally and personally. Her global team of HR
leaders is re-imagining leadership and talent practices to create
the most truly human work environment in the digital age, fueling
Accenture’s ability to live its purpose: to deliver on the promise of
technology and human ingenuity. 
A member of Accenture’s Global Management Committee and
Investment Committee, Ellyn is a strong advocate for inclusion and
diversity and Accenture has been widely recognized externally as
an employer of choice and for their equality efforts. As an author on
the topics of future of work, inclusion and diversity and responsible
leadership, Ellyn frequently advises Accenture’s clients on these
topics and on leading large-scale transformational change based on
the talent and culture transformation she’s led at Accenture. 
Ellyn is a member of the World Economic Forum’s Global Shaper
Community Foundation Board and serves on the board of directors
of BRP Group, Paradigm for Parity and the HR Policy Association.
She also serves on the executive committee of the Professional
Roundtable of CHROs and is active in both the NY Jobs CEO
Council and HR50. Ellyn was named HR Executive of the Year by
Human Resource Executive in 2020 and inducted as a Fellow in
the National Academy of Human Resources in 2021. She holds a
Bachelor of Science degree from Purdue University.
Experience
Accenture
Chief Leadership & Human Resources Officer
March 2014 - Present (9 years)
New York 
Ellyn Shook is Accenture’s chief leadership & human resources officer,
responsible for helping the 674,000 people of Accenture succeed both
 Page 1 of 3
https://www.linkedin.com/in/ellyn-shook-1b51127?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BNL128OsXTm2zWfyIzVvsRg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/ellyn-shook-1b51127?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BNL128OsXTm2zWfyIzVvsRg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
professionally and personally. Her global team of HR leaders is re-imagining
leadership and talent practices to create the most truly human work
environment in the digital age, fueling Accenture’s ability to live its purpose: to
deliver on the promise of technology and human ingenuity. 
A member of Accenture’s Global Management Committee and Investment
Committee, Ellyn is a strong advocate for inclusion and diversity and
Accenture has been widely recognized externally as an employer of choice
and for their equality efforts. As an author on the topics of future of work,
inclusion and diversity and responsible leadership, Ellyn frequently advises
Accenture’s clients on these topics and on leading large-scale transformational
change based on the talent and culture transformation she’s led at Accenture. 
Ellyn is a member of the World Economic Forum’s Global Shaper Community
Foundation Board and serves on the board of directors of BRP Group,
Paradigm for Parity and the HR Policy Association. She also serves on the
executive committee of the Professional Roundtable of CHROs and is active in
both the NY Jobs CEO Council and HR50. Ellyn was named HR Executive of
the Year by Human Resource Executive in 2020 and inducted as a Fellow in
the National Academy of Human Resources in 2021. She holds a Bachelor of
Science degree from Purdue University)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(21).pdf: He
also oversees the development of all assets and offerings across Accenture’s
Services. In addition, Bhaskar is management sponsor for Accenture’s
Responsible Business & Sustainability Services and Industry X business,
which includes digital manufacturing and intelligent products and platforms..
He is a member of Accenture’s Global Management Committee.
Advisor to the CEO
March 2020 - October 2020 (8 months)
India
Group Chief Executive - Technology Services; Accenture
September 2014 - March 2020 (5 years 7 months)
Bangalore -India
 Page 2 of 3
 
Bhaskar Ghosh is group chief executive of Accenture Technology Services
with overall responsibility for the Accenture Application Services business,
spanning systems integration and application outsourcing. Additionally, he
leads platforms, products and global technology delivery. He is a member of
the Accenture Global Management Committee. 
Previously, Dr. Ghosh led the Accenture Delivery Centers for Technology
in India, and he was the global application outsourcing lead. He was also
instrumental in building and managing the Accenture IT infrastructure service
delivery capabilities within the Delivery Centers for Technology.
Before joining Accenture in 2003, Dr. Ghosh was vice president and global
head of IT infrastructure management services at Infosys. Earlier in his
career, he held several senior positions with Philips in India, where he worked
extensively with the consumer electronics industry.
Dr. Ghosh is currently based in Bangalore, India. He holds a bachelor of
science degree from Calcutta University, a master’s degree in business
administration from Calcutta University and a Ph.D. in business process
management from Utkal University in India.
Lead - India Delivery Center
December 2003 - August 2014 (10 years 9 months)
Infosys Technologies Ltd
Vice President - IT Infrastructure Outsourcing
1997 - December 2003 (6 years)
Philips
Reg. automation Lead
1986 - 1997 (11 years)
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(9).pdf: Contact
www.linkedin.com/in/arundhati-
chakraborty-2854955 (LinkedIn)
Top Skills
Outsourcing
BPO
Banking
Arundhati Chakraborty
Senior Managing Director and Global Delivery & Business
Transformation Lead, Accenture Operations
Bengaluru, Karnataka, India
Summary
I am a Senior Managing Director with Accenture leading our Global
Delivery and Business Transformation for Accenture Operations.
I am responsible for leading all our 180,000 people across 226
centers in 33 countries and am a member of Accenture’s Global
Leadership committee.
My role rotates around four aspects – client experience across
scaled operations, transformation, talent and running a profitable
business. Ensuring delivery of extremely complex transitions, driving
compressed transformation by using analytics, AI and disruptive
automation and ensuring that we always have strong differentiated
talent is what I am responsible for. 
Prior to Accenture, I have held very diverse roles that have built
me into who I am today. I have worked in the United States, in
the Philippines and in India, and having led Operations across the
Globe, I have a deep understanding of global and local influences
in leading global businesses and challenges . I also have a great
understanding of building a Shared Service as I was in the pioneer
leadership team of Target as they started expanding beyond
Technology into Business Services. I led the Business Services team
and started building Retail Operations in India for Target. Having
worked in a Shared Service and a Third party environment I have
learned and appreciated the differences in both.
Amongst my other significant experiences, I have led Mergers &
Acquisition for mPhasiS and have successfully made an acquisition
of Digital Risk, a Mortgage analytics and platform company. In
this experience I learned tremendously from the different start ups
we engaged with and also leading and learning the end to end
acquisition process.
My other huge learning was in my role as the HP alliance lead for
mPhasis, where I was based in Plano, Texas handling a complex
alliance and building client relationship during a tough negotiation.
In the foundational part of career I have spent the first 13 years of
my career in Retail Banking Operations with Citibank and Standard
 Page 1 of 5
https://www.linkedin.com/in/arundhati-chakraborty-2854955?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BcTsLMzfPRte63Khd3fjgGw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/arundhati-chakraborty-2854955?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BcTsLMzfPRte63Khd3fjgGw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Chartered Bank where I have specialized in Wealth Management,
Credit Card Operations, Banking Operations and Financial Products
like Mutual Funds and Insurance.
I am passionate about leadership development, about learning, and
on growing women leaders in the workplace. I am also a strong
supporter of empowering women and educating children and have
been helping in these causes over the years.
Experience
Accenture
6 years
Senior Managing Director and Global Delivery Lead, Business Process
Services, Accenture
June 2018 - Present (4 years 9 months)
Bangalore
I lead Global Delivery and Business Transformation for Accenture Operations.
I am responsible for leading 180,000 people across 226 centers in 33
countries and all clients and am a member of Accenture’s Global Leadership
committee.
My role rotates around four aspects – client experience across scaled
operations, transformation, talent and running a profitable business. Ensuring
delivery of extremely complex transitions, driving compressed transformation
by using analytics, AI and disruptive automation and ensuring that we always
have strong differentiated talent is what I am responsible for.
In this role, I have also had the opportunity to lead and learn in the biggest
disruption of our lifetime during Covid. I was responsible for leading the
transition of the entire global operation of this scale from office to home with
zero disruptions)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(4).pdf: Contact
www.linkedin.com/in/arjun-bedi-
ba6251 (LinkedIn)
Top Skills
Business Strategy
Cross-functional Team Leadership
Strategy
Arjun Bedi
Strategic Clients Portfolio Lead I Transformation Catalyst I Global
Management Committee
Greater Philadelphia
Summary
As a member of Accenture’s Global Management Committee and
Chair of Accenture’s Strategic Client Portfolio (250 strategic clients
that drive the majority of Accenture’s revenues) – I advise and lead
the strategy and execution needed to accelerate growth and deepen
client partnerships. 
I am known for:
• Operational and P&L leadership – building and driving high growth
businesses 
• Activating digital/technology innovation at scale
• CEO/C-Suite Advisory - Strategy and Execution
• Attracting and growing leaders that deliver exceptional results
For 25+ years I have led large business areas for Accenture,
including creating new business segments, with progressively
increasing P&L responsibilities.
My passion for the Life Sciences industry started 30+ years ago.
Since then I have led technology transformations at 14 Health & Life
Sciences companies around the world – helping leaders embrace
technology as the catalyst for achieving transformation at scale to
create a differentiated competitive advantage and sustained out-
performance.
I am inspired by the power of collaboration - now in its tenth year,
I founded, and currently lead, Accenture’s Health & Life Sciences
CEO and Board Forum, bringing together top health and life
sciences CEOs and Board members for highly valued conversations
on industry challenges and strategies and execution approaches for
success.
Outside of work you will find me enjoying my two Vizslas, playing
or watching golf, enjoying views from my pilot seat (when the
 Page 1 of 3
https://www.linkedin.com/in/arjun-bedi-ba6251?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BIueZciaFQnOFWWe0aE8qIg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/arjun-bedi-ba6251?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BIueZciaFQnOFWWe0aE8qIg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
opportunity allows), and most importantly spending time with the
three most important women in my life – my wife and two amazing
daughters.
Experience
Accenture
33 years 7 months
Senior Managing Director
March 2018 - Present (5 years)
Chairman, Diamond Client Leadership Council and Member of Global
Management Committee 
Since March 2018, Arjun has been Chairman of Accenture’s Diamond
Client Leadership Council, our portfolio of Top 200 global strategic clients,
responsible for a significant part of our firms revenues. Reporting directly to our
CEO, and working closely with our Global Management Committee, his focus
is to develop and execute strategies to build a resilient portfolio of strategic
clients that delivers high impact to clients along with sustained accretive
profitable growth for the firm. 
Arjun was named to Accenture’s Global Management Committee (GMC)
in March 2020, the global governance body, led by Accenture’s CEO,
responsible for development and execution of Accenture’s strategy.
Senior Managing Director
December 2016 - Present (6 years 3 months)
Life Sciences Client Portfolio Lead - Lead P&L accountability for significant
segment of our Life Sciences business.
Senior Managing Director
June 2016 - December 2018 (2 years 7 months)
Growth Strategy & Execution Lead for Products (NA).
Responsible for developing and executing Growth Strategy for our Products
business in North America. Products includes Life Sciences, Consumer
Products, Industrial, Travel and Retail industry groups and is the largest
Operating Group within Accenture)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(30).pdf: I lead a global team of talented consultants,
engineers, and strategists, helping electricity, gas and water companies
transform to generate new value and growth. 
SENIOR MANAGING DIRECTOR – GLOBAL TRANSMISSION &
DISTRIBUTION LEAD, ACCENTURE
2016/September – 2020/February
I led Accenture’s global Transmission & Distribution practice, helping some
of the world’s largest and most innovative utilities with energy transition, to
redefine their growth strategies, implement smart grid strategies, and design
and execute largescale business transformation. 
Key accomplishments in this role include founding and leading Accenture’s
Smart Grid Leadership Network, a peer-to-peer community of over 70 utility
executives from over 40 companies deploying smart grid initiatives. I also led
Accenture’s diversity networks in Europe, Africa & Latin America.
Partner
1996 - Present (27 years)
Education
The Ohio State University
 Page 2 of 3
 
Master's Degree, Industrial & Systems Engineering · (1994 - 1996)
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(28).pdf: Contact
www.linkedin.com/in/kate-hogan
(LinkedIn)
www.accenture.com/us-en/cmt-
communications (Other)
www.accenture.com/us-en/careers
(Other)
www.accenture.com/us-en/
company-diversity (Other)
Top Skills
Business Transformation
Business Process Improvement
Program Management
Kate Hogan
Grow high value services with incredible clients | COO N. America
San Francisco Bay Area
Summary
I enjoy maximizing growth from investments and removing
operational friction. Every day I am excited to work with teams to win
large and complex deals with high quality deliverable solutions our
clients can count on. Data instigates change and my team and I use
insights to implement new processes and improve Accenture's sales,
pricing, competitive intelligence and delivery quality. 
Prior to this role, I spent 15+ years serving clients in
Communications, Media and Technology (CMT) industries. In
that capacity, I was responsible for applying technology and
new processes to dramatically improve customer and partner
experiences. I enjoyed achieving these outcomes in unique
commercial arrangements, while also achieving external recognition
on improved satisfaction. Developing client trust was critical in my
tenure as a Client Account Leader where my team profitably grew
the account to Diamond status.
My passion for technology started at an early age with a personal
a love for computer games and calculators. Today, I live in Silicon
Valley where my family and I are often found trying out new tech,
hiking with our yellow lab, enjoying live sports, or sharing something
from garden over dinner with friends. 
Experience
Accenture
25 years
COO - North America 
March 2020 - Present (3 years)
San Francisco Bay Area
Senior Managing Director of Operations, Communications High Tech &
Media 
1998 - March 2021 (23 years)
 Page 1 of 2
https://www.linkedin.com/in/kate-hogan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BxX63WOG7QDiq0SUzHCv9Ag%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/kate-hogan?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BxX63WOG7QDiq0SUzHCv9Ag%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com/us-en/cmt-communications
https://www.accenture.com/us-en/cmt-communications
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/company-diversity
https://www.accenture.com/us-en/company-diversity
 
San Francisco Bay Area
Education
The University of Kansas
Bachelor of Science, Business (Finance)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(3).pdf: Contact
www.linkedin.com/in/jack-azagury
(LinkedIn)
www.accenture.com (Other)
www.accenture.com/us-en/careers
(Other)
www.accenture.com/us-en/about/
leadership/leadership-index (Other)
Top Skills
Business Strategy
Mergers
Smart Grid
Languages
French (Full Professional)
Spanish (Professional Working)
English (Native or Bilingual)
Publications
Building the Next Generation Utility
From oil producers to power players:
A smart move?
Are utilities underestimating the
impact of new technologies & new
entrants?
A balanced Energy Transition:
sustainability & security
Smart Transition - Embracing a
competitive and digital future for
utilities
Patents
Method and system for identifying
a business organization that needs
transformation
Jack Azagury
Group Chief Executive - Strategy & Consulting at Accenture
New York City Metropolitan Area
Summary
Jack serves as Accenture’s Group Chief Executive—Strategy &
Consulting. He leads the $15 billion* Strategy & Consulting service.
Jack is a member of the company’s Global Management Committee.
Jack brings deep strategy, technology, and client experience
—the hallmarks of Strategy & Consulting at Accenture—to his
new leadership role, as well as extensive global experience and
perspective, having lived and worked in the UK, France, Japan
and the US supporting many global clients. Most recently, he was
Accenture’s Market Unit Lead – US Northeast, and before that led
the company’s Resources business in North America. Azagury
also served as the global lead for Accenture’s Smart Grid Services
business and led Accenture’s Strategy and Management Consulting
practice for North America Resources. 
For a wide range of clients, he has led large digital transformations,
zero-based operational improvements, energy transition programs,
and mergers, acquisitions and divestitures. He is a frequent
speaker at conferences and has published numerous articles on the
energy transition and the shift towards renewable energy, and the
transformation towards a distributed and digitally enabled grid. 
Jack is deeply involved and active with Accenture’s Employee
Resource Groups, a member of the Board of Directors of Hillel
International and a member of the Board of Directors of the
Partnership for New York. 
Prior to joining Accenture, Jack worked on strategy and business
development in the software industry. 
Jack joined Accenture in 1996 in London and became a managing
director in 2003. He holds a Master of Software Engineering
degree from Imperial College London and a Master of Business
Administration degree from INSEAD. 
 Page 1 of 3
https://www.linkedin.com/in/jack-azagury?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BgFCaFFuXTV27kYze4UNLVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/jack-azagury?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BgFCaFFuXTV27kYze4UNLVQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
http://www.accenture.com
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/about/leadership/leadership-index
https://www.accenture.com/us-en/about/leadership/leadership-index
 
*Illustrative based on mid-point of Accenture’s FY22 guidance
provided in March, results could be higher or lower
Experience
Accenture
26 years 11 months
Group Chief Executive - Strategy & Consulting
June 2022 - Present (9 months)
New York City Metropolitan Area
Senior Managing Director - Market Unit Lead - US Northeast
March 2020 - June 2022 (2 years 4 months)
Responsible for clients, people, offices, community involvement, and P&L
across the Northeast US)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(7).pdf: Contact
www.linkedin.com/in/peter-
burns-97050a (LinkedIn)
Top Skills
Management Consulting
Business Transformation
Strategy
Peter Burns
CEO Accenture ANZ
Greater Sydney Area
Summary
Peter is a Senior Managing Director with Accenture based in Sydney
with more than 29 years of advisory experience in retail banking,
wholesale/ institutional banking, wealth management and insurance.
He has worked with business leaders throughout Australia, Asia
Pacific and the U.K. on corporate strategy, M&A and business model
transformation. 
Prior to Accenture, Peter led PwC’s Global Banking and Capital
Markets Practice having joined PwC with the acquisition of Booz
& Co where he was a member of the Booz & Co. Board and
subsequently the shareholder committee. 
He has a Masters in Business Administration from Harvard
University Graduate School of Business Administration where he
graduated with high distinction and was elected a George F. Baker
Scholar and Bachelor Degree in Commerce from Bond University.
Experience
Accenture
1 year 11 months
CEO Accenture Australia and New Zealand
November 2021 - Present (1 year 4 months)
Australia
Financial Services Lead ANZ
April 2021 - November 2021 (8 months)
Sydney, New South Wales, Australia
Strategy& (Part of the PwC Network)
5 years 9 months
Board Member - PwC SEAC Consulting
July 2020 - March 2021 (9 months)
Sydney, New South Wales, Australia
 Page 1 of 2
https://www.linkedin.com/in/peter-burns-97050a?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BCUX%2BP5wtSgyEnXYnxeldgw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/peter-burns-97050a?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BCUX%2BP5wtSgyEnXYnxeldgw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
PwC Global Banking & Capital Markets Leader | ASEANZ Consulting
Financial Services Leader 
July 2015 - March 2021 (5 years 9 months)
Sydney, New South Wales, Australia
Booz & Company.
23 years 6 months
Member Board Of Directors - Booz & Company
March 2013 - July 2015 (2 years 5 months)
Senior Partner
February 1992 - July 2015 (23 years 6 months)
Sydney, New South Wales, Australia
Education
Harvard Business School
Master of Business Administration (M.B.A.), MBA · (1995 - 1997)
Bond University
Bachelor of Commerce (B.Com.), Commerce -- Marketing and
Statistics · (1989 - 1991)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(29).pdf: Contact
www.linkedin.com/in/shiv-iyer-
b04b27 (LinkedIn)
Top Skills
Strategy
Management Consulting
Business Transformation
Shiv Iyer
US Midwest Market Unit Lead at Accenture
Greater Chicago Area
Summary
In my current role, I am responsible for clients, people, offices,
community involvement and financial performance across the
Midwest United States. Leading more than 11,000 people —
spanning Illinois, Indiana, Iowa, Kansas, Kentucky, Michigan,
Minnesota, Missouri, Nebraska, North Dakota, Ohio, South Dakota,
West Virginia and Wisconsin — I focus on delivering continuous
innovation to clients, attracting and retaining top talent, and
strengthening Accenture’s impact in local communities. I am a
member of Accenture’s Global Management Committee and the
North America Leadership Team and also serve as the Diamond
Client Account Lead for a global consumer products company. 
In my prior role, as Accenture’s Midwest Strategy & Consulting lead,
I was responsible for overseeing the company’s Strategy, Consulting
and Data and Analytics practices. Prior to that, I co-led the creation
of Accenture’s Competitive Agility center of excellence and led
Accenture’s Consumer Products, Retail and Life Sciences consulting
practices for North America. 
I am a regular contributor to the Consumer Brand Association,
convening conversations with CEOs and influencers from around
the world and across industries. I began my consulting career at A.T.
Kearney, where I spent a decade progressing to various leadership
roles and joined Accenture in 2010 as a Managing Director. I serve
on the Chicago board of Pratham USA, an organization whose
mission is to ensure that every child is in school and learning well.
I earned both my bachelor’s degree in instrumentation engineering
and my master’s degree in management sciences at the University
of Mumbai; I earned an MBA from Indiana University’s Kelley School
of Business.
Experience
 Page 1 of 4
https://www.linkedin.com/in/shiv-iyer-b04b27?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B%2Ban%2BsZhWScW5zR87hBfK5w%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/shiv-iyer-b04b27?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3B%2Ban%2BsZhWScW5zR87hBfK5w%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Accenture
2 years
Senior Managing Director- US Midwest Market Unit
March 2022 - Present (1 year)
Chicago, Illinois, United States
Senior Managing Director
March 2021 - Present (2 years)
Alvarez & Marsal
Managing Director 
November 2020 - March 2021 (5 months)
Chicago, Illinois, United States
Accenture
10 years 4 months
Sr. Managing Director -Strategy and Consulting Lead Midwest
July 2010 - October 2020 (10 years 4 months)
Greater Chicago Area
Lead all Strategy , Functional and Industry Consulting and Applied Intelligence
services for the MW Market Unit. 
Products NA Consulting Lead 
January 2018 - November 2019 (1 year 11 months)
Chicago 
Lead all Functional consulting practices for Products ( CPG, Retail,
Lifesciences, Industrials, Automotive and Travel ) in North America.
North America Competitiveness Lead
September 2016 - January 2018 (1 year 5 months)
Chicago
Co -lead Accenture's Competitiveness Center of Excellence for North America.
The Competitiveness COE works with the boards and C-Suites of Fortune
1000 companies to drive a step change in financial performance and agility. 
North America CPG Management Consulting Practice Lead
October 2015 - October 2017 (2 years 1 month)
Greater Chicago Area
Lead Accenture's North America CPG Management Consulting practice.
Responsible for developing and scaling new offerings/client propositions,
thought leadership, talent management and practice development)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(33).pdf: Senior Managing Director - Accenture Financial Services Lead for
Europe 
novembre 2019 - marzo 2020 (5 mesi)
Milano, Italia
I was Financial Services Lead for Europe: in this role, I was responsible for
setting the overall vision and strategy, for sales and revenue growth, and for
guiding the development of client relationships across Europe.
Senior Managing Director - Accenture Financial Services Lead for Italy,
Central Europe and Greece
ottobre 2017 - ottobre 2019 (2 anni 1 mese)
Milano, Italia
Financial Services Lead for ICEG: in this role, I was responsible for the
Accenture Business practices in the Banking, Insurance and Capital Markets
areas in Italy, Central Europe and Greece.
Senior Managing Director - Accenture Strategy - Banking Global Lead
maggio 2015 - settembre 2017 (2 anni 5 mesi)
Milano, Italia
I was Senior Managing Director for Banking within Accenture Strategy. My
role focused on enabling organizations in the Banking industry to improve their
competitive advantage and create future value.
Senior Managing Director - Accenture Strategy - Europe, Africa and
Latin America
giugno 2012 - maggio 2015 (3 anni)
Milano, Italia
I was Senior Managing Director for Europe, Africa and Latin America within
Accenture Strategy. In this role I focused on enabling organizations across all
 Page 2 of 3
 
industries to improve their competitive advantage in markets that are being
transformed by digital technologies.
Senior Managing Director Management Consulting Italy, Greece and
Emerging Markets
giugno 2009 - giugno 2012 (3 anni 1 mese)
I was the Managing Director for Management Consulting & Integrated Markets
in IGEM and member of the EALA MCIM executive committee, with the
objective of driving growth in the IGEM region and bringing the full talents and
power of Accenture to our clients.
Managing Director
gennaio 2007 - giugno 2009 (2 anni 6 mesi)
I was Managing Director of Financial Services Management Consulting
practice at EALA level (Europe, Africa and Latin America) with the objective
of driving growth in the Banking/Insurance/ Capital Market industries across
different geographies.
Formazione
University of California, Irvine - The Paul Merage School of Business
Master of Business Administration (M.B.A.) · (1988 - 1990)
UC Irvine
Bachelor of Science in Economics · (1984 - 1988)
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(49).pdf: Contact
www.linkedin.com/in/sanjeev-
vohra-271386ba (LinkedIn)
Top Skills
Management
Program Management
Business Transformation
Languages
Hindi
Punjabi
English
Sanjeev Vohra
Global Lead - Accenture Applied Intelligence | Growth Strategy |
Business Innovation | Talent Transformation
New York City Metropolitan Area
Summary
“Good is the Enemy of Great” is my motto, which has inspired me to
lead several transformation initiatives in well established companies,
fueling extraordinary growth. 
Conceptualizing and operationalizing a large-scale enterprise
transformation involves taking a full view of the existing business,
the competition & market, and developing an execution strategy
to drive exceptional results. It also calls for mastery over multiple
skills that I’ve built over the years—strategic thinking, in-depth
understanding of business drivers, deep knowledge of legacy &
emerging technologies, change management and talent re-skilling at
scale.
I am very passionate about and committed to continuously exploring
ways to humanize technology and unlock the value of data which is
at the core of all digital transformations. This is especially critical as
we move deeper into the era of Intelligence leveraging the power of
robotics, machine learning and artificial intelligence. 
Another area that is close to my heart is people development. For
more than a decade, I have led large-scale talent development
& transformation programs to ensure our people have the right
leadership and technical skills to stay ahead in this fast changing
world.
Experience
Accenture
20 years 4 months
Senior Managing Director, Global Lead - Applied Intelligence
September 2020 - Present (2 years 6 months)
 Page 1 of 4
https://www.linkedin.com/in/sanjeev-vohra-271386ba?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BV%2B6dV75GSk2bMeeqGl1Lpw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/sanjeev-vohra-271386ba?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BV%2B6dV75GSk2bMeeqGl1Lpw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
In my current role, I help clients across industries and markets evaluate,
maximize, and scale business value through the strategic application of data,
advanced analytics, artificial intelligence and automation. I am responsible
for the company’s Applied Intelligence strategy, business growth, talent
development, industry-specific solutions, strategic partnerships and V&A
investments. I am also a member of Accenture's Global Management
Committee.
Senior Managing Director, Chief Strategy Officer, Accenture
Technology
November 2002 - September 2020 (17 years 11 months)
Bangalore
From 2002 to 2020, I led many strategic initiatives at Accenture, each being in
response to the dynamic demands of our clients and changing global business
environment.
As the Chief Strategy Officer, my focus was on creating new value and growth
avenues for Accenture Technology business by leveraging new technologies,
industry opportunities and emerging business models.
I was responsible for creating a vision for next waves of growth and strategic
roadmap, ensuring effective utilization of investment funds, and managing the
inorganic growth. I was leading the effort for developing our talent strategy
with the aim of creating a continuous learning organization and shaping a
responsible business. 
My mandate also included managing the Industry and Function practices
at Accenture to shape and drive new offerings, specializations, ecosystem
strategies, differentiated assets and new growth plays.
Global Data Business Group Lead, Accenture Technology
January 2017 - February 2020 (3 years 2 months)
Unlocking the business value by re-imagining the Data services
Hot on the heels of the digital transformation was the AI hype in the market.
And why not? By 2017, digital native companies were already demonstrating
the power and potential of artificial intelligence (AI). They were teasing the rest
of the world with what they could do with discoverable and reliable data)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(12).pdf: Contact
www.linkedin.com/in/paolodalcin
(LinkedIn)
www.accenture.it (Other)
www.accenture.com/security
(Other)
Top Skills
Sales Management
Security
Account Management
Languages
Italian (Native or Bilingual)
English (Full Professional)
Paolo Dal Cin
Accenture Security Global Lead
Milan, Lombardy, Italy
Summary
Accenture Security Global Lead, Senior Managing Director
Member of Accenture Global Management Committee: https://
www.accenture.com/us-en/about/leadership/leadership-index
Cyber Security Leader with 20+ years of experience with clients
across multiple industries such as telecommunications, media,
financial services, utilities and the public sector. 
Specialized in cyber security strategy, business resilience, cyber
defense, cloud protection, incident response and managed security
services.
Helping Boards, CEOs, CISOs and CxOs in their Cyber Security and
Digital Resilience journey
$6B Security Business, 16.000 Security Professionals
https://newsroom.accenture.com/news/accenture-appoints-paolo-
dal-cin-to-lead-its-security-business.htm
https://www.accenture.com/us-en/services/security-index
Experience
The Aspen Institute
Member of the Aspen Global Cyber Group
October 2022 - Present (5 months)
ISACA
Member of ISACA Digital Trust Advisory Council 
September 2022 - Present (6 months)
 Page 1 of 3
https://www.linkedin.com/in/paolodalcin?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BxZxqjhjMSoGvMGqITDx40g%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/paolodalcin?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BxZxqjhjMSoGvMGqITDx40g%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
http://www.accenture.it
http://www.accenture.com/security
http://www.accenture.com/security
 
Agenzia per la Cybersicurezza Nazionale
Member of the Technical-Scientific Committee
June 2022 - Present (9 months)
Accenture
20 years
Accenture Security Global Lead
April 2022 - Present (11 months)
Milan, Lombardy, Italy
Helping Boards, CEOs, CISOs and CxOs in their Cyber Security and Digital
Resilience journey
$6B Cyber Security Business
16.000 Security professionals
Senior Managing Director, Accenture Europe Security Lead
December 2020 - June 2022 (1 year 7 months)
Milan, Lombardy, Italy
Accenture Security Lead for Europe and LATAM - Global Managing
Director
March 2019 - December 2020 (1 year 10 months)
Milan Area, Italy
Accenture Security Lead for Europe and LATAM
Managing Director - ICEG Security Lead and CMT Security Global Lead
September 2016 - March 2019 (2 years 7 months)
Milan Area, Italy
Managing Director - Accenture Security 
Accenture Security Lead - ICEG region (Italy, Eastern Europe, Greece)
Accenture Security Global Lead - CMT Market (Communication, Media,
Gaming, Aerospace&Defense, High Tech)
Managing Director - IGEM Security Lead
2003 - August 2016 (13 years)
Accenture Managing Director - Security Lead for the IGEM region (Italy,
Russia, Eastern Europe, Greece, Turkey and Middle East) with Consulting
experience in Information, ICT and Cyber Security; in this role, I'm responsible
for the Security business unit including P&L management, business
development, sales, presales and consulting activities.
 Page 2 of 3
 
University of Udine
Professor - External/Assistant
2005 - 2011 (6 years)
Network Security Course for M.Sc)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(42).pdf: Contact
www.linkedin.com/in/martyrodgers
(LinkedIn)
Top Skills
Strategic Planning
Non-profits
Management Consulting
Marty Rodgers
Public & Private Sector Transformation @ Scale | Leader in I&D |
Corporate Citizenship Expert | Board Member
Falls Church, Virginia, United States
Summary
I help leaders transform their organizations for a post-covid, purpose-
filled, value-driven world leveraging our teams, ecosystem partners,
and recent acquisitions to realize the promise of human ingenuity
and technology. 
Believing that all three sectors have a role to play in solving the
big issues of our time, I have 25 years of corporate, government
and nonprofit sector experience as a management consultant
and executive including developing strategy and implementing
transformational change programs.
Leading our Washington DC Metro Office and South Market Unit
from Maryland to Florida over to Texas, I met my beautiful wife in
grad school 20 years ago and we have three awesome teenagers
Malcolm, Zora, Mae. I root for all things Philadelphia (my hometown)
and all things Irish (my alma mater). I am a service geek and serial
nonprofit board member, love coffee, dessert, and peanut butter.
Experience
Accenture
25 years 6 months
Senior Managing Director - Market Unit Lead - South & Metro
Washington DC Office Managing Director 
March 2020 - Present (3 years)
Washington DC-Baltimore Area
Accenture is a global professional services company with leading capabilities
in digital, cloud and security. Combining unmatched experience and
specialized skills across more than 40 industries, we offer Strategy and
Consulting, Interactive, Technology and Operations services—all powered by
the world’s largest network of Advanced Technology and Intelligent Operations
centers. Our 506,000 people deliver on the promise of technology and human
ingenuity every day, serving clients in more than 120 countries. We embrace
 Page 1 of 4
https://www.linkedin.com/in/martyrodgers?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BLC03djSORKaYtEiNupJpkQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/martyrodgers?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BLC03djSORKaYtEiNupJpkQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
the power of change to create value and shared success for our clients,
people, shareholders, partners and communities.
Senior Managing Director - US Southeast & Metro Washington DC
Office Managing Director
September 2019 - March 2020 (7 months)
Washington D.C. Metro Area
Managing Director for Metro Washington, D.C. & SE Managing Director
for Health & Public Service 
December 2016 - March 2020 (3 years 4 months)
Washington D.C. Metro Area
Managing Director, Accenture Metro Washington DC & Managing
Director, Accenture Federal Services
January 2015 - March 2020 (5 years 3 months)
Managing Director
September 1997 - January 2015 (17 years 5 months)
Lead client service in Accenture Federal Service. Founder & Executive
Director of Accenture Nonprofit Group)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(27).pdf: Contact
www.linkedin.com/in/dyllis-hesse
(LinkedIn)
www.accenture.com (Personal)
Top Skills
Retail
Organizational Design
Strategy
Languages
English
German
Dyllis Hesse
North America Client Relationship Leader I Consumer Industries
I Senior Managing Director I Member of Global Management
Committee
Washington DC-Baltimore Area
Summary
Dyllis is a Member of Accenture´s Global Management Committee
and Accenture’s North America Client Account Lead, representing
the voice of Accenture's largest clients in NA. She's focused
on building and growing relationships with the world’s leading
companies operating in North America to drive 360 degree client
value and co-innovation. 
Dyllis is also a Senior Global Client Account Executive for a
Fortune 500 Company in the hospitality and travel industry, with
international experience in US, UK and Germany. She leads the
overall relationship with one of Accenture's largest clients, the global
P&L for the work Accenture delivers, and ultimately partners with
her clients to drive high performance and innovation in the client/
Accenture collective teams.
In her previous role, based in Atlanta, Georgia for 5 years, she
was the Global Client Account Executive for one of the largest
global Consumer Products companies. This included a focus on
Digital Transformation - particularly on driving growth by leveraging
automation, AI, and analytics as well as cloud solutions, and digital
commerce. 
Dyllis has consulting experience in strategy, digital technology,
change management, and Global Business Services (GBS) to
help clients drive their business transformation agenda. Dyllis has
also worked for a Fashion Retailer in the UK, leading their Retail
Operations business. 
Dyllis's leadership style blends key capabilities of intellect, energy,
optimism and pragmatism. She is strategic, setting ambitious visions,
and then has the executional energy and collaboration skills to rally
the team to deliver it flawlessly. Her passion is leading diverse teams
 Page 1 of 4
https://www.linkedin.com/in/dyllis-hesse?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BBXYQdC6RRmymcVNfW%2Bv6Jg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/dyllis-hesse?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BBXYQdC6RRmymcVNfW%2Bv6Jg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com
 
to challenge the status quo, and create a better version of today.
She is viewed as an energizing leader by her team and her clients.
She's worked across North America, Germany and the UK and loves
leading international teams. Fluent in English and German.
Dyllis is also an engaged mother of 3 young children, based in
Washington D.C. She's an avid runner, tennis player and loves to
travel across the world to new places.
Experience
Accenture
24 years
NA Client Account Leader I Global Management Committee Member
June 2022 - Present (9 months)
Dyllis joined Accenture's Global Management Committee as the North
American Client Account Lead, representing the voice of our largest clients
operating in North America, building and growing relationships with these
leading companies, and driving 360 degree client value and co-innovation.
Senior Managing Director
December 2020 - Present (2 years 3 months)
Washington DC-Baltimore Area
Managing Director
1999 - Present (24 years)
Atlanta, Georgia
I am a Managing Director at Accenture with over 19 years of international
experience helping clients undergo substantial change programs. I have a
strong management consulting background in consumer goods and retail, with
deep experience in strategy, digital technology, change management, and
Global Business Services (GBS). I currently serve as the Global Client Account
Lead for a Fortune 100 Company in the hospitality and travel industry.
I have been based in Atlanta for 5 years, and previously led the Accenture
team at a large CPG company. Mainly focused on Digital Transformation)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(38).pdf: Contact
www.linkedin.com/in/karthik-narain
(LinkedIn)
Top Skills
IT Strategy
Leadership
Strategy
Karthik Narain
Lead - Accenture Cloud First
San Francisco Bay Area
Summary
Karthik Narain leads Accenture Cloud First, with responsibility for
helping clients shape, move and operate their business in the cloud
to accelerate innovation and achieve their digital transformation
goals. He is also a member of the company’s Global Management
Committee.
Accenture Cloud First is a multi-service group that provides the
full stack of cloud services spanning strategy, industry-specific
cloud journeys, cloud data and AI, migration and modernization,
and secure cloud management, as well as change and talent
management for continuous innovation. In his role leading Accenture
Cloud First, Karthik is focused on extending the company’s
leadership in Cloud with Accenture’s ecosystem partners and
through investments in deep industry capabilities and solutions,
cloud acquisitions, and talent.
A technology industry veteran, Karthik most recently served as the
lead for Accenture Technology in North America, helping guide
Global 2000 brands in using the power of the cloud and other
technologies to transform their businesses. Over his 20-year career,
he has led many innovative technology programs for clients across
a variety of industry sectors, including Financial Services, High Tech
and Software and Platforms. Karthik also previously led Technology
services for Accenture’s Communications, Media and High-Tech
industry segments.
Prior to joining Accenture in 2015, Karthik led the High-Tech
business for HCL Technologies where he was responsible for some
of HCL’s marquee client relationships.
Karthik holds a bachelor’s degree in Commerce and Management
from the University of Madras and a master’s degree in Computer
Science and Management from Bharathidasan University. He is
based in Silicon Valley.
 Page 1 of 3
https://www.linkedin.com/in/karthik-narain?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BRRMgAcipSyWYOjf%2Fq%2FnPnw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/karthik-narain?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BRRMgAcipSyWYOjf%2Fq%2FnPnw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Experience
Accenture
8 years 2 months
Lead - Accenture Cloud First
October 2020 - Present (2 years 5 months)
Lead – Technology, North America 
March 2020 - October 2020 (8 months)
Senior Managing Director and Global Technology Lead -
Communications, Media and High Tech
June 2017 - March 2020 (2 years 10 months)
Accelerating the rotation to Digital / NEW through Accenture's Intelligent
Platforms strategy (SAP, Oracle, SFDC, Microsoft and Workday), Cloud
Adoption (AWS, GCP, Azure, Ali Cloud) by creating a Digitally Decoupled
enterprise. Exciting, isn't it! 
Managing Director, NA Technology Lead for Communications, Media
and High Tech
January 2015 - May 2017 (2 years 5 months)
San Francisco Bay Area
Responsible for Accenture's Technology Services business for Comms, Media,
High Tech and A&D in NA
HCL Technologies
Vice President and Vertical Head, High Tech and Automotive
June 2005 - January 2015 (9 years 8 months)
San Francisco Bay Area
P&L responsibility for HCL's High Tech and Automotive vertical segments.
Infosys
Senior Relationship Manager - Financial Services
June 2000 - June 2005 (5 years 1 month)
Greater Minneapolis-St. Paul Area
Responsible for managing and delivering complex programs for a large capital
markets client.
Education
 Page 2 of 3
 
Bharathidasan University, Tiruchirappalli
Masters in Computer systems and Business, Computer Science
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(44).pdf: Contact
www.linkedin.com/in/
christiesmithaccenture (LinkedIn)
Top Skills
Management Consulting
Business Transformation
Organizational Design
Honors-Awards
Women Worth Watching in
Leadership
Most Influential Women in Business
2012-2014
Queer 50 2021
Top 100 Outstanding LGBT+
Executive Role Model 2022
Publications
Its Time to Get Under the Covers
Transforming for Hybrid: Why Trust
is Key to a Productive Workplace
The Future of Work: Productive
Anywhere
The Future of Work: Productive
Anywhere
Four Ways Responsible Tech Can
Change Lives and Business
Christie Smith
Global Lead, Talent & Organization/Human Potential, Accenture:
Enriching the relationship between people and technology to create
transformation
Seattle, Washington, United States
Summary
Nothing changes unless people do. As Global Lead for Talent &
Organization/Human Potential at Accenture, I work with leaders to
create transformation by enriching the relationship between people
and technology.
Every day I am inspired by the incredible enthusiasm, talent and
commitment of my colleagues around the world. We help companies
reimagine their businesses, develop new styles of leadership, create
strong cultures, design and accelerate change programs, find and
skill the right talent, and redesign HR functions.
As part of Accenture’s Global Management Committee and
Accenture’s Pride Champion, I am proud to be part of a leadership
team who care deeply about what we do and the future we can
create.
I am passionate about Inclusion, Diversity and Equity. Ensuring
that we are all seen, heard and valued helps both individuals
and organizations grow and thrive. Accenture’s commitment to
accelerating equality for all has never been more relevant than it is
today. I am also proud to have been included in Fast Company’s
Queer 50 List in 2021. I was previously global vice president for
Inclusion and Diversity at Apple and have worked throughout my
career to create and sustain cultures of inclusivity and belonging
which lead to more adaptable and agile organizations.
I’m also passionate about sustainability. Modern C-suite leaders
are increasingly focused on their responsibility to people and
the environment. For too long, we have focused on financial and
shareholder success. Today, we need to create sustainable and
equitable impact. We need to weave sustainability into the DNA of all
organizations.
 Page 1 of 4
https://www.linkedin.com/in/christiesmithaccenture?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BFuPi3y5ATPikTFXsrcI%2Fkw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/christiesmithaccenture?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BFuPi3y5ATPikTFXsrcI%2Fkw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
What else am I passionate about? Well, there is nothing I love more
than my family. We love the great outdoors, sports, and hanging out
with our dogs. I cherish my time with my wife and kids.
Experience
Accenture
2 years 4 months
Senior Managing Director, Talent & Organization/Human Potential
Global Lead
March 2021 - Present (2 years)
Lead Accenture's Talent & Organization/Human Potential practice and
responsible for:
- Directing a Global team that help's Accenture's clients and their workforces
drive innovation by transforming leadership mindsets, talent, and cultures
-Driving and evolving Accenture's "future of work" thought leadership and
research to deliver breakthrough competitive insights to C-suite leaders
- Shaping the market strategy for the Talent & Organization practice, including
offering development, partnership, acquisition, and investment initiatives)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(2).pdf: Contact
www.linkedin.com/in/muqsitashraf
(LinkedIn)
Publications
The Global Quest for Light Tight Oil:
Reality or Myth?
The Shale Game-changer
The Comeback of Technology in
Unconventionals
Ushering in Collaboration and
Integration: Returning the oil and gas
industry to the era of superior returns
Unconventional Gas Uncertainty
- Implications on Portfolio
Management and Development
Practices
Muqsit Ashraf
Chief Executive - Accenture Strategy | Global Management
Committee - Accenture
Greater Houston
Summary
Muqsit Ashraf leads Accenture Strategy, overseeing a multibillion-
dollar business unit across 19 industries in over 120 countries
that helps clients tap new market opportunities, apply innovative
technologies, and execute large-scale enterprise reinvention
and transformation. He is also a member of Accenture’s Global
Management Committee.
Prior to this role, Muqsit led Accenture’s global Energy industry
practice. He has more than 20 years of experience across the
energy value chain, advising and partnering with global energy
leaders at most of the large international and national oil companies,
oil field and energy service companies, and independents, helping
them redefine business strategy, lead large scale transformation
programs, and drive innovation.
Muqsit leads Accenture’s Global Energy Board of more than 40
global CEOs/CXOs and has sponsored Accenture's relationships in
Energy and Resources with the World Economic Forum, the World
Energy Council, and the World Petroleum Congress.
Muqsit has a bachelor’s degree in Chemistry, and a Master's in
Business Administration from Yale University.
https://www.accenture.com/us-en/about/leadership/leadership-index
Experience
Accenture
7 years 4 months
Global Management Committee
August 2022 - Present (7 months)
Houston, Texas, United States
 Page 1 of 2
https://www.linkedin.com/in/muqsitashraf?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BivDG9hi%2BRaOK8skkX1Xo%2Fw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/muqsitashraf?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BivDG9hi%2BRaOK8skkX1Xo%2Fw%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Chief Executive, Accenture Strategy
August 2022 - Present (7 months)
Houston, Texas, United States
Global Leadership Council (GLC)
June 2016 - Present (6 years 9 months)
Houston, Texas Area
Senior Managing Director and Global Energy Industry Lead
March 2020 - August 2022 (2 years 6 months)
Houston, Texas, United States
Lead Accenture’s Energy practice globally across all service lines - Strategy,
Consulting, Applied Intelligence, Technology/Industry X.0, and Innovation
Senior Managing Director and Global Head of Energy Practice
(Accenture Strategy)
November 2015 - February 2020 (4 years 4 months)
Houston, Texas Area
Schlumberger
Managing Director, North America, (Business Consulting)
June 2009 - October 2015 (6 years 5 months)
Houston, Texas
Managed the North American Oil & Gas business consulting business and the
Houston office for 4+ years - achieved 5x revenues and 3rd place on Vault
ranking for best energy consulting firms. 
Member of the global Management Committee - led divestiture of consulting
business to Accenture Strategy (completed in Nov'15).
Booz & Company (formerly Booz Allen Hamilton)
Principal
August 1999 - May 2005 (5 years 10 months)
Energy and operations practices.
Education
Yale University
 · (1999)
 Page 2 of 2)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(50).pdf: Contact
www.linkedin.com/in/sanjeev-
vohra-271386ba (LinkedIn)
Top Skills
Management
Program Management
Business Transformation
Languages
Hindi
Punjabi
English
Sanjeev Vohra
Global Lead - Accenture Applied Intelligence | Growth Strategy |
Business Innovation | Talent Transformation
New York City Metropolitan Area
Summary
“Good is the Enemy of Great” is my motto, which has inspired me to
lead several transformation initiatives in well established companies,
fueling extraordinary growth. 
Conceptualizing and operationalizing a large-scale enterprise
transformation involves taking a full view of the existing business,
the competition & market, and developing an execution strategy
to drive exceptional results. It also calls for mastery over multiple
skills that I’ve built over the years—strategic thinking, in-depth
understanding of business drivers, deep knowledge of legacy &
emerging technologies, change management and talent re-skilling at
scale.
I am very passionate about and committed to continuously exploring
ways to humanize technology and unlock the value of data which is
at the core of all digital transformations. This is especially critical as
we move deeper into the era of Intelligence leveraging the power of
robotics, machine learning and artificial intelligence. 
Another area that is close to my heart is people development. For
more than a decade, I have led large-scale talent development
& transformation programs to ensure our people have the right
leadership and technical skills to stay ahead in this fast changing
world.
Experience
Accenture
20 years 4 months
Senior Managing Director, Global Lead - Applied Intelligence
September 2020 - Present (2 years 6 months)
 Page 1 of 4
https://www.linkedin.com/in/sanjeev-vohra-271386ba?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BR97H%2BNkPSkGSQBMLPu7s4Q%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/sanjeev-vohra-271386ba?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BR97H%2BNkPSkGSQBMLPu7s4Q%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
In my current role, I help clients across industries and markets evaluate,
maximize, and scale business value through the strategic application of data,
advanced analytics, artificial intelligence and automation. I am responsible
for the company’s Applied Intelligence strategy, business growth, talent
development, industry-specific solutions, strategic partnerships and V&A
investments. I am also a member of Accenture's Global Management
Committee.
Senior Managing Director, Chief Strategy Officer, Accenture
Technology
November 2002 - September 2020 (17 years 11 months)
Bangalore
From 2002 to 2020, I led many strategic initiatives at Accenture, each being in
response to the dynamic demands of our clients and changing global business
environment.
As the Chief Strategy Officer, my focus was on creating new value and growth
avenues for Accenture Technology business by leveraging new technologies,
industry opportunities and emerging business models.
I was responsible for creating a vision for next waves of growth and strategic
roadmap, ensuring effective utilization of investment funds, and managing the
inorganic growth. I was leading the effort for developing our talent strategy
with the aim of creating a continuous learning organization and shaping a
responsible business. 
My mandate also included managing the Industry and Function practices
at Accenture to shape and drive new offerings, specializations, ecosystem
strategies, differentiated assets and new growth plays.
Global Data Business Group Lead, Accenture Technology
January 2017 - February 2020 (3 years 2 months)
Unlocking the business value by re-imagining the Data services
Hot on the heels of the digital transformation was the AI hype in the market.
And why not? By 2017, digital native companies were already demonstrating
the power and potential of artificial intelligence (AI). They were teasing the rest
of the world with what they could do with discoverable and reliable data)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(19).pdf: Contact
www.linkedin.com/in/jimmy-
etheredge (LinkedIn)
www.accenture.com (Other)
www.accenture.com/us-en/careers
(Other)
Top Skills
Acquisition Integration
Management Consulting
Business Process
Jimmy Etheredge
CEO – Accenture, North America
Atlanta, Georgia, United States
Summary
As Accenture’s chief executive officer of North America, I lead our
business in the United States—the company’s largest market—
and Canada. My responsibilities include driving sales, revenue and
profit by bringing clients innovative solutions and services from
Accenture’s strategy and consulting, interactive, technology and
operations practices. I am also responsible for our 60,000 amazing
people in North America and helping them do interesting work
and bring their authentic selves to work everyday. I am a member
of Accenture’s Global Management Committee to ensure we are
bringing the best of Accenture's global capabilities to our North
America clients. 
I am passionate about paving paths for people. I try to be a
champion of improving equality, a proactive supporter of mental
well-being programs and an advocate for bridging gaps for
vulnerable communities. I have helped champion Accenture’s
national professional apprenticeship program, giving members of
underrepresented groups new skills so they can adapt and thrive in
the digital economy. 
In 2020, I was recognized on the “Power Players of Consulting”
list published by Business Insider and ranked 28th in the Atlanta
Business Chronicle’s “Power 100,” a list of the most influential
Atlantans. In 2018, I was named one of Atlanta’s most admired
CEOs by the Atlanta Business Chronicle. I enjoy helping nonprofit
organizations and I'm on the boards of the Global Apprenticeship
Network (a Switzerland-based coalition dedicated to work-readiness
programs), the Metro Atlanta Chamber of Commerce and the Atlanta
Police Foundation. I am on the Board of Councilors for the Carter
Center (President Carter is one of my favorite inspiring leaders). 
I earned a Bachelor of Science degree in industrial engineering from
Georgia Tech and live in Atlanta, where I remain active with my alma
mater as a trustee and board member.
 Page 1 of 3
https://www.linkedin.com/in/jimmy-etheredge?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Ba%2F7%2F%2BRwWSK295ICb0slZGA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/jimmy-etheredge?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3Ba%2F7%2F%2BRwWSK295ICb0slZGA%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com
https://www.accenture.com/us-en/careers
https://www.accenture.com/us-en/careers
 
With over 35 years experience helping clients leverage technology
to transform and grow. I am passionate about the potential of
technology and human ingenuity to change the way the world works
and lives. 
Outside of work you can find me enjoying a good bourbon, playing
bad golf, practicing yoga, living and dying with Georgia Tech and
Atlanta sports (mostly dying) and being tugged through the park by
our 2 boisterous labs.
Experience
Accenture
38 years
Chief Executive Officer - North America
September 2019 - Present (3 years 6 months)
Atlanta, Georgia, United States
Senior Managing Director - US Southeast
December 2016 - September 2019 (2 years 10 months)
Responsible for leading the company’s business in a region covering 10 states
—Alabama, Florida, Georgia, Kentucky, Maryland, Mississippi, North Carolina,
South Carolina, Tennessee and Virginia—as well as the District of Columbia. 
Senior Managing Director - Products North America
2011 - September 2019 (8 years)
Senior Managing Director – Products North America
Responsible for driving sales, revenue and profitability across the Products
Operating Group in North America)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(37).pdf: Contact
www.linkedin.com/in/kaushal-
mody-23b798 (LinkedIn)
Top Skills
Management Consulting
Business Transformation
Operational Excellence
Kaushal Mody
Chief Assets and Solutions Officer, Accenture I Global Management
Committee
Mumbai, Maharashtra, India
Summary
I am a global business leader in Accenture having worked across
multiple areas (consulting, transformation, intelligent operations,
ventures & acquisitions), in various roles (for incubating, scaling and
running businesses, as global innovation lead, as growth & strategy
lead) across multiple industry sectors. I have been strategic advisor
to several Fortune 500 clients and have 25 years of experience. 
Key Business Achievements 
- Leading our acquisition strategy and execution including
strategic assessments 100+ targets, due diligence, deal closure,
transformation and integration post closure including working with
acquired leadership 
- Incubated and scaled high-growth and innovation-led businesses.
Running large global businesses to drive superior profitable growth 
- Key businesses cover supply chain / Industry X, digital marketing
operations, global operations for software and platform clients,
Analytics, intelligent automation and AI business
- Lead Growth & Strategy for Accenture Operations to drive vision,
strategy and business and investment plan execution including
active vacquisition strategy. Delivered superior growth vs. market
basket along with business leadership team
- Lead Accenture Operations' focus on innovation and digital
technologies to drive human + machine operating models and drive
business outcomes at speed and scale
- Key client ownership across various industry sectors including
software & platforms, fintech, financial services, consumer goods,
retail, automotive and industrial products 
- Led Senior level global leadership teams to drive business results 
Key Interest Areas 
- Amplifying the power of new technologies by combining with
deep business understanding to create tangible impact for clients'
businesses and improve the communities we live in
 Page 1 of 3
https://www.linkedin.com/in/kaushal-mody-23b798?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BVrTqOZfTSu%2Bd7HgG%2BcBlPg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/kaushal-mody-23b798?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BVrTqOZfTSu%2Bd7HgG%2BcBlPg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
- Investments in and working with talented teams who are our
biggest strength
- Staunch supporter of the right to quality education for all
- In free time, enjoy traveling, experimenting with different cuisines,
following sports and reading autobiographies.
Key member of top level Accenture executive leadership teams 
- Global Management Committee
- Global Leadership Council 
- Global Operations Committee 
- Applied Intelligence Leadership Council
- Financial Services Global Leadership Team 
- Advisory team of Ventures & Acquisitions 
- CEO Circle
Key skills 
- Incubating and scaling innovation led businesses
- Running large global businesses 
- Business Transformation and Innovation
- Ventures & Acquisitions 
- Growth & Strategy
Experience
Accenture
26 years 5 months
Chief Assets and Solutions Officer
September 2022 - Present (6 months)
India
Senior Managing Director
October 1996 - Present (26 years 5 months)
Mumbai Area, India
Chief Strategy Officer, Offerings and Innovation Lead, Data, AI and
Technology, Business Lead - Accenture Operations 
Business Transformation - Accenture Strategy and Consulting
Infosys, UK
Client Services Leadership
April 2005 - August 2007 (2 years 5 months)
 Page 2 of 3
 
Education
Indian Institute of Management, Ahmedabad
MBA, Finance & Operations · (1994 - 1996)
VJTI
Bachelor of Engineering (B.E.), Electronics · (1989 - 1993)
Ruparel College
 · (1987 - 1989)
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(32).pdf: Put simply focused on clients, people and
purpose 
Experience
AeroFarms
Board Member
March 2014 - Present (9 years)
Newark, New Jersey, United States
Advisory Board and then Full Board Member of the world’s most innovative
and one of the fastest growing disruptive tech urban farming businesses. With
strong links to sustainability, data and analytics and the intersection of digital
and physical technologies. As well as just producing great product! 
The Forum of Young Global Leaders
Board Member
February 2019 - Present (4 years 1 month)
Geneva, Switzerland
Member of the board overseeing the activities of the Young Global Leaders
Forum. A network and platforms set up by the World Economic Forum to
develop the future leaders we need to create positive change. 
Blue Rose Compass
Various not-for-profit board roles
February 2008 - Present (15 years 1 month)
United States, United Kingdom, China, Australia
Accenture
15 years 2 months
 Page 2 of 4
 
Global Management Committee Member, Sustainability Services Lead
& Chief Responsibility Officer
December 2020 - Present (2 years 3 months)
London, England, United Kingdom
Accenture Executive Lead Sponsor - WEF & UN Global Compact 
October 2014 - October 2021 (7 years 1 month)
Senior Managing Director - Accenture Strategy Europe Lead; Global
Sustainability Strategy Lead
December 2017 - 2020 (3 years)
London, England, United Kingdom
Multiple Roles - Growth, Strategy, Sustainability & Technology
January 2008 - December 2017 (10 years)
Shanghai, China & London, England
Global Growth, Strategy & Sustainability Lead - Accenture Strategy - leading
all organic & inorganic growth for Accenture Strategy globally - 3 years 
Global Lead - Digital Transformation of Industries (3 years)
Asia-Pacific Cross-Industry Strategy Lead (Shanghai for 3 years)
Co-founded Accenture’s Sustainability Strategy practice in 2008. Led both
Europe and Asia Pacific regions before global lead role
McKinsey & Company
Senior Advisor
February 2006 - March 2007 (1 year 2 months)
Strategy Practice / Business in Society
ABIS - The Academy of Business in Society
Joint Founding Executive Director
February 2003 - February 2007 (4 years 1 month)
Brussels, Belgium
Accenture
Strategy Consulting
August 2000 - January 2003 (2 years 6 months)
2000-2003 
Andersen Consulting Summer Vacation Placement - 2000 & 2001
Accenture Graduate Training Programme - 2002-3
 Page 3 of 4
 
Costa Rica renewables project, The British Institute, Lastorders.com,
lastminute.com, Ford Motor Co.
Gap Year Roles & While Studying
February 1996 - February 2000 (4 years 1 month)
London, United Kingdom & Costa Rica
Education
University of Oxford
Business Fellow, Smith School of Environment & Economics, Business,
Economics & Policy · (2010 - 2020)
Harvard University, John F. Kennedy School of Government
Global Leadership and Public Policy for the 21st Century · (2014 - 2014)
Yale University
Global Issues & Foundations for Leadership in the 21st
Century · (2013 - 2013)
University of Oxford
Oxford University Business Economics Programme
(OUBEP), Economics · (2011 - 2011)
INSEAD
International Executive Programme, IEP · (2007 - 2007)
 Page 4 of 4)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(61).pdf: Contact
www.linkedin.com/in/
chriscapossela (LinkedIn)
Top Skills
SaaS
Cloud Computing
Enterprise Software
Chris Capossela
EVP & Chief Marketing Officer at Microsoft
Seattle, Washington, United States
Summary
As executive vice president and chief marketing officer, I run
worldwide marketing across both the consumer and commercial
businesses, which includes product marketing, business planning,
brand, advertising, events, media buying, communications and
market research for Microsoft products and services. I also run digital
direct sales and retail partner sales for all Microsoft products. My
team is committed to advancing Microsoft’s mission to empower
every person and every organization on the planet to achieve more,
and it's an honor to learn from them every day. 
My passion for technology and computing began when I was a kid
and wrote a reservation system for our small, family Italian restaurant
in Boston using dBASE for DOS on an early IBM PC.
Experience
Microsoft
31 years 8 months
Executive Vice President and Chief Marketing Officer
March 2014 - Present (9 years)
As executive vice president and chief marketing officer, I run worldwide
marketing across both the consumer and commercial businesses, which
includes product marketing, business planning, brand, advertising, events,
media buying, communications and market research for Microsoft products
and services. I also lead digital direct sales and retail partner sales for all
Microsoft products.
Corporate Vice President, Consumer Channels Group
July 2013 - July 2014 (1 year 1 month)
As the worldwide leader of the Consumer Channels Group, I led sales and
marketing activities with OEM, telco and retail partners. During my tenure,
I oversaw the creation of new business opportunities to deliver amazing
consumer experiences through Windows, Windows Phone, Office, and Xbox.
 Page 1 of 3
https://www.linkedin.com/in/chriscapossela?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BSFp%2FesfHTgeBZXXq8OH8WQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/chriscapossela?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BSFp%2FesfHTgeBZXXq8OH8WQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Chief Marketing Officer
April 2011 - July 2013 (2 years 4 months)
Greater Seattle Area
In my role leading the Central Marketing Group, I led advertising, brand,
research, and communications for Microsoft's products and services. Also
during this timeframe I created the Consumer Channels Group from the
ground up to drive sales and marketing for our consumer products with our
retail, operator, and OEM partners.
Corporate Vice President, Office Marketing
December 2003 - April 2011 (7 years 5 months)
Greater Seattle Area
As Corporate Vice President for the Microsoft Office Division, I was
responsible for marketing the company’s productivity solutions including
Microsoft Office, Office 365, SharePoint, Exchange, Lync, Project, and Visio.
General Manager, Microsoft Project
November 2001 - November 2003 (2 years 1 month)
Greater Seattle Area
Director, Business Operations, EMEA
September 1999 - October 2001 (2 years 2 months)
Speech Assistant to Bill Gates
March 1997 - September 1999 (2 years 7 months)
Greater Seattle Area
Program Manager, Microsoft Access
January 1994 - March 1997 (3 years 3 months)
Greater Seattle Area
Product Manager, Desktop Database Products
August 1992 - January 1994 (1 year 6 months)
Greater Seattle Area
Marketing Manager, Windows Seminar Team
July 1991 - August 1992 (1 year 2 months)
Greater Seattle Area
Education
Harvard University
BA, Economics · (1987 - 1991)
 Page 2 of 3
 
Buckingham Browne & Nichols
High School · (1985 - 1987)
 Page 3 of 3)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(30).pdf: Contact
www.linkedin.com/in/
stephanieajamisonaccenture
(LinkedIn)
www.accenture.com/us-en/about/
leadership/stephanie-jamison
(Company)
Top Skills
Change Management
Management Consulting
Business Process
Stephanie Jamison
Global Energy and Materials Lead at Accenture; Global Management
Committee Member
London, England, United Kingdom
Summary
As Accenture’s Strategy and Consulting Lead for Europe, I head
up the talented team that helps Europe’s leading organisations
accelerate their digital transformation. This includes strategists,
consultants, industry and function experts as well as data scientists
and human performance professionals. 
At the heart of my career is a passion for sustainability and purpose,
backed by over two decades of practice. We live in an exciting time
for the future of the planet as countries which represent 70% of
global GDP commit to a future of net-zero emissions. We must all
unite in the effort to reach those targets. 
Before my current role, I led Accenture’s global Utilities industry
sector and worked with some of the most innovative utilities in North
America and Europe. This role also enabled me to work with the
World Economic Forum on projects including the COVID-19 Action
Group on the Future of Utilities. 
I am a champion of personal growth and value the freedom to
innovate, take risks and learn. I am active in diversity networks and
am an International Women's Forum Fellow. 
I started my career in the USA and now live in London with my
husband and two children. 
I am keen to learn and exchange ideas. Contact me here on
LinkedIn.
Experience
Accenture
27 years
 Page 1 of 3
https://www.linkedin.com/in/stephanieajamisonaccenture?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BIMWh5KgmRVSns4St5LpmhQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/stephanieajamisonaccenture?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BIMWh5KgmRVSns4St5LpmhQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/stephanieajamisonaccenture?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BIMWh5KgmRVSns4St5LpmhQ%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.accenture.com/us-en/about/leadership/stephanie-jamison
https://www.accenture.com/us-en/about/leadership/stephanie-jamison
https://www.accenture.com/us-en/about/leadership/stephanie-jamison
 
Senior Managing Director
September 2016 - Present (6 years 6 months)
London
SENIOR MANAGING DIRECTOR - STRATEGY & CONSULTING LEAD FOR
EUROPE
2021/September - Present
I work with C-suite executives and boards of Europe’s leading organizations to
help them accelerate their digital transformation to enhance competitiveness,
grow profitability and deliver sustainable value. In this role I lead a team
of talented strategists and consultants, industry and function experts, data
scientists and human performance professionals – with a collective mission to
help Accenture’s clients in Europe apply data, analytics, artificial intelligence,
assets and innovation to deliver business outcomes at speed and scale.
SENIOR MANAGING DIRECTOR – GLOBAL INDUSTRY LEAD, UTILITIES 
2020/March – 2021/August
I lead Accenture’s global Utilities business. I am responsible for the strategy
of the practice and for its comprehensive portfolio of services, which extend
across the utility value chain)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile%20(27).pdf: Key projects include Analytics, new Digital Channels, salesforce automation
& improvement, ZBB, ZBO, and GBS strategy. In addition, I have played
a leadership role within our Global Consumer Goods practice, driving our
thought leadership.
 Page 2 of 4
 
Prior to this, I worked in Duesseldorf, Germany for 7 years. During this time,
my focus was on CPG and Retail clients. I led the delivery of large scale
Change programs (such as ZBB initiatives, GBS transition, and BPO) as well
as IT Transformation Projects (eCommerce, SAP, assortment management).
Most notable achievements include delivering 100M EUR+ cost savings,
transitioning a 300+ F&A BPO engagement, and creating new platform to
deliver a new business channel (eCommerce).
Before moving to Germany, I was based in the UK, and worked across a
multitude of CPG & Retail clients and projects.
In summary, I have a strong business consulting background, with experience
in strategy, process and change management, IT & ERP implementation,
Global Business Services (GBS) and BPO, all across multiple functional
domains. As an international leader, my strength is to bring together diverse
teams to work together towards a common goal, keeping them motivated with
my energy, passion, as well as holding them to account with my commitment
to rigor. My aim is to create energized teams, and inspire, especially women.
Selfridges
Retail Operations Manager
2005 - 2007 (2 years)
I was responsible for running the flagship store’s Retail Operations across
the Homewares, Furniture, Travel, Restaurants & Bars categories, across a
business sized +$200M. 
I held total general management responsibility for the operations which
included delivering financial results, recruiting and developing 7+ direct
reports, leading and communicating with a team of over 500 people, as well as
driving improvements in retail standards. 
In addition, l led a number of key strategic initiatives, including a number of
customer service transformation projects, as well as change programs within
my area of the business.
Education
University of Leeds
 Page 3 of 4
 
Economics & Politics with North American Studies · (1994 - 1999)
University of Toronto
BA Hons Economics & Politics & NA Studies 
 Page 4 of 4)
https://synpasedlstore.blob.core.windows.net/acccsuite/Profile.pdf: Contact
www.linkedin.com/in/karthik-narain
(LinkedIn)
Top Skills
IT Strategy
Leadership
Strategy
Karthik Narain
Lead - Accenture Cloud First
San Francisco Bay Area
Summary
Karthik Narain leads Accenture Cloud First, with responsibility for
helping clients shape, move and operate their business in the cloud
to accelerate innovation and achieve their digital transformation
goals. He is also a member of the company’s Global Management
Committee.
Accenture Cloud First is a multi-service group that provides the
full stack of cloud services spanning strategy, industry-specific
cloud journeys, cloud data and AI, migration and modernization,
and secure cloud management, as well as change and talent
management for continuous innovation. In his role leading Accenture
Cloud First, Karthik is focused on extending the company’s
leadership in Cloud with Accenture’s ecosystem partners and
through investments in deep industry capabilities and solutions,
cloud acquisitions, and talent.
A technology industry veteran, Karthik most recently served as the
lead for Accenture Technology in North America, helping guide
Global 2000 brands in using the power of the cloud and other
technologies to transform their businesses. Over his 20-year career,
he has led many innovative technology programs for clients across
a variety of industry sectors, including Financial Services, High Tech
and Software and Platforms. Karthik also previously led Technology
services for Accenture’s Communications, Media and High-Tech
industry segments.
Prior to joining Accenture in 2015, Karthik led the High-Tech
business for HCL Technologies where he was responsible for some
of HCL’s marquee client relationships.
Karthik holds a bachelor’s degree in Commerce and Management
from the University of Madras and a master’s degree in Computer
Science and Management from Bharathidasan University. He is
based in Silicon Valley.
 Page 1 of 3
https://www.linkedin.com/in/karthik-narain?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BGTaDzsgRSJ%2B%2F%2BpJ2sOfRyg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
https://www.linkedin.com/in/karthik-narain?jobid=1234&lipi=urn%3Ali%3Apage%3Ad_jobs_easyapply_pdfgenresume%3BGTaDzsgRSJ%2B%2F%2BpJ2sOfRyg%3D%3D&licu=urn%3Ali%3Acontrol%3Ad_jobs_easyapply_pdfgenresume-v02_profile
 
Experience
Accenture
8 years 2 months
Lead - Accenture Cloud First
October 2020 - Present (2 years 5 months)
Lead – Technology, North America 
March 2020 - October 2020 (8 months)
Senior Managing Director and Global Technology Lead -
Communications, Media and High Tech
June 2017 - March 2020 (2 years 10 months)
Accelerating the rotation to Digital / NEW through Accenture's Intelligent
Platforms strategy (SAP, Oracle, SFDC, Microsoft and Workday), Cloud
Adoption (AWS, GCP, Azure, Ali Cloud) by creating a Digitally Decoupled
enterprise. Exciting, isn't it! 
Managing Director, NA Technology Lead for Communications, Media
and High Tech
January 2015 - May 2017 (2 years 5 months)
San Francisco Bay Area
Responsible for Accenture's Technology Services business for Comms, Media,
High Tech and A&D in NA
HCL Technologies
Vice President and Vertical Head, High Tech and Automotive
June 2005 - January 2015 (9 years 8 months)
San Francisco Bay Area
P&L responsibility for HCL's High Tech and Automotive vertical segments.
Infosys
Senior Relationship Manager - Financial Services
June 2000 - June 2005 (5 years 1 month)
Greater Minneapolis-St. Paul Area
Responsible for managing and delivering complex programs for a large capital
markets client.
Education
 Page 2 of 3
 
Bharathidasan University, Tiruchirappalli
Masters in Computer systems and Business, Computer Science
 Page 3 of 3)
Poll 3: in_progress
Poll 4: in_progress
Poll 5: in_progress
Poll 6: in_progress
Poll 7: in_progress
Poll 8: in_progress
Poll 9: in_progress


CONVERSATION:
User query:
List top 5 candidates for CEO role with details?
Assistant response:
The
The
```