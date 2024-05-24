# Data Science and Feature Engineering and Machine Learning model training

## Data Science

### Data Science Process

1. Define the problem
2. Collect data
3. Data cleaning
4. Data exploration
5. Feature engineering
6. Model selection
7. Model training
8. Model evaluation
9. Model deployment
10. Model monitoring
11. Model retraining
12. Model update
13. Model deprecation
14. Model retirement

### Data Science Tools

1. Python
2. Xgboost
3. Pytorch
4. scikit-learn
5. SeaBorn

### Code

- Now import the necessary libraries

```python
import pandas as pd
import torch
import torch.nn as nn
import torch.optim as optim
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, mean_squared_error
import numpy as np  
import matplotlib.pyplot as plt  
import seaborn as sns  
  
# Sample data creation  
np.random.seed(42)  

import pandas as pd  
from sklearn.model_selection import train_test_split  
from sklearn.linear_model import LinearRegression  
from sklearn.metrics import mean_squared_error, r2_score  
```

- Now load the data

```
df = pd.read_csv('EAC.csv')
```

### Feature Engineering

- Now let's do some feature engineering

```
# Get the distinct count of values for a specific column
department_count = df['Account Desc'].nunique()

print(f"\nDistinct count of 'Account Desc': {department_count}")
```

- understanding distinct values in a column

```
df['Account Desc'].value_counts
```

- Check the distinct values in a column

```
# Get the distinct count of values for each column
distinct_counts = df.nunique()

print("\nDistinct count of values for each column:")
print(distinct_counts)
```

- now pick only columns to work

```
dataplot = data[['Original Budget', 'Change Orders', 'Revised Budget', 'Actuals To Date', 'Remaining Commited', 'ETC', 'EAC']] 
```

- let's build some heatmaps

```
plt.figure(figsize=(10, 8))  
sns.heatmap(dataplot.corr(), annot=True, cmap='coolwarm', fmt=".2f")  
plt.title('Correlation Matrix')  
plt.show()  
```

- Creating column

```
dataplot['Utilization Ratio'] = dataplot['Actuals To Date'] / dataplot['Revised Budget']  
dataplot['Log Revised Budget'] = np.log1p(dataplot['Revised Budget'])  
```

- scatter

```
data = pd.read_csv('EAC.csv')  
dataplot = data[['Original Budget', 'Change Orders', 'Revised Budget', 'Actuals To Date', 'Remaining Commited', 'ETC', 'EAC', 'PM Forecast']] 

plt.figure(figsize=(10, 6))  
sns.scatterplot(x=dataplot['Revised Budget'], y=dataplot['PM Forecast'])  
plt.title('Scatter Plot of Revised Budget vs. PM Forecast')  
plt.xlabel('Revised Budget')  
plt.ylabel('PM Forecast')  
plt.show()  
```

- Split the data

```
# Selecting features - here we use several columns that might be predictors  
features = data[['Year', 'Period', 'Project', 'Seq', 'Original Budget', 'Change Orders', 'Revised Budget', 'Actuals To Date', 'Remaining Commited', 'ETC', 'EAC']]  
# Target variable  
target = data['PM Forecast']  
  
# Splitting data into training and testing sets  
X_train, X_test, y_train, y_test = train_test_split(features, target, test_size=0.2, random_state=42)  
```

- Train the model

```
# Create a linear regression model  
model = LinearRegression()  
  
# Train the model  
model.fit(X_train, y_train)  
```

- validating the model

```
# Predict the PM Forecast on the testing set  
y_pred = model.predict(X_test)  
  
# Calculate the Mean Squared Error and R2 Score  
mse = mean_squared_error(y_test, y_pred)  
r2 = r2_score(y_test, y_pred)  
  
print(f"Mean Squared Error: {mse}")  
print(f"R2 Score: {r2}")  
```

- Visualizing the model

```
# Visualize actual vs predicted costs
plt.figure(figsize=(10, 6))
sns.scatterplot(x=y_test, y=y_pred)
plt.plot([y_test.min(), y_test.max()], [y_test.min(), y_test.max()], 'k--', lw=2)
plt.xlabel('Actual Project Costs')
plt.ylabel('Predicted Project Costs')
plt.title('Actual vs Predicted Project Costs')
plt.show()
```

- visulaze the model

```
# Visualize residuals
residuals = y_test - y_pred
plt.figure(figsize=(10, 6))
sns.histplot(residuals, kde=True)
plt.xlabel('Residuals')
plt.ylabel('Frequency')
plt.title('Distribution of Residuals')
plt.show()
```

- Feature importance

```
# Visualize feature importances (if using a model that supports it)
# For Linear Regression, coefficients represent feature importance
importance = pd.Series(model.coef_, index=features.columns)
importance = importance.sort_values(ascending=False)

plt.figure(figsize=(12, 8))
sns.barplot(x=importance, y=importance.index)
plt.xlabel('Coefficient Value')
plt.ylabel('Feature')
plt.title('Feature Importance')
plt.show()
```

- Xgboost

```
from xgboost import XGBRegressor

# Initialize the model
model = XGBRegressor(n_estimators=100, learning_rate=0.1, max_depth=3, random_state=42)

# Train the model
model.fit(X_train, y_train)

# Predict and evaluate
y_pred = model.predict(X_test)
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)

print(f"XGBoost - MAE: {mae}, MSE: {mse}, RMSE: {rmse}")
```

- predicting

```
# Example: Predicting PM Forecast for a new data point  
new_data = pd.DataFrame({  
    'Year': [24],
    'Period': [6],
    'Project': [2482812500],
    'Seq': [23],
    'Original Budget': [3000000],  
    'Change Orders': [500000],  
    'Revised Budget': [3500000],  
    'Actuals To Date': [2000000],  
    'Remaining Commited': [100000],  
    'ETC': [-1500000],  
    'EAC': [2000000]  
})  
  
new_prediction = model.predict(new_data)  
print(f"Predicted PM Forecast: {new_prediction[0]}") 
```

- Gradient descent

```
from sklearn.ensemble import GradientBoostingRegressor

# Initialize the model
model = GradientBoostingRegressor(n_estimators=100, learning_rate=0.1, max_depth=3, random_state=42)

# Train the model
model.fit(X_train, y_train)

# Predict and evaluate
y_pred = model.predict(X_test)
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)

print(f"Gradient Boosting - MAE: {mae}, MSE: {mse}, RMSE: {rmse}")
```

### LSTM

- conver to tensors

```
# Convert data to PyTorch tensors
X_train = torch.tensor(X_train.to_numpy(), dtype=torch.float32)
y_train = torch.tensor(y_train.values, dtype=torch.float32).view(-1, 1)
X_test = torch.tensor(X_test.to_numpy(), dtype=torch.float32)
y_test = torch.tensor(y_test.values, dtype=torch.float32).view(-1, 1)
```

- Define the Neural Network

```
# Define LSTM model
class LSTMRegressor(nn.Module):
    def __init__(self, input_size, hidden_size, num_layers, output_size):
        super(LSTMRegressor, self).__init__()
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        self.lstm = nn.LSTM(input_size, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        c0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        out, _ = self.lstm(x, (h0, c0))
        out = self.fc(out[:, -1, :])
        return out
```

- Model parameters

```
# Model parameters
input_size = X_train.shape[1]
hidden_size = 50
num_layers = 2
output_size = 1
```

- Initialize the model

```
model = LSTMRegressor(input_size, hidden_size, num_layers, output_size)
```

- Check the error.

```
# Loss and optimizer
criterion = nn.MSELoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Reshape input for LSTM [batch_size, sequence_length, input_size]
X_train = X_train.unsqueeze(1)
X_test = X_test.unsqueeze(1)
```

- Train the model

```
# Training the model
num_epochs = 100
model.train()
for epoch in range(num_epochs):
    outputs = model(X_train)
    optimizer.zero_grad()
    loss = criterion(outputs, y_train)
    loss.backward()
    optimizer.step()
    
    if (epoch+1) % 10 == 0:
        print(f'Epoch [{epoch+1}/{num_epochs}], Loss: {loss.item():.4f}')
```

- Evaluation

```
# Evaluation
model.eval()
with torch.no_grad():
    y_pred = model(X_test)

# Convert predictions to numpy for evaluation
y_pred = y_pred.numpy()
y_test = y_test.numpy()

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)

print(f'LSTM Regressor - MAE: {mae}, MSE: {mse}, RMSE: {rmse}')
```

- Visualize the model

```
# Visualize residuals
import matplotlib.pyplot as plt
import seaborn as sns

residuals = y_test - y_pred

plt.figure(figsize=(10, 6))
sns.histplot(residuals, kde=True, bins=30)
plt.xlabel('Residuals')
plt.ylabel('Frequency')
plt.title('Distribution of Residuals')
plt.show()
```

- Sample with Random forest

```
from sklearn.ensemble import RandomForestRegressor

# Initialize the model
model = RandomForestRegressor(n_estimators=100, random_state=42)

# Train the model
model.fit(X_train, y_train)

# Predict and evaluate
y_pred = model.predict(X_test)
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = mean_squared_error(y_test, y_pred, squared=False)

print(f"Random Forest - MAE: {mae}, MSE: {mse}, RMSE: {rmse}")
```

