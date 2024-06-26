# Docker Compose and Quick install Scripts for Security Tools

This repository contains Docker Compose files for deploying various DFIR (Digital Forensics and Incident Response) tools.

## DFIR-IRIS



**GitHub Repository:** [dfir-iris/iris-web](https://github.com/dfir-iris/iris-web)  
**Documentation:** [DFIR-IRIS Documentation](https://docs.dfir-iris.org/getting_started/)
## Caldera

```bash
git clone https://github.com/mitre/caldera.git --recursive

# Build the docker image. Change image tagging as desired.
# WIN_BUILD is set to true to allow Caldera installation to compile windows-based agents.
# Alternatively, you can use the docker compose YML file via "docker-compose build"
cd caldera
docker build . --build-arg WIN_BUILD=true -t caldera:latest

# Run the image. Change port forwarding configuration as desired.
docker run -p 8888:8888 caldera:latest
```
## Shuffle SOAR
```bash
git clone https://github.com/shuffle/Shuffle
cd Shuffle
docker-compose up -d
```
## Dockge
```yaml
version: "3.8"
services:
  dockge:
    image: louislam/dockge:1
    restart: unless-stopped
    ports:
      # Host Port : Container Port
      - 5001:5001
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/app/data
      # If you want to use private registries, you need to share the auth file with Dockge:
      # - /root/.docker/:/root/.docker

      # Stacks Directory
      # ⚠️ READ IT CAREFULLY. If you did it wrong, your data could end up writing into a WRONG PATH.
      # ⚠️ 1. FULL path only. No relative path (MUST)
      # ⚠️ 2. Left Stacks Path === Right Stacks Path (MUST)
      - /opt/stacks:/opt/stacks
    environment:
      # Tell Dockge where is your stacks directory
      - DOCKGE_STACKS_DIR=/opt/stacks
```
## Portainer

Portainer is a lightweight management UI which allows you to easily manage your Docker host or Swarm cluster.
```yaml
version: '3.2'

services:
  agent:
    image: portainer/agent:2.11.1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:2.11.1
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
    networks:
      - agent_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]

networks:
  agent_network:
    driver: overlay
    attachable: true

volumes:
  portainer_data:
```
## Splunk

Original Splunk Docker Image
```yaml
# Custom Compose File
version: '3'
services:
  splunk:
    image: splunk/splunk:latest
    restart: always
    ports:
      - "8000:8000" #Web Console
      - "9997:9997" #Receiver for Universal forwarder
      - "8089:8089" #Splunk Server for configuring Forwarder Agents
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=admin@123 # 8 Char Password required
    command: start
```
## Splunk Debian Agent
```bash
dpkg -i splunkforwarder-8.2.3-cd0848707637-linux-2.6-amd64.deb
/opt/splunkforwarder/bin/splunk start --accept-license
/opt/splunkforwarder/bin/splunk enable boot-start -systemd-managed 0
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.1.88:9997 -auth admin:admin@123
```
## ELK
```bash
#copy line by line
git clone https://github.com/deviantony/docker-elk.git
cd docker-elk/
sudo docker-compose up setup
sudo docker-compose up -d
#elastic:changeme
```

## Velociraptor
```shell
git clone https://github.com/weslambert/velociraptor-docker
cd velociraptor-docker

#Change credential values in .env as desired

docker-compose up -d
#Access the Velociraptor GUI via https://<hostip>:8889
```
## Guacamole
```yaml
version: '3.6'
services:
  guacamole:
    image: unsafetypin/guacamole
    environment:
    - EXTENSIONS=auth-totp
    volumes:
    - ./config:/config
    ports:
    - "8080:8080"
    restart: always
```
## MISP

**Download:** [MISP Download Page](https://www.misp-project.org/download/)  
**GitHub Repository:** [misp/misp-docker](https://github.com/misp/misp-docker)

MISP (Malware Information Sharing Platform & Threat Sharing) is an open-source threat intelligence platform designed to improve the overall security posture of organizations.

## TheHive
```yaml
version: "3"
services:
  thehive:
    image: strangebee/thehive:5.2
    depends_on:
      - cassandra
      - elasticsearch
      - minio
      - cortex
    mem_limit: 1500m
    ports:
      - "9010:9000"
    environment:
      - JVM_OPTS="-Xms1024M -Xmx1024M"
    command:
      - --secret
      - "mySecretForTheHive"
      - "--cql-hostnames"
      - "cassandra"
      - "--index-backend"
      - "elasticsearch"
      - "--es-hostnames"
      - "elasticsearch"
      - "--s3-endpoint"
      - "http://minio:9000"
      - "--s3-access-key"
      - "minioadmin"
      - "--s3-secret-key"
      - "minioadmin"
      - "--s3-bucket"
      - "thehive"
      - "--s3-use-path-access-style"
      - "--cortex-hostnames"
      - "cortex"
      - "--cortex-keys"
      - "eY2Iio3rty+YCs+fZ7RN43A/Wd5Xg1nB"

  cassandra:
    image: 'cassandra:4'
    mem_limit: 1600m
    ports:
      - "9042:9042"
    environment:
      - MAX_HEAP_SIZE=1024M
      - HEAP_NEWSIZE=1024M
      - CASSANDRA_CLUSTER_NAME=TheHive
    volumes:
      - cassandradata:/var/lib/cassandra
    restart: on-failure

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.12
    mem_limit: 1500m
    expose:
      - "9200"
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    volumes:
      - elasticsearchdata:/usr/share/elasticsearch/data

  minio:
    image: quay.io/minio/minio
    mem_limit: 512m
    command: ["minio", "server", "/data", "--console-address", ":9090"]
    environment:
      - MINIO_ROOT_USER=minioadmin
      - MINIO_ROOT_PASSWORD=minioadmin
    ports:
      - "9190:9090"
    volumes:
      - "miniodata:/data"

  cortex:
    image: thehiveproject/cortex:3.1.7
    depends_on:
      - elasticsearch
    environment:
      - job_directory=/tmp/cortex-jobs
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/cortex-jobs:/tmp/cortex-jobs
    ports:
      - "9001:9001"

volumes:
  miniodata:
  cassandradata:
  elasticsearchdata:
```
TheHive is a scalable  and open source Security Incident Response Platform, tightly integrated with MISP for threat intelligence analysis.

## Wazuh

**GitHub Repository:** [wazuh/wazuh-docker](https://github.com/wazuh/wazuh-docker)  
**Documentation:** [Wazuh Docker Deployment](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html)

Wazuh is a free, open source and enterprise-ready security monitoring solution for threat detection, integrity monitoring, incident response and compliance.

```bash
#single node setup
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.4

cd wazuh-docker
docker-compose up -d
```
```yaml
# Wazuh App Copyright (C) 2021 Wazuh Inc. (License GPLv2)
version: '3'

services:
  generator:
    image: wazuh/wazuh-certs-generator:0.0.1
    hostname: wazuh-certs-generator
    volumes:
      - ./config/wazuh_indexer_ssl_certs/:/certificates/
      - ./config/certs.yml:/config/certs.yml
    environment:
      - HTTP_PROXY=YOUR_PROXY_ADDRESS_OR_DNS
```
## BloodHound
https://support.bloodhoundenterprise.io/hc/en-us/articles/17468450058267-Install-BloodHound-Community-Edition-with-Docker-Compose
```shell
curl -L https://ghst.ly/getbhce | docker compose -f - up

#Download and store docker-compose.yml in the directory from which you would like to run BloodHound CE.
#Open a terminal interface, and from the directory you selected run docker-compose up
#To run BloodHound CE without the need to maintain the terminal interface, use docker-compose up -d, and then docker-compose logs to see the most recent logs from the environment.
#Docker Compose will download the required container images and initialize them. The logs will display the randomized default password to log into your new BloodHound CE environment.
#Copy this password and open a web browser to http://localhost:8080.
#Log in with the default username admin and the password from the logs.
#The password cannot be regenerated. If you lost the password, simply run docker compose down -v and then docker compose up to reset your databases.
#You will be required to change the password, thereafter you're logged into BloodHound CE!

#https://github.com/SpecterOps/BloodHound/blob/main/examples/docker-compose/docker-compose.yml
```
