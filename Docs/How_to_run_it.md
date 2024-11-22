# Summary

- [How to run the code](#how-to-run-the-code)
    - [Docker instalation](#docker-instalation)
    - [Get configuration files](#get-configuration-files)
    - [What to replace](#what-to-replace)
    - [How to run the containers](#how-to-run-the-containers)
- [File strukture](#file-strukture)


# How to run the code

### Docker instalation:

- docker,
- docker compose
- docker-desktop \[**optional**\]

> Find the instruction in the internet

### Get configuration files

> [!NOTE]  
> Download the source code from repository and change all required parameters like shown below

```Bash
###--- How to replase the code
variable = [IP_ADDRESS]

###--- replaced value
variable = 192.168.1.1
```

### What to replace:

| File Path | Section | placeholder | description |
| --- | --- | --- | --- |
| docker-compose.yml | redis | [set_up_the_password_here] | set up a password used to log in to redis server |
| docker-compose.yml | Kafka | [IP_ADDRESS] | set up here your IP address you want to use.  <br>Ex: localhost, home VM IP |
| docker-compose.yml | PostgreSQL | [user]  <br>[password] | Set up password and username you want to use to log in |
| docker-compose.yml | pgAdmin | [EMAIL_address]  <br>[password] | Set up an email with proper domain (like example.com, server.loc) and password |
| Docker_Moint / .env | ---  | [user]  <br>[pass] | user and password used to log in to S3 GUI. Set up strong password with smal and large letter and numbers with at least 8 characters. It's required to run the container properly |
| Docker_Moint / V_prometheus.yml | config / nifi | [IP_ADDRESS] | set up here your IP address you want to use.  <br>Ex: localhost, home VM IP |

<br>

> [!NOTE]  
> Username and password can be created and set by you. You will be using them later to log in to the system

> [!WARNING]  
> IP_ADDRESS is the host address where your docker is installed. That could be for example "localhost"   

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

# File structure

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
