---
description: Setup server for Home Assistant and automation tools.
---

# HomeServer

### Install basic tools

```text
sudo apt install python3-pip python3-venv
sudo apt install build-essential libssl-dev libffi-dev python-dev
```

### Install Home Assistant

1. Create new user `sudo useradd hass-user`
2. Create new venv in home directory of new user `python3 -m venv homeassistant`
3. Activate venv  `cd homeassistant` `source bin/activate`
4. Install wheel  `python3 -m pip install wheel`
5. Install HASS \(in case of errors check if you have installed basic tools\) `python3 -m pip install homeassistant`
6. You can run home assistant by calling `hass`
7. Copy home assistant config folder to user home directory or keep it as it is for default configuration
8. Ensure that homeassistant and .homeassistant folders are owned by hass-user

```text
sudo chown -R hass-user:hass-user /home/hass-user/homeassistant/
sudo chown -R hass-user:hass-user /home/hass-user/.homeassistant/
```

#### References

{% embed url="https://www.home-assistant.io/docs/installation/armbian/" %}



{% embed url="https://www.home-assistant.io/docs/installation/raspberry-pi/" %}

### Start Home Assistant after boot

Check which task manager OS use, Armbian use systemd. You can verify by `ps -p 1 -o comm=`

Create new daemon configuration for Python virtual environment  
`/etc/systemd/system/home-assistant@hass-user.service`

```text
[Unit]
Description=Home Assistant
After=network-online.target

[Service]
Type=simple
User=%i
ExecStart=/home/hass-user/homeassistant/bin/hass -c "/home/hass-user/.homeassistant"

[Install]
WantedBy=multi-user.target
```

Reload daemon   
`sudo systemctl --system daemon-reload`

Enable service autostart  
`sudo systemctl enable home-assistant@hass-user.service`

Start service  
`sudo systemctl start home-assistant@hass-user.service`

Show log from service  
`sudo journalctl -f -u home-assistant@hass-user.service`

Restart service and show boot log  
`sudo systemctl restart home-assistant@hass-user && sudo journalctl -f -u home-assistant@hass-user`

#### References

{% embed url="https://www.home-assistant.io/docs/autostart/systemd/" %}

## Docker 

### Basic commands

```bash
# list containers 
docker container ls --all
# start already created container 
docker start container_name
# update container policy, e.g. autostart after reboot
docker update --restart unless-stopped container_name
```

### Network

Containers are available on exposed port from other network devices, e.g. `docker run -p 8086:8086` is exposed on port 8086. But if you want to established communication between containers you need to create network between them. The most basic type of network is bridge.

{% embed url="https://docs.docker.com/network/bridge/" %}

If you want to address other container from container, both have to be inside a one bridge and you have to address other container by his domain name \(not IP address\).



