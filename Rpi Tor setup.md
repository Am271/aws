[![](https://raw.githubusercontent.com/iiiypuk/rpi-icon/master/256.png)]()

# Rpi Tor Routing setup
## Wifi ➤ Tor ➤ Ethernet ➤ Internet
#### Required
- tor
- hostapd (if not then change iptable rules)

### Setup configs
> **Assumption**: Local IP address of AP is **192.168.2.1**

#### /etc/tor/torrc
```sh
Log notice file /var/log/tor/notices.log
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 192.168.2.1:9040
DNSPort 192.168.2.1:5353
```
#### Iptables rules setup (run as root or sudo)
```sh
iptables -F
iptables -t nat -F
iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 5353
iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040
sh -c "iptables-save > /etc/iptables.ipv4.nat"
```
#### Logging setup (run as root or sudo)
```sh
touch /var/log/tor/notices.log
chown debian-tor /var/log/tor/notices.log
chmod 644 /var/log/tor/notices.log
```