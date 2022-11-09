[![](https://raw.githubusercontent.com/iiiypuk/rpi-icon/master/256.png)]()

# Rpi AP / Routing setup
## Wifi ➜ Ethernet ➜ Internet

> **Run all commands with priviledges or as root**

### Setting up DHCP server for AP
##### Required
- isc-dhcp-server
- iptables
- hostapd

### Configs
#### /etc/default/isc-dhcp-server
```sh
INTERFACESv4="wlan0"
```
#### /etc/dhcp/dhcpd.conf
```sh
default-lease-time 600;
max-lease-time 7200;
authoritative;
 
subnet 192.168.2.0 netmask 255.255.255.0 {
 range 192.168.2.2 192.168.2.254;
 option routers 192.168.2.1;
 option domain-name-servers 192.168.2.1, 1.1.1.1;
```
#### /etc/default/hostapd
```sh
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
#### /etc/init.d/hostapd
```sh
DAEMON_DEFS=/etc/default/hostapd
DAEMON_CONF=/etc/hostapd/hostapd.conf
```
#### /etc/hostapd/hostapd.conf
```sh
interface=wlan0
#driver=nl80211

hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=0
macaddr_acl=0
ignore_broadcast_ssid=0

auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP

# This is the name of the network
ssid=<ssid-of-ap>
# The network passphrase
wpa_passphrase=<password>
```
#### /etc/network/interfaces
```sh
source /etc/network/interfaces.d/*
auto lo
allow-hotplug eth0
auto wlan0
iface lo inet loopback

iface eth0 inet dhcp

iface wlan0 inet static
    address 192.168.2.1
    netmask 255.255.255.0
    gateway 192.168.1.1
```
### Bring the adapter down and up to apply the changes set above
```sh
ifdown wlan0 && sudo ifup wlan0
```
### Unblock softblocked WLAN
```sh
rfkill unblock wlan
```
### Start hostapd manually (AP)
```sh
hostapd /etc/hostapd/hostapd.conf &
```
### To kill hostapd
```sh
pkill hostapd
```
### Modifying iptables to route traffic from wlan to eth and vice versa
```sh
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -o eth0 -i wlan0 -j ACCEPT
iptables -A FORWARD -o wlan0 -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```
### Persistent changes to iptables
#### Run to save current table
```sh
sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
#### Edit /etc/rc.local to apply changes on boot (place before 'exit 0')
```sh
iptables-restore < /etc/iptables-tor.ipv4.nat
```