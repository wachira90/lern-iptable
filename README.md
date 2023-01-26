# lern-iptable
lern-iptable

## UBUNTU 20.04

````
apt update

apt install iptables-persistent

nano /etc/iptables/rules.v4

iptables-restore -t /etc/iptables/rules.v4

service netfilter-persistent reload
````


````
# Limit the number of incoming tcp connections
# Interface 0 incoming syn-flood protection
iptables -N syn_flood
iptables -A INPUT -p tcp --syn -j syn_flood
iptables -A syn_flood -m limit --limit 1/s --limit-burst 3 -j RETURN
iptables -A syn_flood -j DROP

#Limiting the incoming icmp ping request:
iptables -A INPUT -p icmp -m limit --limit 1/s --limit-burst 1 -j ACCEPT
iptables -A INPUT -p icmp -m limit --limit 1/s --limit-burst 1 -j LOG --log-prefix PING-DROP:
iptables -A INPUT -p icmp -j DROP
iptables -A OUTPUT -p icmp -j ACCEPT
````
