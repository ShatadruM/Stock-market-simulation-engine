# 📈 Real-Time Stock Market Data Engineering Pipeline

An end-to-end data engineering project that simulates real-time stock market data streaming, processing, and querying using Apache Kafka and the AWS Cloud ecosystem.

## 🏗️ Architecture
1. **Data Simulation:** A Python producer script reads historical stock market data from a CSV and pushes it row-by-row to simulate a live data stream.
2. **Message Broker (Apache Kafka):** An AWS EC2 instance hosts Apache Kafka and Zookeeper to ingest, queue, and serve the streaming data.
3. **Data Lake Storage (Amazon S3):** A Python consumer reads the Kafka stream and flushes the JSON payload directly into an S3 bucket.
4. **Data Catalog (AWS Glue):** An AWS Glue Crawler scans the S3 bucket to automatically infer the schema and construct a data catalog.
5. **Serverless Analytics (Amazon Athena):** Athena connects to the Glue Catalog, enabling direct, serverless SQL querying of the live streaming data in S3.

## 🛠️ Tech Stack
* **Languages:** Python
* **Cloud Services:** AWS EC2, Amazon S3, AWS Glue, Amazon Athena
* **Streaming:** Apache Kafka, Zookeeper
* **Libraries:** `kafka-python`, `s3fs`, `pandas`

