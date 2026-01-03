# Implementation Documentation – Traffic Accident & Weather Analytics

This document describes the complete technical implementation of the project, including AWS setup, data ingestion, transformations, analytics, and conclusions. 

---

## 1️. Environment Setup

### 1.1 AWS Region
- Project Region: **us-east-1 (N. Virginia)**

### 1.2 AWS Services Used
- **Amazon S3** → Data lake storage  
- **AWS Lambda** → Collecting weather API data  
- **AWS Glue Crawler** → Schema discovery & Data Catalog  
- **Amazon Athena** → Query execution and analytics  

---

## 2️. S3 – Data Lake Setup

### 2.1 Bucket Structure

The project uses **one main S3 bucket**: traffic-accident-weather-sara


Inside this main data lake bucket, folders are used to separate different pipeline stages.

---

### Initial Structure
At the start of the project, three separate folders were created to represent **Raw → Curated pipeline layers**:

![S3 Buckets](s3_buckets_1.png)

- `raw-accidents-imr3jw/`  
  Stores the original accident dataset uploaded to S3 without modification.

- `raw-weather-imr3jw/`  
  Stores weather data collected via the API in JSON format.

- `curated-imr3jw/`  
  Stores the final cleaned and enriched dataset after joining accidents and weather.
---

### Final Structure

During later development and while delivering the solution, additional folders were added to support Athena processing and updated Lambda output.

The final S3 structure looks like this:

![S3 Buckets](s3_buckets_2.png)

Explanation of final folders:

- `raw-accidents-imr3jw/`  
  Original accident data (unchanged, source of truth).

- `raw-weather-imr3jw/`  
  Weather API response files stored in S3.

- `weather_2019/`  
  A refined weather dataset created later in the project to improve usability and date matching with accident data.

- `curated-imr3jw/`  
  Final analytics-ready dataset containing **accidents joined with weather**.

- `athena-results-imr3jw/`  
  This folder is automatically created by **Amazon Athena** to store query result files.

---
---

## 3️. Accident Data Ingestion

### 3.1 Accident Dataset Source

The accident dataset used in this project was obtained from the official UK government open data portal:

https://www.data.gov.uk/dataset/6efe5505-941f-45bf-b576-4c1e09b579a1/road-traffic-accidents

The specific dataset used is the **2019 accident dataset**, which I uploaded to S3 under the filename: 

![Accidents data](2019.csv)


This dataset contains all officially recorded **road traffic accidents in the UK for the year 2019**, including detailed records for the city of Leeds.

---

### 3.2 Why 2019 Leeds Accidents?

For this project, Leeds was selected as the geographical focus area. This means:
- only accidents that occurred in **Leeds** were used in the analysis
- weather API calls were made specifically for Leeds coordinates
- all weather–accident relationships are therefore meaningful within a single city context
---

### 3.3 S3 Storage

The dataset was uploaded into:
`raw-accidents-imr3jw/` 

---

# 4. AWS Lambda – Weather Data Ingestion

AWS Lambda is responsible for retrieving **historical weather data** that corresponds to the accident records. Since the accident dataset contains only accident information but no weather conditions, Lambda enriches the dataset by fetching the weather data from an external API and storing it in S3 for later use.

---

## 4.1 Lambda Configuration

| Property | Value |
|--------|--------|
| Runtime | Python 3.x |
| Execution | On-demand (manual), extendable to scheduled |
| Region | us-east-1 |
| Memory | Default |
| Timeout | Adjusted to allow API execution |
| IAM Role | LabRole (with S3 read/write permission) |

Lambda is serverless, meaning:
- No servers to manage  
- Automatically scales  
- Only costs money when executed  

This makes it highly cost-efficient.

![Lambda function configuruation](lambda_config.png)

---

## 4.2 External Weather API

Lambda calls the **Open-Meteo Historical Weather API**, chosen because it is:

- Free to use  
- Reliable and well documented  
- Provides historical datasets  
- Contains weather metrics relevant to road safety  

Weather fields retrieved:
- Temperature  
- Wind speed  
- Precipitation  

---

## 4.3 Environment Variables

Lambda uses environment variables for flexibility and reusability:

| Key | Value | Purpose |
|-----|--------|--------|
| `BUCKET_NAME` | `traffic-accident-weather-sara` | Where results are stored |
| `WEATHER_PREFIX` | `raw-weather-imr3jw/` | Folder for weather files |

---

## 4.4 Code of lambda function

```python
import json
import boto3
import urllib3
import os

s3 = boto3.client("s3")
http = urllib3.PoolManager()

BUCKET = os.environ["BUCKET_NAME"]  
KEY_PREFIX = "raw-weather-imr3jw/weather_2019/"



def lambda_handler(event, context):

    latitude = 53.8137      
    longitude = -1.5607

    url = (
        "https://archive-api.open-meteo.com/v1/archive?"
        f"latitude={latitude}&longitude={longitude}"
        "&start_date=2019-01-01"
        "&end_date=2019-12-31"
        "&hourly=temperature_2m,wind_speed_10m,precipitation"
    )

    response = http.request("GET", url)
    data = json.loads(response.data.decode("utf-8"))
    
    # Save to S3
    key = KEY_PREFIX + "weather_2019.json"
    
    s3.put_object(
        Bucket=BUCKET,
        Key=key,
        Body=json.dumps(data)
    )

    return {
        "statusCode": 200,
        "message": "Historical weather for 2019 saved to S3",
        "s3_key": key
    }
```
---

## 4.5 Proof of Successful Weather Ingestion

After running the Lambda function, a historical weather JSON file was successfully created and saved into the S3 raw weather zone:

```
raw-weather-imr3jw/weather_2019/weather_2019.json
```

This confirms that:
- The Lambda function executed correctly  
- The API returned valid data  
- S3 write permission worked as expected  

![Weather data](weather_data.png)

---

# 5️. AWS Glue 

AWS Glue was used to automatically detect schema and make both accident and weather datasets queryable via Athena.

---

## 5.1 Glue Database

A new Glue database was created with the name:

```
traffic_project_db
```
![Traffic project db](traffic_db.png)

---

## 5.2 Glue Crawler Setup

A Glue Crawler was configured to scan the raw data folders in S3 and automatically detect table structure.

### Crawler Configuration Summary
- **Data sources included:**
  - raw-accidents-imr3jw
  - raw-weather-imr3jw
- **Target Database:**
  - traffic_project_db
- **IAM Role:**
  - LabRole (automatically applied in AWS Academy)
- **Crawl Type:**
  - “Crawl all subfolders”

![Crawler created](crawler_1.png)
![Crawler seccfully completed](crawler_2.png)

---

## 5.3 Results of the Crawler

Once executed, the crawler successfully created tables in the Glue Data Catalog.

Tables created included:
- Accident table
- Weather table

These tables then became accessible in Athena for querying and transformation.

![Tables in traffic-project-db](traffic_db_tables.png)

---







