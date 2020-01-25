## Aasivak

This program bridges your Hi-Kumo heater/air conditioner system with Home Assistant.

It uses Home Assistant MQTT discovery mechanism: https://www.home-assistant.io/docs/mqtt/discovery/ 

Checkout the project, either edit ```config/default.yml``` or create a ```config/local.yml``` with the properties you need to override. ```api_username``` and ```api_username``` are your Hi-Kumo app credentials. Then run ```Aasivak.py```.

Please use, clone and improve. Some things are not supported. It was tested only with my own devices and installation. This is a very early release, based on reverse engineering of the network traffic. I have no relation to Hitachi (other than using their product) and they may not like it. Use at your own perils.

## Installation

### Setup MQTT discovery on HA
You will need an MQTT broker: [MQTT broker](https://www.home-assistant.io/docs/mqtt/broker/)

And to activate MQTT discovery: [MQTT discovery](https://www.home-assistant.io/docs/mqtt/discovery/)

### Clone the Aasivak repo
```shell script
git clone https://www.github.com/dotvav/aasivak.git
cd aasivak
pip3 install -r requirements.txt
```

### Change the configuration
You can either update the ```config/default.yml``` file or create a new file named ```config/local.yml```. The keys that are present in the local config will override the ones in the default config. If a key is absent from local config, Aasivak will fallback to the value of the default config. I recommend keeping the default config as is and make all the changes in the local config file so that you don't lose them when the default file gets updated from git.

You will need to set the ```api_username``` and ```api_password``` keys. You may need to set the ```mqtt_host``` key if your MQTT broker is not running on the same machine as Aasivak. 

Property | Usage | Note
--- | --- | ---
**`api_username`** | your Hi-Kumo login | You **must** set this.
**`api_password`** | your Hi-Kumo password | You **must** set this.
`api_url` | the Hi-Kumo API url | you should probably not touch this.
`api_user_agent` | the user agent string that Aasivak will use in the header of its HTTP requests to the Hi-Kumo API.
`mqtt_discovery_prefix` | the MQTT topic prefix that HA is monitoring for discovery | You should probably not touch this. HA's default is `homeassistant`. 
`mqtt_state_prefix` | the MQTT topic prefix that Aasivak will use to broadcast the devices state to HA | You should probably not touch this.
`mqtt_command_prefix` | the MQTT topic prefix that Aasivak will listen to for HA commands | You should probably not touch this.
`mqtt_reset_topic` | the MQTT topic where Aasivak receives reset commands | Send any message on this topic to tell Aasivak it must re-register all the devices. You should create an automation to do that every time HA starts.
**`mqtt_host`** | the host name or ip address of the MQTT broker | Use `localhost` or `127.0.0.1` if the MQTT broker runs on the same machine as Aasivak.
`mqtt_client_name` | the name that Aasivak will us on MQTT | You should probably not touch this.
`mqtt_username` | the MQTT broker username | This is needed only if the MQTT broker requires an authenticated connection.
`mqtt_password` | the MQTT broker password | This is needed only if the MQTT broker requires an authenticated connection.
`temperature_unit` | the temperature measurement unit | `°C` by default.
`refresh_delays` | list of waiting durations before calling the Hi-Kumo API to refresh devices state | If you set `[2, 5, 10, 30]` then Aasivak will call the Hi-Kumo API to refresh its state after 2s, then 5s, then 10s, and then every 30s. The delay is reset to 2s when Aasivak receives a command from HA. Some randomness is added to these delays: every time Aasivak needs to wait, it adds or remove up to `logging_delay_randomness/2` to the delay. 
`refresh_delay_randomness` | maximum number of seconds to add to all the waiting durations | See `refresh_delays`. Use `0` for no randomness.
`logging_level` | Aasivak's logging level | INFO


### Start Aasivak manually
```shell script
python3 Aasivak.py
```

### Start Aasivak as a systemd service
Create the following ```/etc/systemd/system/aasivak.service``` file (change the paths as required):

```
[Unit]
Description=Aasivak
Documentation=https://github.com/dotvav/aasivak
After=network.target

[Service]
Type=simple
User=homeassistant
WorkingDirectory=/home/homeassistant/aasivak
ExecStart=/usr/bin/python3 /home/homeassistant/aasivak/Aasivak.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
You may want to start this after the MQTT broker or HA has started: add the appropriate ```After=``` statement.

Run the following to enable and run the service, and see what its status is:
```shell script
sudo systemctl enable aasivak.service
sudo systemctl start aasivak.service
sudo systemctl status aasivak.service
```

## Pull the latest version from Github
Get the latest source
```shell script
cd aasivak
git pull origin master
```
Then restart the systemd service (if you created one):
```shell script
sudo systemctl status aasivak.service
```

## Dependencies

- requests
- paho-mqtt
- pyyaml


