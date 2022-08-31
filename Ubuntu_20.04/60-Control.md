# Control

script `control.sh` pour vérifier que les services essentiels sont actifs

```
$ vi /usr/local/bin/control.sh
$ chmod +x /usr/local/bin/control.sh

----
#!/bin/bash

if [[ -v TERM ]]; then
  NOK='\033[0;31m' #RED
  OK='\033[0;32m' #GREEN
  B='\033[1m\033[4m' #BOLD Underlined
  NC='\033[0m'
else
  NOK=''
  OK=''
  B='* '
  NC=''
fi

echo -e "${B}Control network interfaces:${NC}"
for value in eno1 wlan0 eth0
 do
  ip a | grep "$value:.* UP" > /dev/null
  if [ $? -eq 0 ]; then
    echo -e "$value ${OK}is up${NC}"
  else
    echo -e "$value ${NOK}is down${NC}"
  fi
done
echo

echo -e "${B}Control daemons:${NC}"
for value in sshd freeradius hostapd glauth dnsmasq rtun apt-cacher-ng glauth-ui caddy rpcbind rpc.mountd rpc.idmapd
 do
  ps cax | grep $value > /dev/null
  if [ $? -eq 0 ]; then
    echo -e "$value ${OK}is running${NC}"
  else
    echo -e "$value ${NOK}is not running${NC}"
  fi
done
echo

echo -e "${B}Control iptables:${NC}"
iptables -L -t nat | grep MASQUERADE > /dev/null
if [ $? -eq 0 ]; then
  echo -e "NAT ${OK}ok${NC}"
else
  echo -e "NAT ${NOK}not ok${NC}"
fi
echo

echo -e "${B}IPs:${NC}"
hostname -I
echo


echo -e "${B}Control disks:${NC}"
for value in /dev/mapper/ubuntu--vg-ubuntu--lv /dev/mapper/ubuntu--vg-homeremote--lv /dev/sda2
do
  df -h | grep $value  
done
```

---

## motd

Affichage au login

```
$ vi /etc/update-motd.d/51-control

#!/bin/sh

# if the current release is under development there won't be a new one
if [ "$(lsb_release -sd | cut -d' ' -f4)" = "(development" ]; then
    exit 0
fi

# if it is non-root user, skip
if [ $(id -u) -ne 0 ]; then
    exit 0
fi

if [ -x /usr/local/bin/control.sh ]; then
    exec /usr/local/bin/control.sh
fi

$ chmod +x /etc/update-motd.d/51-control
```
