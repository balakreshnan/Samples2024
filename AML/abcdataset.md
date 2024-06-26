# ABC - Activity Based Costing Synthetic data generation

## Description

- Create data set for Activity Based Costing (ABC) analysis.
- We need to create 5 different data sets as per the code
- Skills, resource, project and resource availability data sets
- We are using faker to generate the data

## Pre-requisites

- Azure subscription
- Azure Machine Learning workspace
- Azure Open AI workspace
- Python environment

## Steps

- install libraries

```
%pip install faker
```

- import the libraries

```
import pandas as pd
from faker import Faker
import random
from datetime import datetime, timedelta
 
# Initialize Faker generator for fake data
fake = Faker()
```

- Create Resources data set
- Functions to create various data sets

```
start_date = datetime.strptime('2024-07-01', '%Y-%m-%d')
  
# Constants  
NUM_RESOURCES = 2000  
NUM_PROJECTS = 2000  
  
# Helper functions  
def generate_skills():  
    skills = ['Python', 'Java', 'SQL', 'Machine Learning', 'Project Management',   
              'Agile', 'JavaScript', 'React', 'Node.js', 'C#', '.NET', 'Spring Framework']  
    return ', '.join(random.sample(skills, random.randint(1, 3)))  
  
def generate_certifications():  
    certs = ['AWS Certified Solutions Architect', 'Oracle Certified Java Developer',   
             'PMP', 'Scrum Master', 'Microsoft Certified: Azure Developer Associate',   
             'Microsoft Certified: Azure Solutions Architect']  
    return ', '.join(random.sample(certs, random.randint(0, 2)))  
  
def generate_experience_level():  
    levels = ['Junior', 'Mid-Level', 'Senior']  
    return random.choice(levels)  
  
def generate_previous_projects():  
    return ', '.join([f'Project {fake.random_uppercase_letter()}' for _ in range(random.randint(1, 3))])  
  
def generate_domain_expertise():  
    domains = ['Healthcare', 'Finance', 'Retail', 'Logistics', 'IT Services', 'E-commerce', 'Education']  
    return ', '.join(random.sample(domains, random.randint(1, 2)))  
  
def generate_availability():  
    return random.randint(20, 40)  
  
def generate_planned_leaves():  
    return random.choice([0, 0, 0, random.randint(1, 10)])  # More likely to have 0 leaves  
  
def generate_required_skills():  
    return ', '.join(random.sample(['Python', 'Java', 'Spring Framework', 'Machine Learning',   
                                    'Project Management', 'Agile', 'JavaScript', 'React',   
                                    'Node.js', 'C#', '.NET'], random.randint(1, 3)))  
  
def generate_estimated_effort():  
    return random.randint(50, 300)  
  
def generate_dates():  
    start_date = fake.date_between(start_date=datetime.strptime('2024-07-01','%Y-%m-%d'), end_date=datetime.strptime('2024-07-31','%Y-%m-%d'))  
    end_date = fake.date_between(start_date=start_date, end_date=datetime.strptime('2024-12-31','%Y-%m-%d'))  
    return start_date, end_date  
  
def generate_preferred_project_types():  
    return ', '.join(random.sample(['Healthcare', 'Finance', 'Retail', 'Logistics', 'IT Services',   
                                    'E-commerce', 'Education'], random.randint(1, 2)))  
  
def generate_preferred_working_hours():  
    return random.choice(['9am-5pm', '8am-4pm', '10am-6pm', 'Flexible'])  
  
def generate_cost():  
    return random.randint(50, 150)  
```

- Now let's create the data set

```
# Data generation  
resources = []  
for i in range(NUM_RESOURCES):  
    resource_id = f'R{i+1:04d}'  
    resources.append({  
        'Resource ID': resource_id,  
        'Skills': generate_skills(),  
        'Certifications': generate_certifications(),  
        'Experience Level': generate_experience_level(),  
        'Previous Projects': generate_previous_projects(),  
        'Domain Expertise': generate_domain_expertise()  
    })  
  
availability = []  
for i in range(NUM_RESOURCES):  
    resource_id = f'R{i+1:04d}'  
    availability.append({  
        'Resource ID': resource_id,  
        'Available Hours/Week': generate_availability(),  
        'Planned Leaves': generate_planned_leaves()  
    })  
  
projects = []  
for i in range(NUM_PROJECTS):  
    project_id = f'P{i+1:04d}'  
    start_date, end_date = generate_dates()  
    projects.append({  
        'Project ID': project_id,  
        'Required Skills': generate_required_skills(),  
        'Estimated Effort': generate_estimated_effort(),  
        'Start Date': start_date.isoformat(),  
        'End Date': end_date.isoformat()  
    })  
  
preferences = []  
for i in range(NUM_RESOURCES):  
    resource_id = f'R{i+1:04d}'  
    preferences.append({  
        'Resource ID': resource_id,  
        'Preferred Project Types': generate_preferred_project_types(),  
        'Preferred Working Hours': generate_preferred_working_hours(),  
        'Geographic Constraints': 'None'  
    })  
  
costs = []  
for i in range(NUM_RESOURCES):  
    resource_id = f'R{i+1:04d}'  
    costs.append({  
        'Resource ID': resource_id,  
        'Cost per Hour': generate_cost()  
    })  
```

- convert to dataframes and save as csv files

```
# Convert to DataFrames  
df_resources = pd.DataFrame(resources)  
df_availability = pd.DataFrame(availability)  
df_projects = pd.DataFrame(projects)  
df_preferences = pd.DataFrame(preferences)  
df_costs = pd.DataFrame(costs)  
  
# Save to CSV files  
df_resources.to_csv('resource_skills.csv', index=False)  
df_availability.to_csv('resource_availability.csv', index=False)  
df_projects.to_csv('project_requirements.csv', index=False)  
df_preferences.to_csv('resource_preferences.csv', index=False)  
df_costs.to_csv('resource_costs.csv', index=False)  
  
print("Data generation complete. CSV files have been saved.") 
```

- Now you should have about 5 different csv files with the data sets