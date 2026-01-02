# Traffic Accident & Weather Analytics Pipeline

This project investigates **how weather conditions influence road traffic accidents**, focusing on identifying patterns that explain why accidents happen more frequently under certain environmental circumstances.

The solution implements a modern cloud-based analytics pipeline on AWS, integrating accident data with live weather API data, transforming it, and enabling analytical queries to answer meaningful business questions.

---

## Project Documentation

| File | Purpose |
|------|--------|
| [BUSINESS_DOCUMENTATION.md](BUSINESS_DOCUMENTATION.md) | Business case, stakeholders, KPIs, architecture, value |
| [IMPLEMENTATION.md](IMPLEMENTATION.md) | Technical execution, AWS services, SQL queries, results interpretation |


---

## Business Goal

The goal of this project is to support **public safety institutions, emergency services, and urban planners** in understanding how weather conditions impact accident frequency and severity.

This knowledge enables **data-driven decision-making**, such as:
- improving safety strategies,
- supporting first responders in resource planning,
- and ensuring appropriate staffing levels depending on expected weather conditions.

Ultimately, this contributes to **better operational efficiency and potentially saving lives**.

---

## Technologies Used

- **AWS S3** – Data Lake storage
- **AWS Lambda** – Weather API ingestion
- **AWS Glue Crawler** – Schema discovery and cataloging
- **AWS Athena** – SQL analytics and querying
- **Open-Meteo API** – External weather data source

---

## Contributors

- Student: **Sára Onder**, Corvinus University of Budapest

---
