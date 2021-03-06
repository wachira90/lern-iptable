# This is an Example of setting iptables on the Server below.
````
         Internet
             |
       +----------+
-------|  Router  | -------------------------------------------------------------
       +----------+                |
             |                     |
    LAN(192.168.0.0/24)            |              LAN(10.0.0.0/24)
             |               +----------+                            +----------+
             |  192.168.0.100|          |10.0.0.1                    |          |
-----------------------------|  Server  |----------------------------|    PC    |
                         eth0|          |eth1                        |          |
                             +----------+                            +----------+
````

	
* DROP INPUT by Default
* ACCEPT OUTPUT by Default
* DROP FORWARD by Default
* ACCEPT Established Connection
* ACCEPT the Connection from loopback
* ACCEPT Ping Connection for 5 times per a minites from internal network(10.0.0.0/24)
* ACCEPT SSH Connection from internal network(10.0.0.0/24)
* ACCEPT Outgoing Packets through the Server from internal network(10.0.0.0/24) and translatte the source address

nano iptables.sh
````
#!/bin/bash

trust_host='10.0.0.0/24'
my_host='10.0.0.100'

echo 1 > /proc/sys/net/ipv4/ip_forward

/sbin/iptables -F
/sbin/iptables -t nat -F
/sbin/iptables -X

/sbin/iptables -P INPUT DROP
/sbin/iptables -P OUTPUT ACCEPT
/sbin/iptables -P FORWARD DROP

/sbin/iptables -A FORWARD -i eth1 -o eth0 -s $trust_host -j ACCEPT
/sbin/iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

/sbin/iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

/sbin/iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT

/sbin/iptables -A INPUT -p icmp --icmp-type echo-request -s $trust_host \
-d $my_host -m limit --limit 1/m --limit-burst 5 -j ACCEPT

/sbin/iptables -A INPUT -p tcp -m state --state NEW -m tcp -s $trust_host \
-d $my_host --dport 22 -j ACCEPT

/sbin/iptables -t nat -A POSTROUTING -o eth0 -s $trust_host -j MASQUERADE
````
/etc/rc.d/init.d/iptables save

/etc/rc.d/init.d/iptables restart
````
[root@dlp ~]# sh iptables.sh
iptables: Saving firewall rules to /etc/sysconfig/iptables: [  OK  ]
iptables: Flushing firewall rules: [  OK  ]
iptables: Setting chains to policy ACCEPT: filter [  OK  ]
iptables: Unloading modules: [  OK  ]
iptables: Applying firewall rules: ip_tables: (C) 2000-2006 Netfilter Core Team
nf_conntrack version 0.5.0 (16384 buckets, 65536 max)
[  OK  ]
````
