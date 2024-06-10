# librenms-oxidized_docker
Simple Portainer/Docker setup for LibreNMS + Oxidized

## Purpose
This is a basic guide to setting up a LibreNMS instance with Oxidized all inside a single Portainer Stack. The same basic steps would work with Docker Compose if you make a few modifications. This is built off the official [docker compose example](https://github.com/librenms/docker/tree/master/examples/compose).

## What is different?
There are a couple of main things I changed from the official compose example.
* Since I'm running in Portainer, you need to use `stack.env` to reference your .env files.
* I setup some named volumes.
* Most importantly for this writeup, I added the oxidized container into the stack.

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
4. Click `Load variables from .env file` at the bottom. You will need to do this step three times to load each of the 3 .env files you downloaded in the Prep step above.
5. Modify the variables as needed. The primary ones you will probably need to update are `LIBRENMS_SNMP_COMMUNITY` and all the `SMTP` values. Go ahead and set a good `MYSQL_PASSWORD` while you are at it.
6. Click `Deploy the stack`

### Oxidized Setup
1. Once your stack comes up, you need to stop the `librenms_oxidized` container. There is a config change you will need to make before things will work.
2. Find the mount point for your oxidized configs. You can find this in Portainer by going to `Volumes` and looking for `librenms_oxidized-config`. The `Mount point` is what you are looking for. In my case it was `/var/lib/docker/volumes/librenms_oxidized-config/_data`.
3. SSH (or console) to your docker host.
4. Edit the oxidized config file: `sudo nano /var/lib/docker/volumes/librenms_oxidized-config/_data/config`
