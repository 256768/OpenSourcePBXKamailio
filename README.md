# SIP proxy Kamailio & PBX Asterisk configuration guide

This guide explains how to install and apply Kamailio and Asterisk configuration files provided in the project repository. It assumes a Linux environment (recommended: Ubuntu 24.04.4 LTS).

## Overview

The repository contains preconfigured files for:
- PBX Asterisk
- SIP proxy Kamailio

The goal is to properly place these configuration files into system directories and ensure both services run with the provided setup.

## Prerequisites
Ensure the following software is installed:
- PBX Asterisk v22.5.2 **(with PJSIP support)**
- SIP proxy Kamailio v6.0.3

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
## Notes

- Ensure no port conflicts (Kamailio and Asterisk must not be running at the same time).
- If using both together, Kamailio typically acts as a SIP proxy routing calls to Asterisk.
- Firewall must allow SIP (port 5060)


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
