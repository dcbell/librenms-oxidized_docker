# librenms-oxidized_docker
Simple Portainer/Docker setup for LibreNMS + Oxidized

## Purpose
This is a basic guide to setting up a LibreNMS instance with Oxidized all inside a single Portainer Stack. The same basic steps would work with Docker Compose if you make a few modifications. This is built off the official [docker compose example](https://github.com/librenms/docker/tree/master/examples/compose).

## Prerequisite
1. Host OS with Docker installed. I used Ubuntu 24.02.
2. Portainer installed. You can find instructions [here](https://docs.portainer.io/start/install-ce/server/docker/linux)

## Setup
### Prep
1. Go to the official docker compose example and download the 3 env files (`.env`, `librenms.env`, `msmtpd.env`)
### Portainer Stack
1. Login to Portainer and connect to local, assuming you didn't setup another environment.
2. Go into `Stacks` and click `+ Add stack`
3. Paste in the [custom compose file](compose.yml).
