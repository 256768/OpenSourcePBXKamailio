# SIP proxy Kamailio & PBX Asterisk configuration guide for laboratories at BUT FEEC

This guide explains how to install and apply Kamailio and Asterisk configuration files provided in the project repository. It assumes a Linux environment (recommended: Ubuntu 24.04.4 LTS).

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
- SIP proxy Kamailio v6.0.3
- Docker

## Clone the repository

```bash
https://github.com/256768/OpenSourcePBXKamailiohttps://github.com/256768/OpenSourcePBXKamailio/releases/new
cd OpenSourcePBXKamailio/
```

## Kamailio configuration

### Locate configuration files

Kamailio configs are typically located in */etc/* under:

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
### Docker setup
For monitoring purposes it is necessary to install following packages:
- Homer
- Prometheus
- Grafana
- process-exporter
- node-exporter

These packages can be installed via Docker, run the following in terminal.

### Prometheus
Run to install:
```bash
docker run -d --name prometheus -p 9090:9090 -v /etc/prometheus/prometheus.yml:/config/config.yml prom/prometheus:latest --config.file=/config/config.yml
```
Insert this config into the ```/etc/prometheus/prometheus.yml``` configuration file:
```bash
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['<ip-address>:9091']
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['<ip-address>:9100']
  - job_name: 'process-exporter'
    static_configs:
      - targets: ['<ip-address>:9256']
```

### process-exporter
Run to install:
```bash
docker run -d --name process-exporter -p 9256:9256 -v /etc/process_exporter/process-exporter.yml:/config/config.yml -v /proc:/host/proc ncabatoff/process-exporter -config.path /config/config.yml -procfs /host/proc
```
Insert config into the ```/etc/process_exporter/process-exporter.yml``` configuration file:
```bash
process_names:
  - name: "kamailio"
    cmdline:
      - "^/usr/local/sbin/kamailio"

  - name: "asterisk"
    cmdline:
      - "^/usr/sbin/asterisk"
```

### node-exporter
Run to install: 
```bash
docker run -d --name nodeexporter --restart unless-stopped -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro,rslave" prom/node-exporter:latest --path.procfs=/host/proc --path.sysfs=/host/sys --path.rootfs=/rootfs
```

## Notes

- Ensure no port conflicts (Kamailio and Asterisk must not be running at the same time).
- Firewall must allow SIP (port 5060).
- When installing packages via Docker, create the configuration file first.
- Ensure that the _kamailio_ and _asterisk_ binaries are present in the specified paths (```/usr/local/sbin/kamailio``` and ```/usr/sbin/asterisk```).


## Troubleshooting
These configuration files should not contain any errors, in case of errors (for example the SIP proxy/PBX exiting) you can view logs by:

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
