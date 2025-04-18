# Summary

- [Environment](#environment)
    - [The Environment contains](#the-environment-contains)
    - [The Environment Structure](#the-environment-structure)
    - [Short overview of all technologies](#short-overview-of-all-technologies)
- [Code](#code)
    - [docker-compose.yml](#docker-composeyml)
    - [.env](#env)
    - [prometheus.yml](#prometheusyml)


# Environment

### The Environment contains:

-  Apache NiFi 1.20.0
-  Apache NiFi 2.3.0
-  Apache Nifi Registry
-  Prometheus 
-  S3 \[ minIO \]
-  Redis 
-  Apache Kafka
-  Apache Zookeeper
-  PostgreSQL (server & client)

## The Environment Structure:

![NiFiaaC-Srodowisko.drawio.png](./NiFiaaC.png)

## Short overview of all technologies:

### NiFi

Apache NiFi is a powerful data routing and transformation system. It provides a graphical interface to design data flows and manage data processes in real-time. NiFi simplifies data ingestion, processing, and distribution across various systems and devices.
> [!NOTE]  
> In this project **NiFi** will be exposed on `http://[IP_ADDRESS]:8080` for V1 and on `https://nifi.example.com:8443` for V2 with SSL and certs configured on that domain. To access it via your computer please add DNS record to your local `hosts` file

### NiFi Registry

NiFi Registry is a version control and management system for NiFi flows. It allows users to save, track changes, and manage versions of data flows created in NiFi. NiFi Registry facilitates collaboration, reuse, and deployment of data flow configurations across environments.
> [!NOTE]  
> In this project **NiFi Registry** will be exposed on `http://[IP_ADDRESS]:18080`.

### Prometheus

Prometheus is an open-source monitoring and alerting toolkit. It collects metrics from monitored targets by scraping HTTP endpoints. Prometheus stores data in a time-series database and allows querying and visualization of metrics using a built-in expression language and various visualization tools.
> [!NOTE]  
> In this project **Prometheus** will be exposed on `http://[IP_ADDRESS]:9090`.

### S3

Amazon Simple Storage Service (S3) is a scalable object storage service offered by Amazon Web Services (AWS). It provides secure, durable, and highly available storage for a wide variety of data types. S3 is commonly used for data backup, archival, and serving static web content.

**MinIO** - is an open-source, self-hosted object storage server compatible with Amazon S3. It allows users to store and manage large amounts of unstructured data in a distributed environment. minIO replicates the S3 API, enabling seamless integration with applications designed to work with S3 storage. As a lightweight and scalable solution, minIO is commonly used for various use cases, including data backup, archival, and serving static web content. It offers features such as data encryption, access control, and high availability, making it suitable for both small-scale deployments and enterprise-grade storage solutions

> [!NOTE]  
> In this project **S3** will be exposed on `http://[IP_ADDRESS]:9001`.

### Redis

Redis is an open-source, in-memory data structure store used as a database, cache, and message broker. It supports various data structures such as strings, hashes, lists, sets, and more. Redis is known for its high performance, versatility, and rich set of features.

> [!NOTE]  
> In this project **Redis** can be access via CMD via command `redis-cli`.

### Kafka

Apache Kafka is a distributed streaming platform designed to handle real-time data feeds. It is used for building real-time data pipelines and streaming applications. Kafka provides fault tolerance, scalability, and high throughput for publishing, subscribing, and storing streams of records in a fault-tolerant manner.

> [!NOTE]  
> In this project **Kafka** can be access via CMD via commands available in  `/opt/kafka_2.13-2.8.1/bin/` folder or external application like **Kafka Offset / Kafka Tool**.

### Zookeeper

Apache ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. It is used by distributed systems to manage and coordinate processes in a distributed environment. ZooKeeper ensures high availability and reliability for distributed applications.

### PostgreSQL

PostgreSQL is an open-source relational database management system known for its robustness, scalability, and advanced features. Developed in the 1980s, it supports ACID compliance, various data types (like JSON and XML), and extensibility with custom data types and functions. Its Multi-Version Concurrency Control (MVCC) allows for high performance with concurrent transactions. PostgreSQL is widely used in web applications, data warehousing, and geospatial analysis, making it a popular choice for many industries.

### pgAdmin

pgAdmin is an open-source administration tool for PostgreSQL databases, offering a user-friendly graphical interface. It allows users to manage databases, execute SQL queries, and visualize data. Key features include a powerful query editor, backup and restore options, and cross-platform support. pgAdmin simplifies database management for both beginners and experienced users.
> [!NOTE]  
> In this project **S3** will be exposed on `http://[IP_ADDRESS]:28080`.


### Portainer

Container with web user interface allowing us to manage and control other container in the docker. Web UI can be accessed on 9443 port. To be able to use this container go to **/CODE/Portainer/** and run ```sudo docker-compose up -d``` in this folder

> [!NOTE]  
> In this project **S3** will be exposed on `https://[IP_ADDRESS]:9443`.

### Firefox as a Container

Container used as a jumphost for systems with strict restrictions for DNS changes or networking configuration. For details about this container go to DockerHub website [here](https://hub.docker.com/r/linuxserver/firefox)

# Code:

## docker-compose.yml:

```yml
###--NiFi---###################################################################################################
version: '3'
services:
  nifi:
    image: apache/nifi:1.20.0
    container_name: nifi
    restart: unless-stopped
    network_mode: bridge
    volumes:
      - ./Docker_Mount/V_NiFi_VOLUME:/opt/nifi/nifi-current/VOLUME
    ports:
      # HTTP
      - 8080:8080/tcp
      # HTTPS
      #- 8443:8443/tcp
      # Remote Input Socket
      - 10000:10000/tcp
      # JVM Debugger
      - 8000:8000/tcp
      # Prometheus metrics
      - 9093:9093
      # Cluster Node Protocol
      #- 11443:11443/tcp
    environment:
      ########## Web ##########
      # nifi.web.http.host
      NIFI_WEB_HTTP_HOST: '0.0.0.0'
      # nifi.web.http.port
      #   HTTP Port
      NIFI_WEB_HTTP_PORT: 8080

  nifi2:
    image: apache/nifi:2.3.0
    container_name: nifi2
    restart: unless-stopped
    network_mode: bridge
    volumes:
      - ./Docker_Mount/V_NiFi_2_config:/opt/nifi/nifi-current/conf
      - ./Docker_Mount/V_NiFi_VOLUME:/opt/nifi/nifi-current/VOLUME
    ports:
      # HTTP
      # - 8081:8081/tcp
      # HTTPS
      - 8443:8443/tcp
      # Remote Input Socket
      # - 10000:10000/tcp
      # JVM Debugger
      #- 8000:8000/tcp
      # Prometheus metrics
      # - 29093:9093
      # Cluster Node Protocol
      #- 11443:11443/tcp
    environment:
      ########## Web ##########
      # nifi.web.http.host
      # NIFI_WEB_HTTP_HOST: '0.0.0.0'
      NIFI_WEB_HTTPS_HOST: ''
      # nifi.web.http.port
      #   HTTP Port
      # NIFI_WEB_HTTP_PORT: 8081
      NIFI_WEB_HTTPS_PORT: 8443
###--NiFi-Registry---###################################################################################################
  registry:
    network_mode: bridge
    image: apache/nifi-registry:1.20.0
    container_name: nifi-registry
    restart: unless-stopped
    ports:
     - "18080:18080"
    environment:
     - LOG_LEVEL=INFO
     - NIFI_REGISTRY_DB_DIR=/opt/nifi-registry/database
     - NIFI_REGISTRY_FLOW_PROVIDER=file
     - NIFI_REGISTRY_FLOW_STORAGE_DIR=/opt/nifi-registry/flow_storage
###--Redis---###################################################################################################  
  redis:
    network_mode: bridge
    container_name: redis
    image: redis:6.2-alpine
    restart: always
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning --requirepass [set_up_the_password_here]
###--Kafka---###################################################################################################  
  zookeeper:
    image: wurstmeister/zookeeper
    container_name: zookeeper
    restart: always
    ports:
      - "2181:2181"
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    restart: always
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: [IP_ADDRESS]
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
###--Prometheus---###################################################################################################  
  prometheus:
    image: prom/prometheus:v2.36.2
    container_name: prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - 9090:9090
    restart: always
    volumes:
      - ./Docker_Mount/V_prometheus.yml:/etc/prometheus/prometheus.yml
###--minIO-S3---################################################################################################### 
  minio:
    image: minio/minio
    network_mode: bridge
    container_name: S3
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./Docker_Mount/V_MINIO_storage:/data
    command: server --console-address ":9001" /data
    env_file: ./Docker_Mount/.env
    restart: always
###---PostgreSQL---###################################################################################################
  postgres:
    network_mode: bridge
    container_name: PostgreSQL
    image: 'postgres:latest'
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: [user]
      POSTGRES_PASSWORD: [password]
      POSTGRES_DB: nifi_db
    volumes:
    - ./Docker_Mount/V_PostgreSQL-data/:/var/lib/postgresql/data/
  pgadmin:
    image: dpage/pgadmin4
    container_name: pg_admin
    restart: always
    ports:
      - 28080:80
    network_mode: bridge
    environment:
        PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-[EMAIL_address]}
        PGADMIN_DEFAULT_PASSWORD: [password]
###---Firefox---###################################################################################################
  firefox:
    image: lscr.io/linuxserver/firefox:latest
    container_name: firefox
    network_mode: bridge
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - FIREFOX_CLI=https://www.linuxserver.io/ #optional
    volumes:
      - /path/to/config:/config
    ports:
      - 3000:3000
      - 3001:3001
    shm_size: "1gb"
    restart: unless-stopped

```

## .env:

```txt
MINIO_ROOT_USER=[user]
MINIO_ROOT_PASSWORD=[pass]
```

## prometheus.yml:

```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: "codelab-monitor"

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: prometheus

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: nifi
      # metrics_path defaults to '/metrics'
      # scheme defaults to 'http'.

    static_configs:
      - targets: ["[IP_ADDRESS]:9093"]
```

> [!TIP]  
> If you want to run above code, go [here](./How_to_run_it.md) for detais.