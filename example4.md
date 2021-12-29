# This is an Example of setting iptables on the Server below.
````
         Internet
             |
       +----------+
-------|  Router  | ------------------------------------------------------------------
       +----------+       |                                 +----------+
             |            |                        10.0.0.31|          |
    LAN(192.168.0.0/24)   |   LAN(10.0.0.0/24)    +---------| Backend1 |
             |            |                       |         |          |  +----------+
             |      +----------+                  |         +----------+  |          |
--------------------|  Router  |-------------+----|-----------------------|    PC    |
         192.168.0.2+----------+10.0.0.1     |    |         +----------+  |          |
             |            |                  |    |         |          |  +----------+
             |      +----------+             |    +---------| Backend2 |
             |  eth0|          |eth1         |     10.0.0.32|          |
             +------|  Server  |-------------+              +----------+
        192.168.0.80|          |10.0.0.80
                    +----------+
````

* DROP INPUT by Default
* ACCEPT OUTPUT by Default
* DROP FORWARD by Default
* ACCEPT Established Connection
* ACCEPT the Connection from loopback
* Forward the Packets to 22 on eth0 to the same port on Backend1
* Forward the Packets to 80 on eth0 to the same port on Backend2
* ACCEPT Ping Connection for 5 times per a minites from internal network(10.0.0.0/24)
* ACCEPT SSH Connection from internal network(10.0.0.0/24)
* ACCEPT Outgoing Packets through the Server from internal network(10.0.0.0/24) and translatte the source address

nano iptables.sh

````
#!/bin/bash

trust_host='10.0.0.0/24'
my_internal_ip='10.0.0.80'
my_external_ip='192.168.0.80'

listen_port_1='22'
backend_host_1='10.0.0.31'
backend_port_1='22'

listen_port_2='80'
backend_host_2='10.0.0.32'
backend_port_2='80'

echo 1 > /proc/sys/net/ipv4/ip_forward

/sbin/iptables -F
/sbin/iptables -t nat -F
/sbin/iptables -X

/sbin/iptables -P INPUT DROP
/sbin/iptables -P OUTPUT ACCEPT
/sbin/iptables -P FORWARD DROP

/sbin/iptables -A FORWARD -i eth1 -o eth0 -s $trust_host -j ACCEPT
/sbin/iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

/sbin/iptables -A FORWARD -p tcp --dst $backend_host_1 --dport $backend_port_1 -j ACCEPT
/sbin/iptables -A FORWARD -p tcp --dst $backend_host_2 --dport $backend_port_2 -j ACCEPT

/sbin/iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
/sbin/iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
/sbin/iptables -A INPUT -p icmp --icmp-type echo-request -s $trust_host \
-d $my_internal_ip -m limit --limit 1/m --limit-burst 5 -j ACCEPT
/sbin/iptables -A INPUT -p tcp -m state --state NEW -m tcp -s $trust_host \
-d $my_internal_ip --dport 22 -j ACCEPT

/sbin/iptables -t nat -A POSTROUTING -o eth0 -s $trust_host -j MASQUERADE

/sbin/iptables -t nat -A PREROUTING -p tcp --dst $my_external_ip --dport $listen_port_1 \
-j DNAT --to-destination $backend_host_1:$backend_port_1
/sbin/iptables -t nat -A POSTROUTING -p tcp --dst $backend_host_1 --dport $backend_port_1 \
-j SNAT --to-source $my_internal_ip

/sbin/iptables -t nat -A PREROUTING -p tcp --dst $my_external_ip --dport $listen_port_2 \
-j DNAT --to-destination $backend_host_2:$backend_port_2
/sbin/iptables -t nat -A POSTROUTING -p tcp --dst $backend_host_2 --dport $backend_port_2 \
-j SNAT --to-source $my_internal_ip
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
