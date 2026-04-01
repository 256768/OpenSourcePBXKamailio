# SIP proxy Kamailio & PBX Asterisk configuration guide

This guide explains how to install and apply Kamailio and Asterisk configuration files provided in the project repository. It assumes a Linux environment (recommended: Ubuntu 24.04 LTS).

## Overview

The repository contains preconfigured files for:
- PBX Asterisk
- SIP proxy Kamailio

The goal is to properly place these configuration files into system directories and ensure both services run with the provided setup.

## Prerequisites

Installed required software:
- PBX Asterisk v22.5.2 **(USING PJSIP!)**
- SIP proxy Kamailio v6.0.3

## Clone the repository

```bash
https://github.com/256768/OpenSourcePBXKamailio
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

### 3. Set permissions

```bash
sudo chown -R asterisk:asterisk /etc/asterisk
```

### 4. Restart Asterisk

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

---
---
