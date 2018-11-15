# DON'T FOLLOW THIS GUIDE AS OF YET. I`M STILL WORKING ON IT!

# System Dashboard
------------------------------

Start of the Dahsboard page. I will try to explain the best I can. I will also post the links to the original developers who with the help of their scripts made this Dashboard possible.

**This installation it's based on my Ubuntu Server 16.04. It may or not work on all distros!**

![dashboard](https://user-images.githubusercontent.com/32961904/48569188-60054f80-e901-11e8-81a5-14f26b317f4b.jpg)


# Things you need before you start:

I will not explian how to install these aplications, there are numerous guides on the Internet that you can look at!

Program | Links | Command
------------ | ------------- | -------------
Grafana | https://grafana.com | apt-get install grafana
InfluxDB | https://www.influxdata.com | apt-get install influxdb
Tautulli | https://tautulli.com | [Howto](https://github.com/Tautulli/Tautulli-Wiki/wiki/Installation#linux)
Plex | https://plex.tv | [Download](https://www.plex.tv/media-server-downloads/#plex-media-server)
Ombi | https://ombi.io/ | [Howto](https://github.com/tidusjar/Ombi/wiki/Installation)

# Optional

Program | Links | Command
------------ | ------------- | -------------
Zimbra Mail-Server | https://zimbra.com | [Howto](https://zimbra.org/download/zimbra-collaboration/8.8.9)

# Start

After installing all the required components, you need to create at least 3 InfluxDB databases.

```sh
influx
Connected to http://localhost:8086 version 1.6.4
InfluxDB shell version: 1.6.4
> create database plex_data
> create database plexpy
> create database newplex
> show databases
name: databases
name
----
_internal
plex_data
newplex
plexpy
> quit
```

# Clone the following Repos

```sh
git clone https://github.com/barrycarey/Plex-Data-Collector-For-InfluxDB.git
git clone https://github.com/DirtyCajunRice/grafana-scripts.git
git clone https://github.com/Drewster727/tautulli-influxdb-export.git
```

# Optional Repo for the Zimbra Mail-Server
Follow the intructions there!

```sh
git clone https://github.com/jorgedlcruz/zimbra-grafana.git
```
# Setup the cloned Git repos!

**1. Plex-Data-Collector-For-InfluxDB**

**Requirements**

* Python 3.x
* InfluxDB server

Run `pip install -r requirements.txt`

Python Packages
* [influxdb](https://github.com/influxdata/influxdb-python)
* [plexapi](https://pypi.org/project/PlexAPI/)
* [requests](https://pypi.org/project/requests/)

open the folder **Plex-Data-Collector-For-InfluxDB**, edit the **config.ini** file with the following details:

```sh
[GENERAL]
# Seconds between each polling run
Delay = 10
ReportCombined = True

[INFLUXDB]
Address = localhost
Port = 8086
Database = plex_data
# If Authentication is enabled you must provide an admin user
Username =
Password =
Verify_SSL = False

[PLEX]
# If using multiple servers all must be on same account
Username = YOUR PLEX USERNAME
Password = YOUR PLEX PASSWORD
Servers = YOUR PLEX IP ADDRESS
# If Secure Connections set to required on Plex server HTTPS must be set to True
HTTPS = False
Verify_SSL = False

[LOGGING]
# Valid Options: critical, error, warning, info, debug
Level = error
```
Change permission of the file **plexcollector.py**

```sh
chmod +x plexcollector.py
```
* Create a service

```sh
cd /etc/systemd/system

```
```sh
nano plexdata.service
```
**Paste the following** (change the path where you cloned the repo accordingly)

```sh
[Unit]
Description=PlexData Service
After=multi-user.target

[Service]
Type=idle
WorkingDirectory=/PATH/TO/YOUR/Plex-Data-Collector-For-InfluxDB/
ExecStart=/usr/bin/python3 /PATH/TO/YOUR/Plex-Data-Collector-For-InfluxDB/plexcollector.py
Restart=on-failure
RestartSec=15s
TimeoutSec=15s

[Install]
WantedBy=multi-user.target
```
* Enable the Service

```sh
systemctl enable plexdata.service
```

**2. grafana-scripts**

open the folder **grafana-scripts**, edit the **configuration.py** file with the following details:

```sh
########################### INFLUXDB CONFIG ###########################
influxdb_url = 'localhost'
influxdb_port = 8086
influxdb_username = ''
influxdb_password = ''

############################ OMBI CONFIG ##############################
ombi_url = 'YOUR OMBI HOSTNAME'
ombi_api_key = 'YOUR OMBI API KEY'
ombi_influxdb_db_name = 'newplex'

########################## TAUTULLI CONFIG ############################
tautulli_url = 'YOUR TAUTULLI HOSTNAME'
tautulli_api_key = 'YOUR TAUTULLI API KEY'
tautulli_failback_ip = 'YOUR TAUTULLI IP ADDRESS'
tautulli_influxdb_db_name = 'newplex'
```
* Change permissions of the files **configuration.py** **ombi.py** **tautulli.py**

```sh
chmod +x configuration.py ombi.py tautulli.py
```

* Create a cronjob

```sh
crontab -e
```
```sh
* * * * * /usr/bin/python3 /PATH/TO/YOUR/grafana-scripts/ombi.py --total
* * * * * /usr/bin/python3 /PATH/TO/YOUR/grafana-scripts/tautulli.py
```

**3. tautulli-influxdb-export**

Change permission of the file **plexpy_influxdb_export.py**

```sh
chmod +x plexpy_influxdb_export.py
```

edit the file **plexpy_influxdb_export.service** as follow:

```sh
[Unit]
Description=PlexPy2InfluxDB Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/bin/python /PATH/TO/YOUR/tautulli-influxdb-export/plexpy_influxdb_export.py --plexpyhost YOU_TAUTULLI_HOST --plexpyapikey YOUR_TAUTTULI_API_KEY
Restart=on-failure
RestartSec=15s
TimeoutSec=15s

[Install]
WantedBy=multi-user.target
```

copy the file **plexpy_influxdb_export.service** to your **system** folder

```sh
cp plexpy_influxdb_export.service /etc/systemd/system
```

* Enable the Service
```sh
systemctl enable plexpy_influxdb_export.service
```


# Dashboard

Download the dashboard from this repo and upload it to your Grafana:

![import](https://user-images.githubusercontent.com/32961904/48556703-faa36580-e8e4-11e8-9ebb-4d0f6020933c.jpg)



![image](https://user-images.githubusercontent.com/32961904/48545440-3844c600-e8c6-11e8-9436-5fbf493d3403.png)
