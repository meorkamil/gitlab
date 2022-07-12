## GitLab home lab
___
Quick GitLab repo using docker. 

- Basic understand on implementation of runners
- Understanding how Ansible can be integrate with runners

## Requirements
___
- __[Docker](https://docs.docker.com/engine/install/)__
- __[Docker Compose](https://docs.docker.com/compose/install/)__

## Clone repo
---
Create a based directory eg; `/opt/docker` .

Clone this repo
```bash
git clone https://github.com/meorkamil/gitlab.git
```
## Environment Setup
---
Edit `volumes` and `environment` specs on `docker-compose.yml` based on your environment Win/Linux/MacOS or your needs
```yaml
version: "3.8"

services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.kamil.net'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.kamil.net'
        registry_external_url 'http://registry.gitlab.kamil.net:5000'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './config:/etc/gitlab'
      - './logs:/var/log/gitlab'
      - './data:/var/opt/gitlab'

  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - './docker-registry/auth:/auth'
      - './docker-registry/data:/data'
```
## Enable Local registry
---
You can use the existing gitlab registry. Please [refer](https://docs.gitlab.com/ee/administration/packages/container_registry.html) GitLab Container Registry Docs.

Create registry credentials
```bash
# Create directory to store credentials
mkdir -p /opt/docker/gitlab/docker-registry

# Use docker library httpd container to create httpasswrd 
docker run --rm -it -v "/opt/docker/gitlab/docker-registry:/backup" httpd bash

# Run this inside httpd container 
cd /backup
htpasswd -Bc registry.password <username>
```
Credential files `registry.password` must be added in the registry container env `REGISTYR_AUTH_HTPASSWD_PATH` 

**Remarks**

Please enable insecure registry in `daemon.json`

Reference: https://docs.docker.com/registry/insecure/

## Start up
___

Bring up services
```bash
docker-compose up -d
```

## Runner Registration
---
Please refers gitlab docs: 
https://docs.gitlab.com/runner/install/docker.html

## GitLab CI
---
[Docs](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html) -.gitlab-ci.yml
