# dhcp/dns

Distribution IP, route, dns aux sous réseaux locaux

## docs généralistes

https://www.linuxtricks.fr/wiki/dnsmasq-le-serveur-dns-et-dhcp-facile-sous-linux

https://doc.ubuntu-fr.org/configuration_serveur_dns_dhcp

## Installation

```
$ apt install dnsmasq
```

## Configuration

```
$ vi /etc/dnsmasq.d/lan
---------------
no-resolv
interface=eth0
dhcp-range=192.168.200.10,192.168.200.200,12h
server=8.8.8.8
server=8.8.4.4
addn-hosts=/etc/dnsmasq-lan-host.conf
localise-queries
local=/lan/
domain=local.lan
expand-hosts
no-negcache
---------------
$ vi /etc/dnsmasq.d/wifi
---------------
no-resolv
interface=wlan0
dhcp-range=192.168.100.10,192.168.100.200,12h
server=8.8.8.8
server=8.8.4.4
addn-hosts=/etc/dnsmasq-wifi-host.conf
localise-queries
local=/wifi/
domain=local.wifi
expand-hosts
no-negcache
---------------

$ vi  /etc/dnsmasq-wifi-host.conf
192.168.100.1  proxy www serv

$ vi  /etc/dnsmasq-lan-host.conf
192.168.200.1  proxy ldap nfs www serv

```

### modifier resolv conf

```
$ vi /etc/systemd/resolved.conf
---------------
[Resolve]
DNSStubListener=no
---------------
$ systemctl restart systemd-resolved
```

## Démarrage auto

```
$ vi /lib/systemd/system/dnsmasq.service
# à ajouter pour démarrer seulement sur réseau actif
[Unit]
...
Requires=network-online.target
After=network.target network-online.target
StartLimitBurst=5
StartLimitIntervalSec=50
...

$ systemctl daemon-reload

$ systemctl start dnsmasq
```

# Bugs

* Parfois dnsmasq ne démarre pas au boot

**Solution:**

démarrage par crontab au boot

```
$ crontab -e 
@reboot sudo service dnsmasq start
```

* Ne démarre pas si eth0 ou wlan0 down (eth0 peut être down si aucune machine connectée !!!)

**Solution:**

```
$ apt-get install netplug

$ vi /etc/netplug/netplugd.conf 
eth0

$ mv /etc/netplug/netplug /etc/netplug/netplug.orig

$ vi /etc/netplug/netplug
---
#!/bin/sh
PATH=/usr/bin:/bin:/usr/sbin:/sbin
export PATH

dev="$1"
action="$2"
file="/etc/dnsmasq.d/lan"


case "$action" in
in)
    #echo "$dev : $action : plugged in" >> /tmp/netplug.log
    grep \#interface $file
    if [ $? -eq 0 ]; then
           sed -i s/^\#interface/interface/ $file
           systemctl restart dnsmasq
    fi
    ;;
out)
    #echo "$dev : $action : unplugged" >> /tmp/netplug.log
    grep -v \#interface $file
    if [ $? -eq 0 ]; then
           sed -i s/^interface/\#interface/ $file
           systemctl restart dnsmasq
    fi
    ;;
probe)
    #echo "$dev : $action : probed" >> /tmp/netplug.log
    ;;
*)
    echo "$dev : $action : I feel violated" >> /tmp/netplug.log
    exit 1
    ;;
esac
---


$ chmod +x /etc/netplug/netplug

```

### listes des IP attribuées
```
$ cat /var/lib/misc/dnsmasq.leases
```

#### configuration manuelle pour test avant usage de netplan

_pour information_

```
$ ifconfig wlp3s0b1 192.168.100.1 netmask 255.255.255.0 up
$ # route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.100.1
```
