# Implementation Documentation – Traffic Accident & Weather Analytics

This document describes the complete technical implementation of the project, including AWS setup, data ingestion, transformations, analytics, and conclusions. 

---

## 1️. Environment Setup

### 1.1 AWS Region
- Region used: us-east-1 (N. Virginia)

### 1.2 Tools Used
- AWS S3
- AWS Lambda
- AWS Glue Crawler
- AWS Athena

---

## 2️. S3 – Data Lake Setup

### 2.1 Bucket Structure
After creating a bucket for the whole project (traffic-accidents-weather-sara). Inside this folder, orginally three separate buckets were used instead of folders:



