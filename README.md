# librenms-oxidized_docker
Simple Portainer/Docker setup for LibreNMS + Oxidized

## Purpose
This is a basic guide to setting up a LibreNMS instance with Oxidized all inside a single Portainer Stack. The same basic steps would work with Docker Compose if you make a few modifications. This is built off the [official docker compose example](https://github.com/librenms/docker/tree/master/examples/compose).

This writeup includes the compose and config files I used, as well as various instructions for things you need to do as you move along.

## What is different?
There are a couple of main things I changed from the official compose example.
* Since we are using Portainer, you need to use `stack.env` to reference your .env files. If you are curious about this part, you can read more [here](https://www.portainer.io/blog/using-env-files-in-stacks-with-portainer).
* I used named volumes for each access to the various configs/files.
* Most importantly for this writeup, I added the oxidized container into the stack.

For Oxidized, the config is changed to use git to store configs locally and to access the librenms container using its containername in the URL.

## Prerequisite
1. Host OS with Docker installed. I used Ubuntu 24.02.
2. Portainer installed. You can find instructions [here](https://docs.portainer.io/start/install-ce/server/docker/linux)

## Setup

### Prep
1. Go to the [official docker compose example](https://github.com/librenms/docker/tree/master/examples/compose) and download the 3 env files (`.env`, `librenms.env`, `msmtpd.env`)

### Portainer Stack
1. Login to Portainer and connect to the local environment, assuming you didn't setup another environment.
2. Go into `Stacks` and click `+ Add stack`
3. Paste the [custom compose file](compose.yml) into the web editor.
5. Click `Load variables from .env file` at the bottom. You will need to do this step three times to load each of the 3 .env files you downloaded in the Prep step above.
6. Modify the variables as needed. The primary ones you will probably need to update are `LIBRENMS_SNMP_COMMUNITY` and all the `SMTP` values. Go ahead and set a good `MYSQL_PASSWORD` while you are at it.
7. Click `Deploy the stack`

### Initial Setup
1. Connect to the LibreNMS web interface using your browser. `http://[docker host or IP]:3300/`
   - if you changed the port mappings in the compose file, make sure to adjust.
2. Complete the initial setup steps. On the validation page, you may see some errors about the Poller. Those cleared up for me after a few minutes and went all green.
3. Go ahead and connect to at least one or two devices so you can test config backups later. How you go about this will depend on a number of factors. For simple testing, you can just add them manually by IP and not worry about auto-discovery.
4. Go to ``http://[docker host or IP]:3300/api-access/` and generate a token. Save this for later.

### Oxidized Setup
1. Once your stack comes up, you need to stop the `librenms_oxidized` container. There is a config change you will need to make before things will work.
2. Find the mount point for your oxidized configs. You can find this in Portainer by going to `Volumes` and looking for `librenms_oxidized-config`. The `Mount point` is what you are looking for. In my case it was `/var/lib/docker/volumes/librenms_oxidized-config/_data`.
3. SSH (or console) to your docker host.
4. Edit the oxidized config file: `sudo nano /var/lib/docker/volumes/librenms_oxidized-config/_data/config`
5. You can start with this [customized config file](oxidized_config)
   - Make sure to edit the usernames and passwords. There is a default at the top, and in the `models` section there is a template for doing a login for each model type if needed.
   - On the last line, you will need to put the API token you got in the Initial Setup step earlier.
   - In the `source` section there is a URL. You will **not** need to edit that since both containers are in the same stack. It will be accessing it using the container name and the internal container port.
6. Go into Portainer and start the `librenms_oxidized` container again.

### Enable Oxidized
1. Use a browser to access LibreNMS web interface: `http://[docker host or IP]:3300/`
2. Go to the gear in the top right, and go to `Global Settings`.
3. Go to the `External` tab.
4. Enter the URL as `http://librenms_oxidized:8888`. Keep in mind, this is the internal container name and port, so unless you modified the container name, this should work.
5. Enable the config versioning option and the return of groups.
6. Variable mapping - here you may need to add some mapping so that the OS that LibreNMS passes along will match a model that Oxidized is familiar with. For instance, I had to add `arubaos-cx` > `aoscx` for my CX switches. Once things are connected, if your Oxidized logs show errors similar to `{:name=>"[IP of switch}", :model=>"[LibreNMS OS value]", :group=>nil} raised Oxidized::ModelNotFound with message...` then you will need to add mapping here.
   - If needed, just set the `Source` dropdown to `os`, set the dropdown on the left to `Match` and enter the LibreNMS OS (it's what showed up in the error above). Then the `Target` should be set to `os` and in `Replacement`, enter a value that exists in Oxidized. You can find the list of supported models [here](https://github.com/ytti/oxidized/blob/master/docs/Supported-OS-Types.md).
7. Now turn on the top option to `Enable Oxidized Support`.

## The End
You can monitor the Oxidized logs from Portainer to make sure it is able to connect to the switches. Once the initial backups complete, you should be able to view them from within the device page.
