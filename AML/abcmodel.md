# Acitvity based Costing Modelling using Deep Learning

## Introduction

- Activity Based Costing (ABC) is a methodology that identifies activities in an organization and assigns the cost of each activity resource to all products and services according to the actual consumption by each.
- How to target the right skills and resources for a project.
- Based on their availability
- Based on their experience and skills based on requirements
- Data is key to this analysis

## Pre-requisites

- Azure subscription
- Azure Machine learning workspace
- Azure storage
- Create data based on this code: https://github.com/balakreshnan/Samples2024/blob/main/AML/abcdataset.md

## Steps

- install necessay libraries

```
%pip install tensorflow
```

- import the libraries

```
import pandas as pd  
import numpy as np  
from sklearn.model_selection import train_test_split  
from sklearn.preprocessing import StandardScaler, LabelEncoder  

from sklearn.preprocessing import MultiLabelBinarizer, StandardScaler 
```

- Load the data

```
# Load datasets  
# Load datasets  
resource_skills = pd.read_csv('resource_skills.csv')  
resource_availability = pd.read_csv('resource_availability.csv')  
project_requirements = pd.read_csv('project_requirements.csv')  
resource_preferences = pd.read_csv('resource_preferences.csv')  
resource_costs = pd.read_csv('resource_costs.csv')   
  
# Combine dataframes  
df = pd.merge(resource_skills, resource_availability, on='Resource ID')  
df = pd.merge(df, resource_preferences, on='Resource ID')  
df = pd.merge(df, resource_costs, on='Resource ID')  
```

- replace unknown data

```
replace_value = 'Missing'
df['Certifications'] = df['Certifications'].fillna(replace_value)
```

- one hot encoding

```
# MultiLabelBinarizer for multi-hot encoding  
mlb = MultiLabelBinarizer() 
```

- Apply one hot encoding

```
# Apply multi-hot encoding to skills and other multi-label categorical features  
df['Skills'] = df['Skills'].apply(lambda x: x.split(', '))  
df['Certifications'] = df['Certifications'].apply(lambda x: x.split(', '))  
df['Previous Projects'] = df['Previous Projects'].apply(lambda x: x.split(', '))  
df['Domain Expertise'] = df['Domain Expertise'].apply(lambda x: x.split(', '))  
df['Preferred Project Types'] = df['Preferred Project Types'].apply(lambda x: x.split(', ')) 

skills_encoded = mlb.fit_transform(df['Skills'])  
skills_df = pd.DataFrame(skills_encoded, columns=mlb.classes_) 
```

- concate encoded data

```
# Concatenate encoded features back to the original dataframe  
df = df.drop(columns=['Skills'])  
df = pd.concat([df, skills_df], axis=1) 
```

- encode certifications

```
# Repeat for other multi-label categorical features as needed  
certifications_encoded = mlb.fit_transform(df['Certifications'])  
certifications_df = pd.DataFrame(certifications_encoded, columns=mlb.classes_)  
  
df = df.drop(columns=['Certifications'])  
df = pd.concat([df, certifications_df], axis=1) 
```

- encode project expertise

```
previous_projects_encoded = mlb.fit_transform(df['Previous Projects'])  
previous_projects_df = pd.DataFrame(previous_projects_encoded, columns=mlb.classes_)  
  
df = df.drop(columns=['Previous Projects'])  
df = pd.concat([df, previous_projects_df], axis=1)
```

- encode domain expertise

```
domain_expertise_encoded = mlb.fit_transform(df['Domain Expertise'])  
domain_expertise_df = pd.DataFrame(domain_expertise_encoded, columns=mlb.classes_)  
  
df = df.drop(columns=['Domain Expertise'])  
df = pd.concat([df, domain_expertise_df], axis=1) 
```

- enode preferred project types

```
preferred_project_types_encoded = mlb.fit_transform(df['Preferred Project Types'])  
preferred_project_types_df = pd.DataFrame(preferred_project_types_encoded, columns=mlb.classes_)  
  
df = df.drop(columns=['Preferred Project Types'])  
df = pd.concat([df, preferred_project_types_df], axis=1)  
```

- handle experience

```
# Handle Experience Level, Preferred Working Hours, and Geographic Constraints  
df['Experience Level'] = df['Experience Level'].map({'Junior': 0, 'Senior': 1})  
df['Preferred Working Hours'] = df['Preferred Working Hours'].map({'9am-5pm': 0, '8am-4pm': 1, 'Flexible': 2})  
df['Geographic Constraints'] = df['Geographic Constraints'].map({'None': 0})  
```

- Normalize the data


```
# Normalize numerical data  
scaler = StandardScaler()  
df[['Available Hours/Week', 'Planned Leaves', 'Cost per Hour']] = scaler.fit_transform(df[['Available Hours/Week', 'Planned Leaves', 'Cost per Hour']])  
  
# Prepare project requirements  
project_requirements['Required Skills'] = project_requirements['Required Skills'].apply(lambda x: x.split(', '))  
required_skills_encoded = mlb.transform(project_requirements['Required Skills'])  
required_skills_df = pd.DataFrame(required_skills_encoded, columns=mlb.classes_)  
  
# Concatenate encoded features back to the original dataframe  
project_requirements = project_requirements.drop(columns=['Required Skills'])  
project_requirements = pd.concat([project_requirements, required_skills_df], axis=1)  
  
# Prepare target variable  
project_requirements['Target'] = required_skills_encoded.argmax(axis=1) 
```

- Split the data

```
# Split data into train and test sets  
X = df.drop(columns=['Resource ID'])  
y = project_requirements['Target']  
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

- Build the model

```
import torch  
import torch.nn as nn  
import torch.optim as optim  
from torch.utils.data import DataLoader, TensorDataset  
  
class ResourceAssignmentModel(nn.Module):  
    def __init__(self, input_dim, num_classes):  
        super(ResourceAssignmentModel, self).__init__()  
        self.fc1 = nn.Linear(input_dim, 64)  
        self.fc2 = nn.Linear(64, 32)  
        self.fc3 = nn.Linear(32, num_classes)  
          
    def forward(self, x):  
        x = torch.relu(self.fc1(x))  
        x = torch.relu(self.fc2(x))  
        x = self.fc3(x)  # No activation function here because CrossEntropyLoss expects raw logits  
        return x  
  
# Convert data to tensors  
X_train_tensor = torch.tensor(X_train.values, dtype=torch.float32)  
y_train_tensor = torch.tensor(y_train.values, dtype=torch.long)  # Long tensor for class labels  
X_test_tensor = torch.tensor(X_test.values, dtype=torch.float32)  
y_test_tensor = torch.tensor(y_test.values, dtype=torch.long)  # Long tensor for class labels  
  
# Create DataLoader  
train_dataset = TensorDataset(X_train_tensor, y_train_tensor)  
train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)  
  
# Initialize model, loss function, and optimizer  
input_dim = X_train.shape[1]  
num_classes = len(np.unique(y_train))  
model = ResourceAssignmentModel(input_dim, num_classes)  
criterion = nn.CrossEntropyLoss()  
optimizer = optim.Adam(model.parameters(), lr=0.001)  
```

- Train the model

```
# Training loop  
num_epochs = 500  
model.train()  
for epoch in range(num_epochs):  
    for X_batch, y_batch in train_loader:  
        # Forward pass  
        y_pred = model(X_batch)  
        loss = criterion(y_pred, y_batch)  
          
        # Backward pass and optimization  
        optimizer.zero_grad()  
        loss.backward()  
        optimizer.step()  
      
    if (epoch + 1) % 10 == 0:  
        print(f'Epoch [{epoch + 1}/{num_epochs}], Loss: {loss.item():.4f}')  

```

- Evaluation

```
# Evaluation  
model.eval()  
with torch.no_grad():  
    y_pred = model(X_test_tensor)  
    _, predicted = torch.max(y_pred, 1)  
    accuracy = (predicted == y_test_tensor).sum().item() / y_test_tensor.size(0)  
    print(f'Accuracy: {accuracy:.4f}')  
```

- Save the model

```
# Save the model  
torch.save(model.state_dict(), 'resource_assignment_model.pth')
```