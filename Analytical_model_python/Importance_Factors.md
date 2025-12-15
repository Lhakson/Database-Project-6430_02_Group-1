import pyodbc
import pandas as pd


```python
CONN_STRING = (
    r'Driver={ODBC Driver 17 for SQL Server};'
    r'Server=LHAKSON\SQLSERVER2022;' 
    r'Database=Port_Authority_Bus_Database;'
    r'Trusted_Connection=yes;'   
)
try:

    cnxn = pyodbc.connect(CONN_STRING)
    print("Connection established successfully!")

except pyodbc.Error as ex:
    sqlstate = ex.args[0]
    print(f"Error connecting: {sqlstate}")
```

    Connection established successfully!
    


```python
import numpy as np
import statsmodels.api as sm
from sklearn.preprocessing import StandardScaler
from statsmodels.stats.outliers_influence import variance_inflation_factor
```


```python
np.random.seed(42)
N = 10000
```


```python
df = pd.DataFrame({
    'DeparturePassengerCount': np.random.randint(5000, 50000, N), 
    'Month': np.random.randint(1, 13, N),
    'DayOfWeek': np.random.choice(['Mon', 'Fri', 'Sun', 'Tues', 'Weds', 'Thurs', 'Sat'], N),
    'IsHoliday': np.random.choice([0, 1], N, p=[0.95, 0.05]), # DimDate
    'CarrierType': np.random.choice(['Commuter', 'Intercity'], N, p=[0.7, 0.3]), # DimCarrier
    'AvgTemperature_F': np.random.uniform(20, 90, N), # DimWeather
    'Snowfall_in': np.random.uniform(0, 10, N), # DimWeather
    'Precipitation_in': np.random.uniform(0, 2, N), # DimWeather
    'WindSpeed_in': np.random.uniform(5, 30, N), # DimWeather
})

# Define Target (Y) and Predictors (X)
Y = df['DeparturePassengerCount']
X = df.drop('DeparturePassengerCount', axis=1)

print("Data loaded and split into X (Features) and Y (Target).")
```

    Data loaded and split into X (Features) and Y (Target).
    


```python

categorical_cols = ['Month', 'DayOfWeek', 'CarrierType']


X['Month'] = X['Month'].astype(str)


X_encoded = pd.get_dummies(X, columns=categorical_cols, drop_first=True)

# Numerical columns for scaling
scaling_cols = ['AvgTemperature_F', 'Snowfall_in', 'Precipitation_in', 'WindSpeed_in']

print("Categorical features encoded.")
```

    Categorical features encoded.
    


```python

scaler = StandardScaler()


X_scaled_numerical = scaler.fit_transform(X_encoded[scaling_cols])

X_scaled_numerical_df = pd.DataFrame(
    X_scaled_numerical, 
    columns=scaling_cols, 
    index=X_encoded.index
)


X_final = pd.concat([
    X_scaled_numerical_df, 
    X_encoded.drop(scaling_cols, axis=1)
], axis=1)

print("Numerical features scaled.")
```

    Numerical features scaled.
    


```python

X_final_ols = sm.add_constant(X_final, has_constant='add') 


X_final_ols = X_final_ols.astype(float) 
Y = Y.astype(float)  


X_final_ols = X_final_ols.fillna(0) 
Y = Y.fillna(Y.mean()) 

# Fit the OLS model
model = sm.OLS(Y, X_final_ols).fit()

# --- Determine Factor Importance ---

importance_df = model.params.drop('const').abs().sort_values(ascending=False).to_frame(name='Importance (Abs. Coeff)')


original_coeffs = model.params.drop('const').to_frame(name='Original Coeff')


importance_df = importance_df.merge(original_coeffs, left_index=True, right_index=True)
importance_df['Impact'] = np.where(importance_df['Original Coeff'] > 0, 'Positive', 'Negative')

print("\n--- Top 10 Most Important Prediction Factors (Goal 2) ---")
print(importance_df.head(10))
```

    
    --- Top 10 Most Important Prediction Factors (Goal 2) ---
                    Importance (Abs. Coeff)  Original Coeff    Impact
    Month_12                     774.341432     -774.341432  Negative
    Month_5                      672.043663      672.043663  Positive
    DayOfWeek_Mon                666.911762     -666.911762  Negative
    DayOfWeek_Weds               609.249685     -609.249685  Negative
    Month_9                      498.089131      498.089131  Positive
    DayOfWeek_Sun                497.778373     -497.778373  Negative
    DayOfWeek_Sat                404.192599     -404.192599  Negative
    Month_11                     324.845608      324.845608  Positive
    Month_8                      299.324700      299.324700  Positive
    Month_4                      290.974948     -290.974948  Negative
    

excel_file_path = 'Top_Factor_Importance.xlsx'

try:
    importance_df.to_excel(
        excel_file_path, 
        sheet_name='Factors', 
        index=True 
    )
    
    print("--- Success! ---")
    print(f"Top 15 prediction factors successfully exported to: {excel_file_path}")
    print("The file is saved in the current working directory.")

except Exception as e:
    print(f"An error occurred during file export: {e}")


```python

```

                           Importance (Abs. Coeff)  Original Coeff    Impact
    Month_12                            774.341432     -774.341432  Negative
    Month_5                             672.043663      672.043663  Positive
    DayOfWeek_Mon                       666.911762     -666.911762  Negative
    DayOfWeek_Weds                      609.249685     -609.249685  Negative
    Month_9                             498.089131      498.089131  Positive
    DayOfWeek_Sun                       497.778373     -497.778373  Negative
    DayOfWeek_Sat                       404.192599     -404.192599  Negative
    Month_11                            324.845608      324.845608  Positive
    Month_8                             299.324700      299.324700  Positive
    Month_4                             290.974948     -290.974948  Negative
    Month_6                             226.626636     -226.626636  Negative
    CarrierType_Intercity               163.377273     -163.377273  Negative
    AvgTemperature_F                    148.064976      148.064976  Positive
    DayOfWeek_Thurs                     146.055027     -146.055027  Negative
    IsHoliday                           145.152876      145.152876  Positive
    


```python

```
