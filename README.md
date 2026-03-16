# 🚖 Uber Real-Time Data Engineering Pipeline (Azure + Databricks)

## 📌 Project Overview

This project demonstrates how to build a **real-time data engineering pipeline** similar to what companies like Uber use to process ride events. The pipeline captures ride booking events from a **web application**, streams them through **Azure Event Hub**, processes them using **Azure Databricks with Apache Spark Structured Streaming**, and stores the data in **Azure Data Lake Storage (ADLS)** using **Medallion Architecture (Bronze → Silver → Gold layers)**.

The goal of this project is to simulate how **real-time ride data flows through a modern data platform**, where events are ingested, processed, cleaned, and transformed into analytics-ready datasets.

---

# 🏗️ Project Architecture

### END-to-End Data Flow

Web Application
↓
Azure Event Hub (Streaming Ingestion)
↓
Azure Databricks (Streaming Processing)
↓
Azure Data Lake Storage (ADLS)
↓
Bronze Layer → Silver Layer → Gold Layer
↓
Analytics / Reporting / Business Insights

---

# 📊 Technologies Used

| Technology                     | Purpose                     |
| ------------------------------ | --------------------------- |
| Python                         | Simulating ride events      |
| Azure Event Hub                | Real-time event streaming   |
| Azure Databricks               | Data processing             |
| Apache Spark                   | Distributed data processing |
| Azure Data Lake Storage (ADLS) | Data storage                |
| Delta Lake                     | Reliable data format        |
| Azure Data Factory             | Pipeline orchestration      |

---

# 1️⃣ Web Application (Event Producer)

## What is the Web Application?

The web application simulates an **Uber ride booking system**. Every time a user books a ride, the application generates a **ride event** and **Azure Event Hub**

These events represent real-world ride activity.

### Example Ride Data Generated

* Ride ID
* Passenger ID
* Driver ID
* Pickup location
* Drop location
* Ride fare
* Timestamp

### Example Event (JSON)

```json
{
  "ride_id": "c970ff1c-d0d8-4cc2-9896-b9f06c2e14b6",
  "passenger_id": "P102",
  "driver_id": "D501",
  "pickup_location": "Hyderabad",
  "drop_location": "Airport",
  "fare": 450,
  "ride_status": "completed",
  "timestamp": "2026-03-14T10:20:00"
}

This simulates real-time Uber ride activity. 
```

### What Happens Here?

1. A ride request is generated.
2. The application converts ride data into JSON format.
3. The event is sent to **Azure Event Hub**.

This application acts as the **event producer**.

---

# 2️⃣ Azure Event Hub

## What is Azure Event Hub?

Azure Event Hub is a **real-time data streaming platform** used to collect and process millions of events per second.

It acts as a **data ingestion service** between the application and the processing engine.

### Key Concepts

#### Event

An **event** is a single piece of information produced by an application.

Example Uber events:

* Ride requested
* Ride accepted
* Ride started
* Ride completed
* Payment processed

Each event contains ride-related data.

---

### Namespace

A **namespace** is a container that holds one or more event hubs.

Example:

```
Namespace : UberEventsP
```

---

### Event Hub Name

An **Event Hub** is a stream where events are sent and stored temporarily.

Example:

```
Event Hub Name : uber-topic
```

---

### What Happens in Event Hub?

1. Web app sends ride events.
2. Event Hub receives the events.
3. Events are temporarily stored in partitions.
4. Databricks consumes the events in real time.

Event Hub works similar to **Kafka streaming systems**.

---

# 3️⃣ Streaming

## What is Streaming?

Streaming means processing data **continuously as it is generated**, instead of processing large batches of data at scheduled intervals.

### Batch vs Streaming

| Batch Processing               | Streaming Processing     |
| ------------------------------ | ------------------------ |
| Data processed every few hours | Data processed instantly |
| Large datasets                 | Continuous flow          |
| Delayed insights               | Real-time insights       |

Since Uber rides happen every second, **streaming processing is required**.

---

# 4️⃣ Azure Databricks

## What is Azure Databricks?

Azure Databricks is a **big data analytics platform built on Apache Spark** that allows engineers to process massive amounts of data efficiently.

It is used to:

* Process streaming data
* Perform transformations
* Clean and structure data
* Store processed data into Data Lake

---

### What Happens in Databricks?

1. Connect to Azure Event Hub
2. Read streaming data
3. Apply transformations using Spark
4. Store results into ADLS

Databricks processes the data using **Spark Structured Streaming**.

---

# 5️⃣ Azure Data Lake Storage (ADLS)

## What is ADLS?

Azure Data Lake Storage is a **cloud-based storage system designed for big data analytics**.

It can store:

* Raw data
* Structured data
* Processed datasets
* Historical records

### Why Data Lake?

Data lakes allow organizations to:

* Store unlimited data
* Keep raw and processed datasets
* Run large analytics workloads

---

# 6️⃣ Medallion Architecture

This project follows **Medallion Architecture**, which organizes data into three layers:

```
Bronze Layer → Silver Layer → Gold Layer
```

Each layer improves **data quality and usability**.

---

# 🥉 Bronze Layer (Raw Data Layer)

## What is Bronze Layer?

The Bronze layer stores **raw data exactly as it arrives from the streaming source**.

No transformations are applied.

### What Happens in Bronze Layer?

1. Databricks reads streaming events from Event Hub
2. Raw data is written directly into ADLS
3. Data is stored in **Delta format**

### Storage Path Example

```
/mnt/uber-data/bronze/ride_events/
```

### Purpose of Bronze Layer

* Preserve raw data
* Enable reprocessing if errors occur
* Maintain original event history

---

# 🥈 Silver Layer (Cleaned Data Layer)

## What is Silver Layer?

The Silver layer contains **cleaned and structured data**.

This layer removes bad or invalid data.

### Operations Performed

Typical transformations include:

* Removing null values
* Removing duplicate records
* Fixing incorrect data types
* Filtering invalid records
* Standardizing column formats

Example filtering rule:

```
fare > 0
```

### Data Flow

```
Bronze Data → Cleaning & Transformation → Silver Tables
```

### Storage Path

```
/mnt/uber-data/silver/ride_cleaned/
```

---

# 🥇 Gold Layer (Business Layer)

## What is Gold Layer?

The Gold layer contains **aggregated and business-ready data**.

This layer is used for **analytics, dashboards, and reporting**.

### Operations Performed

Examples of aggregations:

* Total rides per city
* Average fare
* Revenue per day
* Peak ride hours
* Driver performance

Example table:

| city      | total_rides | total_revenue |
| --------- | ----------- | ------------- |
| Hyderabad | 1200        | 450000        |

### Data Flow

```
Silver Data → Aggregation → Gold Tables
```

### Storage Path

```
/mnt/uber-data/gold/analytics/
```

**Gold Data Flow
**

Silver Data
⬇
Aggregation
⬇
Business Metrics
⬇
Gold Tables

# 7️⃣ Azure Data Factory (ADF)

## What is Azure Data Factory?

Azure Data Factory is a **data integration and orchestration service**.

It is used to **automate and schedule data workflows**.

### What We Use ADF For

* Schedule Databricks pipelines
* Orchestrate data workflows
* Move data between services
* Automate ETL pipelines

### Example Workflow

```
ADF Pipeline
   ↓
Trigger Databricks Notebook
   ↓
Process Bronze → Silver → Gold
```

---

# 8️⃣ Event Hub → Databricks Integration

Databricks connects to Event Hub using a **connection string**.

Spark Structured Streaming reads events continuously.

Example concept:

```
Event Hub → Spark Structured Streaming → Databricks Processing
```

---

# 9️⃣ ADLS → Databricks Connection

Databricks connects to ADLS using **mount points** or **storage credentials**.

Example storage path:

```
abfss://uber-data@storageaccount.dfs.core.windows.net/
```

This allows Databricks to read and write data into the data lake.

---

# 🔟 End-to-End Pipeline Flow

Step 1
Web application generates Uber ride events.

Step 2
Ride events are sent to Azure Event Hub.

Step 3
Event Hub receives and stores streaming events.

Step 4
Azure Databricks reads events using Spark streaming.

Step 5
Raw events are stored in **Bronze layer**.

Step 6
Data is cleaned and transformed into **Silver layer**.

Step 7
Business-level aggregations are created in **Gold layer**.

Step 8
Gold datasets are used for **analytics and reporting**.



# 📈 Key Features

✔ Real-time data ingestion
✔ Streaming data processing
✔ Scalable cloud architecture
✔ Medallion data architecture
✔ Reliable Delta Lake storage
✔ End-to-end Azure data pipelines 

# 🎯 Learning Outcomes

Through this project you will learn:

* Real-time data streaming architecture
* Azure Event Hub integration
* Spark Structured Streaming
* Azure Databricks processing
* Medallion Architecture implementation
* Data Lake storage strategies
* Building production-style data pipelines

---

# 🚀 Future Improvements

* Add Power BI dashboards
* Implement monitoring and logging
* Add data quality validation
* Implement CI/CD for pipelines
* Add real-time analytics

---


## Project Architecture 
![Project Architecture](architecture.png)



# 👩‍💻 Author

**Sravanthi Modiboyena**

Data Engineering Project built using Azure ecosystem demonstrating real-time streaming pipelines and modern data lake architecture.


