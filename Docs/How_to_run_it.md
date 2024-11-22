# Summary

- [How to run the code](#how-to-run-the-code)
    - [Docker instalation](#docker-instalation)
    - [Download the source code from repository and change all required parameters](#download-the-source-code-from-repository-and-change-all-required-parameters)
    - [What to replace](#what-to-replace)
    - [How to run the containers](#how-to-run-the-containers)
- [File strukture](#file-strukture)


# How to run the code

### Docker instalation:

- docker,
- docker compose
- docker-desktop \[**optional**\]

> Find the instruction in the internet

### Download the source code from repository and change all required parameters

```Bash
###--- How to replase the code
variable = [IP_ADDRESS]

###--- replaced value
variable = 192.168.1.1
```

### What to replace:

- docker-compose.yml
    - Redis section:
        - \[set_up_the_password_here\]
    - Kafka
        - \[IP_ADDRESS\]
	- postgres
		- \[user]
		- \[password\]
	- pgadmin
		- \[EMAIL_address\]
		- \[password\]
- .env
    - \[user\]
    - \[pass\]
- Prometheus conf
    - \[IP_ADDRESS\]

> [!NOTE]  
> Username and password can be created and set by you. You will be using them later to log in to the system

> [!WARNING]  
> IP_ADDRESS is the host addres where your docker is installed. That could be for example "localhost"   

### How to run the containers:

```Bash
###----START-----

###--- run docker compose with logs 
docker-compose up
 or
sudo docker-compose up

###--- run docker compose without logs 
docker-compose up -d
 or
sudo docker-compose up -d

##--- run docker compose with streaming log to the file [recommended]
docker-compose up > logs.log
 or
sudo docker-compose up > logs.log


###----STOP-----

press "Ctrl + C"

###--- remove / stop containers
docker-compose down
 or
sudo docker-compose down
```

# File strukture

- NiFI_home_dir
    - docker-compose.yml
    - Pointainer
        - docker-compose.yml
    - Docker_Mount/
        - V_MINIO_storage/
        - V_PostgreSQL-data/
        - V_NiFi_VOLUME/
        - .env
        - V_prometheus.yml
