# Accès à distance par reverse tunnel

# Installation

```
$ wget https://serveur-pub.net/deb/rtun_1.3.2-0~static0_amd64.deb
$ dpkg -i rtun_1.3.2-0~static0_amd64.deb
```

## Configuration client

```
$ vi /etc/rtun.yml
----
gateway_url: ws://serveur-pub.net:7600

# A key registered in the gateway server configuration file. ***SEC***
# create with openssl rand -hex 32
auth_key: xxxxxxxxxxecd33b7c078631d3424137ff332d7897ecd6e9ddee28df138a0064

forwards:
  # Forward 10122/tcp on the gateway server to localhost:22 (tcp)
  - port: 10122/tcp
    destination: 127.0.0.1:22
  # Forward 10180/tcp on the gateway server to localhost:80 (tcp)
  - port: 10180/tcp
    destination: 127.0.0.1:80
----
```

```
$ systemctl restart rtun
```


# Serveur

## Compilation

_Pour information_

```
$ git clone https://github.com/snsinfu/reverse-tunnel
$ cd reverse-tunnel

$ CGO_ENABLED=0 go build -o rtun github.com/snsinfu/reverse-tunnel/agent/cmd
$ CGO_ENABLED=0 go build -o rtun-server github.com/snsinfu/reverse-tunnel/server/cmd
```

`CGO_ENABLED=0` pour compilation statique

## Configuration serveur

ex: serveur-pub.net

```
$ vi /etc/rtun-server.yml
----
control_address: 0.0.0.0:7600

# List of authorized agents follows.
agents:
  - auth_key: zzzzzzzzzzzzzzzzzzzzz31d3424137ff332d7897ecd6e9ddee28df138a0064
    ports: [10022/tcp]
  - auth_key: xxxxxxxxxxecd33b7c078631d3424137ff332d7897ecd6e9ddee28df138a0064
    ports: [10122/tcp, 10180/tcp]

----
# chaine de 32  caractères aléatoires
$ openssl rand -hex 32
```

## Démarrage automatique

```
$ vi /lib/systemd/system/rtun.service
----
[Unit]
Description=Rtun tunneling agent
After=network.target

[Service]
Type=simple
ExecStart=/home/app/rtun/rtun -f /home/app/rtun/rtun.yml
Restart=always

[Install]
WantedBy=default.target
----

$ systemctl daemon-reload
$ systemctl start rtun

```

# Usage

```
$other# ssh master@serveur-pub.net -p 10122
```

## logs client

```
$ grep rtun /var/log/syslog

```

## Création de paquet debian/ubuntu

_Pour information, pour les curieux_

[doc de référence](https://vincent.bernat.ch/en/blog/2019-pragmatic-debian-packaging)

```
$ wget https://github.com/snsinfu/reverse-tunnel/archive/refs/tags/v1.3.2.tar.gz

$ mv v1.3.2.tar.gz rtun_1.3.2.orig.tar.gz  # _ et .orig. obligatoire
$ tar zxvf rtun_1.3.2.orig.tar.gz
$ mv reverse-tunnel-1.3.2/ rtun-1.3.2

$ cd rtun-1.3.2
```


**Création des fichiers de description:**

  - debian/compat
  - debian/changelog
  - debian/control



```
$ mkdir debian

$ dch --create -v 1.3.2 --package rtun # creation de changelog

$ echo 11 > debian/compat

$ vi debian/control
---
Source: rtun
Maintainer: Yves Agostini <yves@vya.me>
Build-Depends: debhelper (>= 11)

Package: rtun
Architecture: any
Description: remote access
 remote acces client for golang github.com/snsinfu/reverse-tunnel
---

```


**Modifications :**

Usage de ``dquilt`` pour patcher le code d'origine

Ajout à ``Makefile`` d'une cible target (le paquet debian) et compilation statique

[doc dquilt](https://www.debian.org/doc/manuals/maint-guide/modify.en.html)

```

$ dquilt new fix-makefile-rtun.patch 

$ dquilt add Makefile 

$ vi Makefile

export GOROOT = /snap/go/current 
export GOBIN = /snap/go/current/bin/go

... 

    install:
        CGO_ENABLED=0 ${GOBIN} build -o $(DESTDIR)/usr/bin/rtun ./agent/cmd
 
---

$ dquilt refresh

```

**debian/rules**
```
#!/usr/bin/make -f

DISTRIBUTION = "static"
VERSION = 1.3.2
PACKAGEVERSION = $(VERSION)-0~$(DISTRIBUTION)0

%:
        dh $@ --with quilt

override_dh_auto_build:
override_dh_auto_test:
override_dh_installdocs:
override_dh_auto_install:
        $(MAKE) install DESTDIR=debian/rtun

override_dh_installsystemd:
        dh_installsystemd --name=rtun

override_dh_gencontrol:
        dh_gencontrol -- -v$(PACKAGEVERSION)
```

**debian/rtun.install** fichiers à installer

_installer rtun.yml dans /etc_

```
debian/rtun.yml etc
```

**debian/source/include-binaries** inication du binaire
```
debian/usr/bin/rtun
```

**debian/rtun.yml**  fichier configuration

**debian/rtun.conffiles** protection lors des mises à jours
```
/etc/rtun.yml
```

**debian/rtun.service** démarrage
```
[Unit]
Description=Rtun tunneling agent
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/rtun -f /etc/rtun.yml
Restart=always

[Install]
WantedBy=default.target
```

**Construction**
```
$ debuild -us -uc -b

# verification du contenu

$ dpkg-deb -c ../rtun\_1.3.2\_amd64.deb

```
