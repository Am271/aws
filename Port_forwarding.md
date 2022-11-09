### IP forwarding
```sh
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
```
#### **OR** Edit /etc/sysctl.conf and run
```sh
sysctl -p
```
#### to apply settings
### Forwarding rules to basic firewall
> **Assumption**: Public interface is **eth0**, private interface is **eth1**, private IP(eth1) is **10.0.0.1**, private IP(eth0) is **10.0.0.2** and port to forward is port **80**
```sh
iptables -A FORWARD -i eth0 -o eth1 -p tcp --syn --dport 80 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
### Routing traffic
```sh
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 10.0.0.1
iptables -t nat -A POSTROUTING -o eth1 -p tcp --dport 80 -d 10.0.0.1 -j SNAT --to-source 10.0.0.2
```