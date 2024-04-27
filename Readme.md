# Docker Compose Files

This repository contains Docker Compose files for deploying various DFIR (Digital Forensics and Incident Response) tools.

## DFIR-IRIS

**GitHub Repository:** [dfir-iris/iris-web](https://github.com/dfir-iris/iris-web)  
**Documentation:** [DFIR-IRIS Documentation](https://docs.dfir-iris.org/getting_started/)

## Portainer

Portainer is a lightweight management UI which allows you to easily manage your Docker host or Swarm cluster.

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
      - "8000:8000"
    environment:
      - SPLUNK_START_ARGS=--accept-license
      - SPLUNK_PASSWORD=admin@123 # 8 Char Password required
    command: start
```
## ELK

## Velociraptor

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

TheHive is a scalable  and open source Security Incident Response Platform, tightly integrated with MISP for threat intelligence analysis.

## Wazuh

**GitHub Repository:** [wazuh/wazuh-docker](https://github.com/wazuh/wazuh-docker)  
**Documentation:** [Wazuh Docker Deployment](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html)

Wazuh is a free, open source and enterprise-ready security monitoring solution for threat detection, integrity monitoring, incident response and compliance.

