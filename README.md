# System Dashboard
------------------------------

Start of the Dahsboard page. I will try to explain the best I can. I will also post the links to the original developers who with the help of their scripts made this Dashboard possible.

**This installation it's based on my Ubuntu Server 16.04. It may or not work on all distros!**

![Dasboard](https://user-images.githubusercontent.com/32961904/54270416-ad2bc200-457f-11e9-94f0-6ae8bc8450cc.jpg)

# Things you need before you start:

I will not explian how to install these aplications, there are numerous guides on the Internet that you can look at!

Program | Links | Command
------------ | ------------- | -------------
Grafana | https://grafana.com | apt-get install grafana
InfluxDB | https://www.influxdata.com | apt-get install influxdb
Tautulli | https://tautulli.com | [Howto](https://github.com/Tautulli/Tautulli-Wiki/wiki/Installation#linux)
Plex | https://plex.tv | [Download](https://www.plex.tv/media-server-downloads/#plex-media-server)
Ombi | https://ombi.io/ | [Howto](https://github.com/tidusjar/Ombi/wiki/Installation)
Telegraf | https://www.influxdata.com | [Howto](https://docs.influxdata.com/telegraf/v1.8/introduction/installation/)

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
* Follow the intructions there!

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

**Change these settings!**
* Username = YOUR PLEX USERNAME
* Password = YOUR PLEX PASSWORD
* Servers = YOUR PLEX IP ADDRESS

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
sudo systemctl enable plexdata.service
```
* Start the Service

```sh
sudo service plexdata start
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
sudo systemctl enable plexpy_influxdb_export.service
```

* Start the Service

```sh
sudo service plexpy_influxdb_export start
```

# Create InfluxDB datasources

![create-datasources](https://user-images.githubusercontent.com/32961904/48622037-ce9be900-e9a5-11e8-8a9b-7907b173c12b.jpg)

```sh
* Name:       Plex_Data
* Type:       influxDB
* URL:        http://localhost:8086
* Access:     Server (Default)
* Database:   plex_data
```
```sh
* Name:       Plex Users
* Type:       influxDB
* URL:        http://localhost:8086
* Access:     Server (Default)
* Database:   plexpy
```
```sh
* Name:       Tautulli
* Type:       influxDB
* URL:        http://localhost:8086
* Access:     Server (Default)
* Database:   newplex
```
# System Stats
System stats that you see on the Dashboard are based on Telegraf! Create a datasource as well if you want to see some system details. Edit the file ***/etc/telegraf/telegraf.conf*** to monitor the services you want!

```sh
* Name:       plex
* Type:       influxDB
* URL:        http://localhost:8086
* Access:     Server (Default)
* Database:   telegraf
```
# Dashboard

Download the dashboard from this repo, import it to your Grafana and adapt the datasources if needed:

![import](https://user-images.githubusercontent.com/32961904/48556703-faa36580-e8e4-11e8-9ebb-4d0f6020933c.jpg)




