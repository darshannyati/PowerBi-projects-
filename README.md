# 🌾 Power BI Crop Analytics with AWS S3 & Snowflake

## 📌 Project Overview
This project focuses on analyzing agricultural data related to crops using **Power BI**, **AWS S3**, and **Snowflake**. The dataset, stored in **AWS S3**, contains information on **temperature, rainfall, humidity, and crop yield** across different **seasons, locations, and years**. The data is processed and transformed in **Snowflake SQL** before being visualized in **Power BI** to derive meaningful insights.

## 🛠️ Tech Stack
- **AWS S3** – Stores the raw dataset.
- **AWS IAM** – Manages roles and policies for secure access.
- **Snowflake** – Loads and transforms data using **Snowflake SQL**.
- **Power BI** – Connects directly to Snowflake for visualization and analysis.

## 📊 Key Features
- **Data Integration:** AWS S3 connected to Snowflake via **integration object**.
- **Data Transformation:** Cleaned and processed dataset in **Snowflake SQL**.
- **Interactive Dashboards:** Power BI reports analyzing:
  - Crop yield trends across different seasons and years.
  - Impact of **temperature, rainfall, and humidity** on crop production.
  - Location-wise agricultural insights.
 
  - ## 🚀 Steps to Load Data into Snowflake

### **1️⃣ Create an AWS S3 Integration in Snowflake**
```sql
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE STORAGE INTEGRATION PBI_INTEGRATION
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::************:role/powerbi.role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://powerbi.project000/')
  COMMENT = 'Power BI S3 Integration';

SHOW INTEGRATIONS;
DESC INTEGRATION PBI_INTEGRATION;
```

### **2️⃣ Create Database and Schema**
```sql
CREATE DATABASE PowerBI;
CREATE SCHEMA PBI_Data;
```

### **3️⃣ Create Table for Storing Crop Data**
```sql
CREATE TABLE PBI_Dataset (
    Year INT,	
    Location STRING,	
    Area INT,
    Rainfall FLOAT, 
    Temperature FLOAT, 
    Soil_type STRING,
    Irrigation STRING, 
    Yields INT,
    Humidity FLOAT,
    Crops STRING,
    Price INT,
    Season STRING
);
```

### **4️⃣ Load Data from AWS S3 to Snowflake**
```sql
CREATE STAGE PowerBI.PBI_Data.pbi_stage
URL = 's3://powerbi.project000'
STORAGE_INTEGRATION = PBI_INTEGRATION;

COPY INTO PBI_Dataset 
FROM @pbi_stage
FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER = ',' SKIP_HEADER = 1)
ON_ERROR = 'CONTINUE';
```

### **5️⃣ Validate Data in Snowflake**
```sql
LIST @pbi_stage;
SELECT * FROM PBI_Dataset;
SELECT Year, COUNT(*) AS Count FROM PBI_Dataset GROUP BY Year ORDER BY Year;
```

---

## 🔧 **Data Transformations in Snowflake**  

### **6️⃣ Create a Copy Table for Analysis**
```sql
CREATE TABLE Agriculture AS 
SELECT * FROM PBI_Dataset;

SELECT * FROM Agriculture;
```

### **7️⃣ Data Adjustments (Scaling Rainfall & Area)**
```sql
UPDATE Agriculture
SET Rainfall = 1.1 * Rainfall;

UPDATE Agriculture
SET Area = 0.9 * Area;
```

### **8️⃣ Categorizing Year Groups**
```sql
ALTER TABLE Agriculture ADD Year_Group STRING;

UPDATE Agriculture
SET Year_Group = 'Y1'
WHERE Year BETWEEN 2004 AND 2009;

UPDATE Agriculture
SET Year_Group = 'Y2'
WHERE Year BETWEEN 2010 AND 2015;

UPDATE Agriculture
SET Year_Group = 'Y3'
WHERE Year BETWEEN 2016 AND 2019;
```

### **9️⃣ Categorizing Rainfall Levels**
```sql
ALTER TABLE Agriculture ADD Rainfall_Groups STRING;

UPDATE Agriculture
SET Rainfall_Groups = 'Low'
WHERE Rainfall BETWEEN 255 AND 1199;

UPDATE Agriculture
SET Rainfall_Groups = 'Medium'
WHERE Rainfall BETWEEN 1200 AND 2799;

UPDATE Agriculture
SET Rainfall_Groups = 'High'
WHERE Rainfall >= 2800;

SELECT * FROM Agriculture;
```
