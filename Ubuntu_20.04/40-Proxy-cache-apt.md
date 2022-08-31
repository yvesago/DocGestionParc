# proxy cache apt

### docs

https://wiki.debian-fr.xyz/Apt-cacher-ng

## Installation

```
$ apt install apt-cacher-ng 
# tunnels HTTP : No

```

## Configuration

```
$ vi /etc/apt-cacher-ng/acng.conf

...
Port:9999

...
# pas de cache sur https
PassThroughPattern: ^(.*):443$


$ systemctl restart apt-cacher-ng
```

## test

http://[ip-public]:9999/acng-report.html

ou

http://proxy.local.lan:9999/acng-report.html

### mot de passe de gestion

```
$ vi /etc/apt-cacher-ng/security.conf
AdminAuth: master:xxxSECxxx


$ systemctl restart apt-cacher-ng
```

## Clients

détection de proxy et usage si présent

```
$ vi /etc/apt/detect_proxy.sh
---
#!/bin/bash
try_proxies=(
proxy.local.lan:9999
proxy:9999
)
for proxy in "${try_proxies[@]}"; do
    if nc -z ${proxy/:/ }; then
        proxy=http://$proxy/
        echo "$proxy"
        exit
    fi
done
echo DIRECT
---

$ chmod +x /etc/apt/detect_proxy.sh

$ vi /etc/apt/apt.conf.d/01acng
Acquire::http::Proxy-Auto-Detect "/etc/apt/detect_proxy.sh";

```

## Nettoyage quotidien du cache par

`/etc/cron.daily/apt-cacher-ng`

## Contrôle

```
$ systemctl status apt-cacher-ng

$ du -csh /var/cache/apt-cacher-ng/*
4,2M	/var/cache/apt-cacher-ng/security.ubuntu.com
54M	/var/cache/apt-cacher-ng/uburep
20K	/var/cache/apt-cacher-ng/_xstore
58M	total
```
