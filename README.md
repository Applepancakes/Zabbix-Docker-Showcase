**Zabbix-Docker-Showcase**


# Zabbix-Docker-Showcase

## Table of Contents
- [Overview](#overview)
- [Ground Rules](#ground-rules)
- [Containers](#containers)
- [User Credentials](#user-credentials)
- [General Use & Maintenance](#general-use--maintenance)
- [Troubleshooting the Containers](#troubleshooting-the-containers)
- [Docker Information](#docker-information)
- [Various Resources](#various-resources)

## Overview
This repository demonstrates how to deploy and manage Zabbix using Docker containers. Zabbix is a powerful open-source monitoring solution for various IT components, including networks, servers, virtual machines (VMs), and cloud services.

## Ground Rules
- Zabbix documentation features multiple examples of Docker containers.
- Zabbix does not require MIBs, though they can be added.
- Zabbix does not need SNMP traps unless requiring data more frequently than 1-minute intervals.
- Docker Desktop only supports bridge and host networks; Windows cannot support macvlan networks.

### Step 1
Create the necessary directories for the MySQL and Zabbix server:

```
mkdir C:\docker\zabbix\server
mkdir C:\docker\zabbix\mysql
```

### Networking
This section is critical for container communication across different ports. To create a custom Docker network, run:

docker network create zabbix-network


## Containers
The following components need to be running in order:
1. MySQL
2. Zabbix Server
3. Zabbix Frontend

### MySQL Component
Store the following command in a .txt file and rename it to .bat for execution:
```
docker run -d --name zabbix-mysql ^
        --network zabbix-network ^
        -v "C:\docker\zabbix\mysql:/var/lib/mysql" ^
        -e MYSQL_ROOT_PASSWORD=r00tpa33 ^
        -e MYSQL_DATABASE=zabbix ^
        -e MYSQL_USER=zabbix ^
        -e MYSQL_PASSWORD=agentpa33 ^
        -e TZ=America/New_York ^
        -p 3306:3306 ^
        mysql:8.0-oracle ^
        --character-set-server=UTF8MB4 ^
        --collation-server=UTF8MB4_bin ^
        --default-authentication-plugin=caching_sha2_password
```

### Zabbix Server Component
```
docker run --name zabbix-server-mysql -t ^
    --network=zabbix-network ^
    -e DB_SERVER_HOST="zabbix-mysql" ^
    -e MYSQL_DATABASE="zabbix" ^
    -e MYSQL_USER="zabbix" ^
    -e MYSQL_PASSWORD="agentpa33" ^
    -e MYSQL_ROOT_PASSWORD="r00tpa33" ^
    -v C:\docker\zabbix\server:/var/lib/zabbix ^
    -p 10051:10051 ^
    --restart unless-stopped ^
    -d zabbix/zabbix-server-mysql:alpine-7.0-latest
```

### Zabbix Frontend Component
```
docker run -d --name zabbix-web-nginx-mysql ^
    --network=zabbix-network ^
    -e ZBX_SERVER_HOST="zabbix-server-mysql" ^
    -e DB_SERVER_HOST="zabbix-mysql" ^
    -e MYSQL_DATABASE="zabbix" ^
    -e MYSQL_USER="zabbix" ^
    -e MYSQL_PASSWORD="agentpa33" ^
    -e MYSQL_ROOT_PASSWORD="r00tpa33" ^
    -e TZ=America/New_York ^
    -p 8080:8080 ^
    --restart unless-stopped ^
    -d zabbix/zabbix-web-nginx-mysql:alpine-7.0-latest
```

## User Credentials
- **Root Account**
  - **Username**: root
  - **Password**: r00tpa33 (set by `MYSQL_ROOT_PASSWORD`)
- **Zabbix User**
  - **Username**: zabbix (set by `MYSQL_USER`)
  - **Password**: agentpa33 (set by `MYSQL_PASSWORD`)

## General Use & Maintenance
### Accessing Docker Containers
To access the Zabbix server container, run:

docker exec -it zabbix-server-mysql /bin/sh


### Order for Starting Containers
1. Start the MySQL container first.
2. Start the Zabbix server container next.
3. Finally, start the Zabbix frontend container.

### Order for Stopping Containers
1. Stop the Zabbix frontend container first.
2. Stop the Zabbix server container next.
3. Stop the MySQL container last.

## Troubleshooting the Containers
- Check for communication issues due to router configuration, APN filters, or IP problems.
- Use the following commands to check logs:

```
    docker logs zabbix-web-nginx-mysql > "C:\tmp\zabbix-web-nginx-mysql_$(Get-Date -Format yyyy-MM-dd).txt"
    docker logs zabbix-server-mysql > "C:\tmp\zabbix-server-mysql_$(Get-Date -Format yyyy-MM-dd).txt"
```

## Docker Information
To install additional tools inside the container:
```
docker exec -u root -it zabbix-server-mysql apk update
docker exec -u root -it zabbix-server-mysql apk add net-snmp-tools
````

### Changing the Subnet of the Docker Network
1. List existing networks:
   
   docker network ls
   
3. Remove the existing network:

   docker network rm zabbix-network
   
4. Create a new network with a specified subnet:

   docker network create zabbix-network
   

### SNMP Walk the Container
```
snmpwalk -v2c -c WorkTest 192.168.8.10
snmpwalk -v2c -c Press 192.168.8.238
```

## Various Resources
- [Zabbix Documentation](https://www.zabbix.com/documentation/current/manual)
- [Docker Documentation](https://docs.docker.com/)
- [Zabbix Docker Wiki](https://github.com/zabbix/zabbix-docker/wiki/Docker-101---Before-starting-with-containerized-Zabbix)
- [DataCamp MySQL Docker Setup](https://www.datacamp.com/tutorial/set-up-and-configure-mysql-in-docker)



