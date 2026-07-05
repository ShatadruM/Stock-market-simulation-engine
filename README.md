# 📈 Real-Time Stock Market Data Engineering Pipeline

An end-to-end data engineering project that simulates real-time stock market data streaming, processing, and querying using **Apache Kafka** and the **AWS Cloud ecosystem**.

---

## 🏗️ Architecture

```
 ┌────────────┐      ┌──────────────────┐      ┌───────────────┐
 │  CSV Data  │ ───► │  Kafka Producer  │ ───► │  Kafka Topic  │
 │ (indexProcessed.csv) │  (Python script) │      │  "demo_test"  │
 └────────────┘      └──────────────────┘      └───────┬───────┘
                                                          │
                                                          ▼
                                                ┌──────────────────┐
                                                │  Kafka Consumer  │
                                                │  (Python script) │
                                                └───────┬──────────┘
                                                          │
                                                          ▼
                                                ┌──────────────────┐
                                                │   Amazon S3      │
                                                │  (Data Lake)     │
                                                └───────┬──────────┘
                                                          │
                                                          ▼
                                                ┌──────────────────┐
                                                │   AWS Glue       │
                                                │  Crawler/Catalog │
                                                └───────┬──────────┘
                                                          │
                                                          ▼
                                                ┌──────────────────┐
                                                │  Amazon Athena   │
                                                │  (SQL Queries)   │
                                                └──────────────────┘
```

1. **Data Simulation** — A Python producer script reads historical stock market data from a CSV file and pushes it row-by-row to a Kafka topic, simulating a live data stream (one record per second).
2. **Message Broker (Apache Kafka)** — An AWS EC2 instance hosts Apache Kafka and Zookeeper to ingest, queue, and serve the streaming data.
3. **Data Lake Storage (Amazon S3)** — A Python consumer reads the Kafka stream and writes each JSON message directly into an S3 bucket as an individual object.
4. **Data Catalog (AWS Glue)** — An AWS Glue Crawler scans the S3 bucket to automatically infer the schema and build/update a data catalog.
5. **Serverless Analytics (Amazon Athena)** — Athena connects to the Glue Catalog, enabling direct, serverless SQL querying of the streaming data sitting in S3 — no ETL or servers required.

---

## 🛠️ Tech Stack

| Category | Tools / Services |
|---|---|
| **Language** | Python 3 |
| **Streaming** | Apache Kafka, Zookeeper |
| **Cloud Compute** | AWS EC2 (Kafka broker host) |
| **Cloud Storage** | Amazon S3 |
| **Cataloging** | AWS Glue Crawler |
| **Querying** | Amazon Athena |
| **Python Libraries** | `kafka-python`, `s3fs`, `pandas` |

---

## 📂 Project Structure

```
.
├── KafkaProducer.ipynb   # Reads CSV data and streams it to a Kafka topic
├── KafkaConsumer.ipynb   # Consumes the Kafka topic and writes records to S3
├── indexProcessed.csv   # Historical stock market dataset (source data)
│   
└── README.md
```

---

## ⚙️ Prerequisites

Before running this project, make sure you have:

- An AWS account with permissions for **EC2**, **S3**, **Glue**, and **Athena**
- An EC2 instance with **Apache Kafka** and **Zookeeper** installed and running
- AWS credentials configured locally (e.g. via `aws configure` or environment variables) so `s3fs` can authenticate
- Python 3.8+
- The following Python packages:
  ```bash
  pip install kafka-python s3fs pandas
  ```

---

## 🔧 Configuration

Both notebooks connect to the Kafka broker using a `bootstrap_servers` argument that currently points to a placeholder address:

```python
bootstrap_servers=[':9092']
```

Before running either notebook, replace this with your **EC2 instance's public or private IP**, for example:

```python
bootstrap_servers=['<your-ec2-ip>:9092']
```

Make sure port `9092` is open in your EC2 instance's security group (and `2181` if Zookeeper needs external access).

---

## 🚀 How It Works

### 1. `KafkaProducer.ipynb`
- Installs `kafka-python` and imports the required libraries.
- Creates a `KafkaProducer` instance, serializing message values as UTF-8 encoded JSON.
- Sends a one-off test message to confirm connectivity.
- Loads `data/indexProcessed.csv` into a pandas DataFrame.
- Enters an infinite loop that:
  1. Randomly samples **one row** from the DataFrame.
  2. Sends it as a JSON message to the `demo_test` Kafka topic.
  3. Sleeps for **1 second** to simulate a real-time feed.
- Calls `producer.flush()` to ensure all buffered messages are sent before the producer shuts down.

### 2. `KafkaConsumer.ipynb`
- Creates a `KafkaConsumer` subscribed to the `demo_test` topic, deserializing incoming bytes from UTF-8 JSON.
- Initializes an `S3FileSystem` connection via `s3fs`.
- Continuously iterates over incoming Kafka messages, writing **each message** to S3 as a separate JSON file:
  ```
  s3://<your-bucket-name>/stock_market_<count>.json
  ```
- The `count` in the filename increments with each message, ensuring unique object keys.

---

## ▶️ Running the Pipeline

1. **Start Kafka & Zookeeper** on your EC2 instance.
2. **Create the Kafka topic** (if it doesn't already exist):
   ```bash
   kafka-topics.sh --create --topic demo_test --bootstrap-server <your-ec2-ip>:9092 --partitions 1 --replication-factor 1
   ```
3. **Run `KafkaConsumer.ipynb`** first so it's ready to receive messages.
4. **Run `KafkaProducer.ipynb`** to begin streaming stock data into the topic.
5. **Verify data landing in S3** — check your bucket for incoming `stock_market_*.json` files.
6. **Run an AWS Glue Crawler** against the S3 bucket to infer the schema and populate the Glue Data Catalog.
7. **Query the data in Amazon Athena** using standard SQL against the cataloged table.

---


## 🚧 Future Improvements

- Add schema validation/error handling in the consumer before writing to S3
- Batch small S3 writes into larger files to reduce object count and improve Athena query performance
- Add partitioning (e.g. by date) in S3 for more efficient Athena queries
- Containerize the producer/consumer scripts with Docker for easier deployment
- Add monitoring/alerting for the Kafka consumer lag

---


