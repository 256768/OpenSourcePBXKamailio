# SIP proxy Kamailio & PBX Asterisk configuration guide for laboratories at BUT FEEC

This guide explains how to install and apply Kamailio and Asterisk configuration files provided in the project repository. It assumes a Linux environment (recommended: Ubuntu 24.04.4 LTS).

Note: Do not run both services at the same time. The environment is configured so that students can switch between services (measure one, deactivate, then measure the other) without reconfiguring the system. This structure also ensures that future lab administrators have a clear baseline for how the tasks were originally architected.

## Overview

The repository contains preconfigured files for:
- PBX Asterisk
- SIP proxy Kamailio
- Prometheus
- process-exporter

The goal is to properly place these configuration files into system directories and ensure both services run with the provided setup.

## Prerequisites
Ensure the following software is installed:
- PBX Asterisk v22.5.2 **(with PJSIP support)**
- SIP proxy Kamailio v6.0.3 **(with MySQL module)**
- MariaDB server used for Kamailio
  
First, install all necessary prerequisites:

```bash
apt update && apt install docker.io docker-compose git
```
And now create directories for configuration files:
```bash
mkdir -p /etc/prometheus /etc/process_exporter
```

## Clone the repository
```bash
git clone https://github.com/256768/OpenSourcePBXKamailio
cd OpenSourcePBXKamailio/
```

## Kamailio configuration

### Locate configuration files

Kamailio configs are typically located in */etc/* or */usr/local/etc* directory under:

```
kamailio/
├── kamailio.cfg
├── modules/
└── ...
```

### Copy files to the *kamailio/* directory

Default Kamailio config directory:

```
/etc/kamailio/
```
Note: if you installed Kamailio by compiling it from the source code, then your config directory is probably */usr/local/*.

Copy configuration files:
```bash
sudo cp -r kamailio/* /etc/kamailio/ # or /usr/local/etc/kamailio/
```
**Important**: it is necessary to edit the ```kamctlrc``` file, too, but for security reasons, the exact configuration file won't be published in this repository. It is necessary to uncomment these variables:
```SIP_DOMAIN```, ```DBENGINE```, ```DBHOST```, ```DBRWUSER```, ```DBRWPW```, ```DBROUSER``` and ```DBROPW```. By default, the variables should work as they are (if you did not change the DB password, etc.), except for SIP_DOMAIN. Set this variable to your domain, for example ```sip.yourdomain.org``` or your server IP address, for example ```192.168.128.1```. Please note, that the ```kamctlrc``` configuration file can be located both in locations ```/etc/kamailio/kamctlrc``` and ```/usr/local/etc/kamailio/kamctlrc```. Make sure you edit the one that corresponds to your installation prefix, or use a symbolic link to ensure kamctl can find your configuration.

### Adjust permissions

```bash
sudo chown -R kamailio:kamailio /etc/kamailio # or /usr/local/etc/kamailio
```

### Enable and start Kamailio

```bash
sudo systemctl enable kamailio
sudo systemctl restart kamailio
```

Check status:

```bash
sudo systemctl status kamailio
```
Create the database:
```bash
kamdbctl create
```
Here you will have to choose the correct encoding.

## Asterisk configuration

### Locate configuration files

Asterisk configuration files are located in:

```
asterisk/
├── pjsip.conf
├── extensions.conf
├── modules.conf
└── ...
```

### Copy files to system directory

Default Asterisk config directory:

```
/etc/asterisk/
```

Copy files:

```bash
sudo cp -r asterisk/* /etc/asterisk/
```

### Set permissions

```bash
sudo chown -R asterisk:asterisk /etc/asterisk
```

### Restart Asterisk

```bash
sudo systemctl enable asterisk
sudo systemctl restart asterisk
```

Access CLI:

```bash
sudo asterisk -rvvv
```

## Verification

### Kamailio

Check if Kamailio is listening on SIP port (default 5060):

```bash
ss -tuln | grep 5060
```

### Asterisk

Inside Asterisk CLI:

```bash
pjsip show endpoints
```

## Docker setup
For monitoring purposes it is necessary to install following packages:
- Homer 7
- Prometheus
- Grafana
- node-exporter
- process-exporter

These packages can be installed via Docker, run the following in terminal.

### Homer 7
Homer 7 pulls also the Prometheus, Grafana and node-exporter packages, but it is necessary to edit them afterwards, so install Homer 7 first.
Clone the Homer 7 repository:
```bash
git clone https://github.com/sipcapture/homer7-docker
```
Navigate to the directory:
```bash
cd homer7-docker/heplify-server/hom7-prom-all
```
Start the Docker container:
```bash
docker-compose up -d
```

### Prometheus
Insert the ```prometheus.yml``` configuration file into the ```/etc/prometheus/``` directory and then run: 
```bash
docker stop prometheus
docker rm prometheus
docker run -d --name prometheus -p 9090:9090 -v /etc/prometheus/prometheus.yml:/config/config.yml prom/prometheus:latest --config.file=/config/config.yml
```

### node-exporter
Run to install: 
```bash
docker stop nodeexporter
docker rm nodeexporter
docker run -d --name nodeexporter --restart unless-stopped -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro,rslave" prom/node-exporter:latest --path.procfs=/host/proc --path.sysfs=/host/sys --path.rootfs=/rootfs
```

### process-exporter
Insert the ```process-exporter.yml``` configuration file into the ```/etc/process_exporter/``` directory and then run: 
```bash
docker run -d --name process-exporter -p 9256:9256 -v /etc/process_exporter/process-exporter.yml:/config/config.yml -v /proc:/host/proc ncabatoff/process-exporter -config.path /config/config.yml -procfs /host/proc
```
### EasySIPp
Install by running:
```bash
docker run -dt --name easySIPp --network host --name easysipp -v easysipp/forms.py:/app/easySIPp/forms.py krndwr/easysipp -p 8080:8080
```


## Notes

- Ensure no port conflicts (Kamailio and Asterisk must not be running at the same time).
- Firewall must allow SIP (port 5060).
- When installing packages via Docker, create the configuration file first.
- Ensure that the *kamailio* and *asterisk* binaries are present in the specified paths (```/usr/local/sbin/kamailio``` and ```/usr/sbin/asterisk```).


## Troubleshooting
In case of errors (for example the SIP proxy/PBX exiting) you can view logs by:

**Kamailio:**
```bash
journalctl -u kamailio -f
```

**Asterisk:**
```bash
journalctl -u asterisk -f
```

## Disclaimer
This project is currently under active development and is not considered stable. Features may change, break, or be removed at any time without prior notice.
Use this software at your own risk. The author provides no guarantees regarding functionality, reliability, or suitability for any purpose and are not responsible for any damage or issues caused by its use.
This project is licensed under the MIT License. See the LICENSE file for more details.

---
---
