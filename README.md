# Raspberry Pi Zero – Hotspot Setup and Recovery Guide

## 1. Set Up the Pi as a Wi-Fi Hotspot
Use when: Setting up a self-contained control system so irrigation, lighting, and automation continue to operate without relying on external Wi-Fi or internet. This is also used when preparing the system for handover, allowing a new occupant to immediately connect via tablet or laptop and operate or modify the setup without any network configuration.

### Install Required Packages

```bash
sudo apt update
sudo apt install hostapd dnsmasq -y

sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```

### Backup Existing Configuration

```bash
sudo cp /etc/dhcpcd.conf /etc/dhcpcd.conf.backup

if [ -f /etc/dnsmasq.conf ]; then
    sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.backup
fi
```

### Configure Static IP

Edit:

```bash
sudo nano /etc/dhcpcd.conf
```

Add to the end of the file:

```text
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

### Configure DHCP Server

Backup the default file:

```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Create a new configuration:

```bash
sudo nano /etc/dnsmasq.conf
```

Contents:

```text
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.50,255.255.255.0,24h
```

### Configure Wi-Fi Access Point

Create:

```bash
sudo nano /etc/hostapd/hostapd.conf
```

Contents:

```text
interface=wlan0
driver=nl80211
ssid=NodeRedPi
hw_mode=g
channel=6
wmm_enabled=0
auth_algs=1
wpa=2
wpa_passphrase=REPLACE_WITH_PASSWORD
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```

### Point hostapd to the Configuration

Edit:

```bash
sudo nano /etc/default/hostapd
```

Set:

```text
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### Enable Services

```bash
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
```

### Reboot

```bash
sudo reboot
```

---

## 2. Confirm the Hotspot Is Working
Use when: After setup or changes, to verify devices can connect reliably, Node-RED is accessible, IP assignment is correct, and the system recovers properly after reboot or power loss.

### Connect a Device

Connect a laptop, phone or tablet to:

```text
SSID: NodeRedPi
Password: <your password>
```

### Verify IP Address Assignment

The connected device should receive an address in the range:

```text
192.168.4.2 - 192.168.4.50
```

### Verify Node-RED

Open:

```text
http://192.168.4.1:1880
```

If using the Dashboard:

```text
http://192.168.4.1:1880/ui
```

### Verify SSH Access

```bash
ssh pi@192.168.4.1
```

### Verify Automatic Recovery

1. Power off the Pi.
2. Wait 10 seconds.
3. Power it back on.

Confirm:

* Hotspot appears automatically.
* Devices reconnect automatically.
* Node-RED is accessible.

---

## 3. Revert Hotspot and Connect to a New Wi-Fi Network
Use when: Moving the system from standalone operation to integration with an existing Wi-Fi network, typically when a new owner wants broader network access, remote control, or integration with their own infrastructure.

### Connect to the Pi Hotspot

Connect your laptop to:

```text
SSID: NodeRedPi
```

SSH into the Pi:

```bash
ssh pi@192.168.4.1
```

### Restore Normal Wi-Fi Configuration

Edit:

```bash
sudo nano /etc/dhcpcd.conf
```

Remove:

```text
interface wlan0
static ip_address=192.168.4.1/24
nohook wpa_supplicant
```

Alternatively restore the backup:

```bash
sudo cp /etc/dhcpcd.conf.backup /etc/dhcpcd.conf
```

### Restore dnsmasq Configuration

```bash
sudo rm -f /etc/dnsmasq.conf

if [ -f /etc/dnsmasq.conf.orig ]; then
    sudo mv /etc/dnsmasq.conf.orig /etc/dnsmasq.conf
fi
```

### Configure the New Wi-Fi Network

Run:

```bash
sudo raspi-config
```

Navigate to:

```text
System Options
  → Wireless LAN
```

Enter:

* New SSID
* New Password

### Disable Hotspot for Future Boots

Do not stop the services, as this will disconnect your current session.

Instead disable them so they do not start after the reboot:

```bash
sudo systemctl disable hostapd
sudo systemctl disable dnsmasq
```

### Reboot

```bash
sudo reboot
```

The hotspot will remain active until the Pi reboots.

After reboot:

* The hotspot will no longer be available.
* The Pi will connect to the configured Wi-Fi network.
* Connect your laptop to the same Wi-Fi network.
* Access the Pi using its new IP address.

### Find the Pi's New IP Address

Check your router's DHCP client list and locate the Raspberry Pi.

Connect using:

```bash
ssh pi@<pi-ip-address>
```

Or access Node-RED:

```text
http://<pi-ip-address>:1880
```

---

## 4. Change to a New Wi-Fi Network (Without Using Hotspot)
Use when: Updating Wi-Fi credentials after router changes or network updates while keeping all system services running without interruption.

### Run raspi-config

```bash
sudo raspi-config
```

Navigate to:

```text
System Options
  → Wireless LAN
```

Enter:

* New SSID
* New Password

### Reboot

```bash
sudo reboot
```

### Confirm Connection

Check the router's DHCP client list and locate the Raspberry Pi.

Confirm Node-RED is available:

```text
http://<pi-ip-address>:1880
```
