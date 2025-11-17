**Project Structure**
Patient-Flow-Optimization/
│── README.md
│── data/
│     └── patient_flow_data.csv  <-- your dataset goes here
│── python/
│     ├── clean_data.py
│     ├── feature_engineering.py
│     └── eda.py
│── sql/
│     ├── create_tables.sql
│     └── analysis_queries.sql
│── powerbi/
│     ├── model_description.txt
│     └── dax_measures.txt
│── documentation/
      └── data_dictionary.md

**Scripts**
**Python Scripts**
**1. clean_data.py**
import pandas as pd

# Load dataset
df = pd.read_csv("../data/patient_flow_data.csv")

# Convert timestamps
df['Arrival_Time'] = pd.to_datetime(df['Arrival_Time'])
df['Triage_Start'] = pd.to_datetime(df['Triage_Start'])
df['Doctor_Start'] = pd.to_datetime(df['Doctor_Start'])
df['Discharge_Time'] = pd.to_datetime(df['Discharge_Time'])

# Calculate durations
df['Wait_Time'] = (df['Triage_Start'] - df['Arrival_Time']).dt.total_seconds() / 60
df['Triage_Duration'] = (df['Doctor_Start'] - df['Triage_Start']).dt.total_seconds() / 60
df['Treatment_Duration'] = (df['Discharge_Time'] - df['Doctor_Start']).dt.total_seconds() / 60
df['Total_LOS'] = (df['Discharge_Time'] - df['Arrival_Time']).dt.total_seconds() / 60


**2. feature_engineering.py**

import pandas as pd

df = pd.read_csv("../data/patient_flow_cleaned.csv")

# Create peak hour category
df['Hour'] = df['Arrival_Time'].str[11:13].astype(int)
df['Peak_Hour'] = df['Hour'].apply(lambda x: 'Peak' if 10 <= x <= 12 else ('Medium' if 14 <= x <= 17 else 'Off-Peak'))

# Map departments to groups if needed
department_map = {
    'General':'General',
    'Pediatrics':'Pediatrics',
    'Ortho':'Orthopedic',
    'Gynecology':'Gynecology',
    'Casualty':'Emergency',
    'Cardiology':'Cardio',
    'Neurology':'Neuro',
    'Emergency':'Emergency',
    'ENT':'ENT',
    'Dermatology':'Skin'
}
df['Dept_Group'] = df['Department'].map(department_map)

df.to_csv("../data/patient_flow_features.csv", index=False)
**3. eda.py**
import pandas as pd

df = pd.read_csv("../data/patient_flow_features.csv")

# Summary statistics
print(df[['Wait_Time','Triage_Duration','Treatment_Duration','Total_LOS']].describe())

# Top 5 busiest departments
print(df['Department'].value_counts().head())

# Peak hour analysis
print(df['Peak_Hour'].value_counts())

**SQL Scripts
create_tables.sql**

CREATE TABLE patient_flow (
    PatientID INT PRIMARY KEY,
    Arrival_Time DATETIME,
    Triage_Start DATETIME,
    Doctor_Start DATETIME,
    Discharge_Time DATETIME,
    Department VARCHAR(50),
    Doctor_ID INT,
    Staff_Count INT,
    Wait_Time FLOAT,
    Triage_Duration FLOAT,
    Treatment_Duration FLOAT,
    Total_LOS FLOAT,
    Peak_Hour VARCHAR(10),
    Dept_Group VARCHAR(50)
);

**analysis_queries.sql**
-- Average wait time by department
SELECT Department, AVG(Wait_Time) as Avg_Wait
FROM patient_flow
GROUP BY Department;

-- Peak hour patient count
SELECT Peak_Hour, COUNT(*) as Patient_Count
FROM patient_flow
GROUP BY Peak_Hour;

-- Staff utilization by department
SELECT Department, AVG(Treatment_Duration) / Staff_Count as Utilization_Per_Staff
FROM patient_flow
GROUP BY Department;

**Step 4: Power BI Dashboard Instructions
Page 1 — Overview**

**KPIs:** Avg Wait, Avg Triage Duration, Avg Treatment Duration, Avg Total LOS

**Visuals:** Line chart for patient inflow, column chart for department counts

**Page 2 — Bottlenecks**

**Histogram:** Wait_Time distribution

**Stacked Column**: Triage vs Treatment Duration by Department

**Page 3 — Staff Utilization**

**Scatter/Bubble**: Staff_Count vs Patient_Count

**Slicer:** Peak_Hour

**DAX Measures**
AvgWait = AVERAGE(patient_flow[Wait_Time])
AvgTriage = AVERAGE(patient_flow[Triage_Duration])
AvgTreatment = AVERAGE(patient_flow[Treatment_Duration])
TotalLOS = AVERAGE(patient_flow[Total_LOS])

**Step 5: README.md Template**
# Patient Flow Optimization Dashboard

## Problem Statement
Analyze hospital patient flow, reduce wait times, and improve staff utilization.

## Dataset
- Source: `archive (15).zip`  
- Columns: PatientID, Arrival_Time, Triage_Start, Doctor_Start, Discharge_Time, Department, Doctor_ID, Staff_Count

## Project Workflow
1. Data Cleaning (Python)  
2. Feature Engineering (Peak Hour, Dept Groups)  
3. Exploratory Data Analysis (Python + SQL)  
4. Power BI Dashboard: KPIs, department analysis, staff utilization

## Insights
- Average wait time trends per department  
- Peak hour patient inflow  
- Staff utilization efficiency

## Folder Structure

Patient-Flow-Optimization/
│── README.md
│── data/
│── python/
│── sql/
│── powerbi/
│── documentation/



## Next Steps
- Add predictive models for patient load  
- Simulate staffing optimization  




















# Save cleaned data
df.to_csv("../data/patient_flow_cleaned.csv", index=False)
