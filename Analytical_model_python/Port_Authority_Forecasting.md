import pyodbc
import pandas as pd


```python
CONN_STRING = (
    r'Driver={ODBC Driver 17 for SQL Server};'
    r'Server=LHAKSON\SQLSERVER2022;' 
    r'Database=Port_Authority_Bus_Database;'
    r'Trusted_Connection=yes;'   
try:
    # 2. Establish the connection
    cnxn = pyodbc.connect(CONN_STRING)
    print("Connection established successfully!")

except pyodbc.Error as ex:
    sqlstate = ex.args[0]
    print(f"Error connecting: {sqlstate}")
```

    Connection established successfully!
    


```python
import pandas as pd
from prophet import Prophet
```


```python
SQL_QUERY = """
SELECT
    DD.FullDate,
    FP.DeparturePassengerCount,
    DC.CarrierName,
    FP.DepartureBusCount,
    DC.CarrierType
    
FROM
    FactPassenger FP
JOIN
    DimDate DD ON FP.DateID = DD.DateID
JOIN
    DimCarrier DC ON FP.CarrierID = DC.CarrierID
WHERE
    DD.FullDate BETWEEN '2020-11-30' AND '2025-08-01' -- Filter for your historical period
ORDER BY
    DD.FullDate;
"""
try:
    # Establish connection
    cnxn = pyodbc.connect(CONN_STRING)

    # Load the queried data directly into a Pandas DataFrame
    df_history = pd.read_sql(SQL_QUERY, cnxn)
    print("Historical data loaded successfully into df_history!")
    
except pyodbc.Error as ex:
    print(f"Error connecting or querying database: {ex}")
    df_history = pd.DataFrame() # Create an empty DataFrame in case of failure

finally:
    if 'cnxn' in locals() and cnxn:
        cnxn.close()
        print("SQL Server connection closed.")

if not df_history.empty:
    print("\nData Preview:")
    print(df_history.head())
```

    Historical data loaded successfully into df_history!
    SQL Server connection closed.
    
    Data Preview:
        FullDate  DeparturePassengerCount CarrierName  DepartureBusCount  \
    0 2020-11-30                      615     Academy                 37   
    1 2020-11-30                      750   Greyhound                 36   
    2 2020-11-30                      486       Martz                 17   
    3 2020-11-30                    26512  NJ Transit               2108   
    4 2020-11-30                      274    Lakeland                 13   
    
      CarrierType  
    0    Commuter  
    1   Intercity  
    2    Commuter  
    3    Commuter  
    4    Commuter  
    

    C:\Users\Admin\AppData\Local\Temp\ipykernel_33752\736913113.py:25: UserWarning: pandas only supports SQLAlchemy connectable (engine/connection) or database string URI or sqlite3 DBAPI2 connection. Other DBAPI2 objects are not tested. Please consider using SQLAlchemy.
      df_history = pd.read_sql(SQL_QUERY, cnxn)
    


```python
from prophet import Prophet 
import pandas as pd


CARRIER_TO_EXCLUDE = 'C & J Bus Lines' 
START_DATE_FORECAST = '2025-08-02'  
END_DATE_FORECAST = '2030-12-31'    

carrier_forecasts = {}

if not df_history.empty:
    df_history = df_history[df_history['CarrierName'] != CARRIER_TO_EXCLUDE].copy()
    unique_carriers = df_history['CarrierName'].unique()
    
    print(f"\nForecasting {len(unique_carriers)} carriers weekly from {START_DATE_FORECAST} to {END_DATE_FORECAST}.")

    
    df_future_fixed = pd.DataFrame({
        'ds': pd.date_range(start=START_DATE_FORECAST, 
                            end=END_DATE_FORECAST, 
                            freq='W-SAT')
    })
    
    print(f"Generated first 5 forecast dates: {df_future_fixed['ds'].head().tolist()}")


   
    for carrier in unique_carriers:
        print(f"\n--- Training WEEKLY Model for: {carrier} ---")
        
       
        df_carrier = df_history[df_history['CarrierName'] == carrier].copy()
        df_prophet = df_carrier.rename(columns={'FullDate': 'ds', 'DeparturePassengerCount': 'y'})
        df_prophet['ds'] = pd.to_datetime(df_prophet['ds'])
        df_prophet['y'] = pd.to_numeric(df_prophet['y'])
        
       
        model = Prophet(seasonality_mode='multiplicative', daily_seasonality=False, weekly_seasonality=False)
        model.fit(df_prophet)
        
        
        forecast = model.predict(df_future_fixed)
        
       
        forecast['CarrierName'] = carrier 
        carrier_forecasts[carrier] = forecast[['ds', 'yhat', 'CarrierName']]

    #
    final_output_long_simple = pd.concat(carrier_forecasts.values(), ignore_index=True)
    final_output_long_simple.columns = ['Date', 'ProjectedPassengerCount', 'CarrierName']
    final_output_long_simple = final_output_long_simple.sort_values(by=['Date', 'CarrierName']).reset_index(drop=True)

    print("\n--- Final WEEKLY Forecast Combined (2025-2030) ---")
    print(final_output_long_simple.head())
    
```

    21:03:58 - cmdstanpy - INFO - Chain [1] start processing
    21:03:58 - cmdstanpy - INFO - Chain [1] done processing
    21:03:58 - cmdstanpy - INFO - Chain [1] start processing
    21:03:58 - cmdstanpy - INFO - Chain [1] done processing
    

    
    Forecasting 11 carriers weekly from 2025-08-02 to 2030-12-31.
    Generated first 5 forecast dates: [Timestamp('2025-08-02 00:00:00'), Timestamp('2025-08-09 00:00:00'), Timestamp('2025-08-16 00:00:00'), Timestamp('2025-08-23 00:00:00'), Timestamp('2025-08-30 00:00:00')]
    
    --- Training WEEKLY Model for: Academy ---
    
    --- Training WEEKLY Model for: Greyhound ---
    

    21:03:58 - cmdstanpy - INFO - Chain [1] start processing
    21:03:58 - cmdstanpy - INFO - Chain [1] done processing
    21:03:58 - cmdstanpy - INFO - Chain [1] start processing
    21:03:58 - cmdstanpy - INFO - Chain [1] done processing
    

    
    --- Training WEEKLY Model for: Martz ---
    
    --- Training WEEKLY Model for: NJ Transit ---
    

    21:03:58 - cmdstanpy - INFO - Chain [1] start processing
    21:03:58 - cmdstanpy - INFO - Chain [1] done processing
    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    

    
    --- Training WEEKLY Model for: Lakeland ---
    
    --- Training WEEKLY Model for: Trailways ---
    

    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    

    
    --- Training WEEKLY Model for: Coach USA ---
    
    --- Training WEEKLY Model for: TransBridge ---
    

    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    

    
    --- Training WEEKLY Model for: Peter Pan/Bonanza ---
    
    --- Training WEEKLY Model for: HCEE - Community ---
    

    21:03:59 - cmdstanpy - INFO - Chain [1] start processing
    21:03:59 - cmdstanpy - INFO - Chain [1] done processing
    

    
    --- Training WEEKLY Model for: Decamp ---
    
    --- Final WEEKLY Forecast Combined (2025-2030) ---
            Date  ProjectedPassengerCount       CarrierName
    0 2025-08-02              1542.523067           Academy
    1 2025-08-02              5803.990150         Coach USA
    2 2025-08-02              1083.819959            Decamp
    3 2025-08-02              1690.365683         Greyhound
    4 2025-08-02              1729.622139  HCEE - Community
    

filename_final_weekly = 'Weekly_Bus_Forecast_2025_2030.xlsx'


final_output_long_simple.to_excel(filename_final_weekly, index=False)

print(f"✅ Success! The final weekly forecast has been saved to: {filename_final_weekly}")




