# Network

## identifier les interfaces

```
$ lshw -c network  | grep logical
```

en particulier les noms d'interfaces USB/ethernet

## netplan

Fixe les IPs et les noms des interfaces

* `enp2s0` : interface externe
* `wlan0` : interface wifi
* `eth0` : interface réseau interne ex: USB/ethernet

```
$ vi /etc/netplan/00-installer-config.yaml
---------------
network:
  ethernets:
    enp2s0:
      dhcp4: true
    wlp3s0b1:
      match:
        macaddress: 0c:54:a5:4b:xx:zz
      set-name: wlan0
      dhcp4: false
      addresses:  [192.168.100.1/24]
    enx00e04c680025:
      match:
        macaddress: 00:e0:4c:68:yy:ww
      set-name: eth0
      dhcp4: false
      dhcp6: false
      addresses: [192.168.200.1/24]
      optional: true
  version: 2
---------------

$ netplan generate
$ netplan apply

```

## vérifier

```
$ ip a
```