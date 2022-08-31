iptables
=============

Gestion des flux réseau entre les interfaces

## activer le NAT
```
$ vi  /etc/sysctl.conf
 net.ipv4.ip_forward=1
$ sysctl -p
```

## règles (à compléter)
```  
# Flush
$ iptables -F
$ iptables -F -t nat

# Nat
$ iptables -t nat -A POSTROUTING -o eno1 -j MASQUERADE
```

## enregistrement des règles
```
$ iptables-save > /etc/iptables.rules
```

## contrôle

```
$ iptables -L -t nat
..
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  anywhere             anywhere
```


## appliquer les règles au démarrage
```
$ vi /etc/systemd/system/restore-iptables-rules.service
---------------
[Unit]
Description = Apply iptables rules
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'iptables-restore < /etc/iptables.rules'

[Install]
WantedBy=network-pre.target
---------------

$ systemctl enable restore-iptables-rules.service
```

