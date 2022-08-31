# Comptes utilisateurs

Utilisation de `glauth` comme serveur ldap léger. Les comptes sont conservés dans un simple fichier.

## Installation Debian

```
$ wget https://github.com/yvesago/glauth/releases/download/v2.1.0/glauth_2.1.0-0.static0_amd64.deb
$ dpkg -i glauth_2.1.0-0.static0_amd64.deb
```

## Configuration

```
$ vi /etc/glauth/sample-simple.cfg

#debug = true
syslog = true
...
watchconfig = true

[ldap]
  enabled = false
  ...

[ldaps]
  enabled = true
  listen = "0.0.0.0:3894"
  cert = "/etc/ssl/certs/server.local.lan.crt"
  key = "/etc/ssl/private/server.local.lan.key"

[backend]
  ...
  #nameformat = "cn"
  nameformat = "uid"

  ...
  # pour authentification des clients
  anonymousdse = true  


```

renommer
 - le groupe ``superheros`` par ``admins``
 - le groupe ``svcaccts`` par ``applications``
 - supprimer le groupe ``vpn``
 - ajouter un groupe ``users`` avec ``gidnumber = 5503``


autorisation d'accès à la clé de chiffrement SSL
```
$ usermod -a -G ssl-cert glauth


$ systemctl restart glauth
```

## Vérification
```
$ systemctl status glauth
● glauth.service - Ldap Glauth
     Loaded: loaded (/lib/systemd/system/glauth.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-08-01 12:55:00 GMT; 18s ago
...
```


## logs

```
$ grep glauth /var/log/syslog
```


## tests
```
$ vi /etc/ldap/ldap.conf 
TLS_CACERT      /etc/ssl/certs/server.local.lan.crt

$ openssl s_client localhost:3894

```


## test depuis machine distante
```
$ apt install ldap-utils

$ LDAPTLS_REQCERT=never ldapsearch -LL -H 'ldaps://192.168.xxx.yyy:3894/' -x -D uid=serviceuser,dc=glauth,dc=com -w mysecret -x -bdc=glauth,dc=com uid=serviceuser
```


# Installation binaire

_Pour information_

## Compilation sur autre machine

```

$ git clone https://github.com/glauth/glauth.git
$ cd glauth/v2
$ make fast

# déploiement
$ scp bin/linuxamd64/glauth master@192.168.xxx.yyy:/home/app/glauth
```

## démarrage automatique

```
$ vi /etc/systemd/system/glauth.service

---------------
[Unit]
Description= Ldap Glauth
After=network.target

[Service]
Type=simple
ExecStart=nohup /home/app/glauth/glauth -c /home/app/glauth/sample-simple.cfg

[Install]
WantedBy=multi-user.target
---------------

$ systemctl enable glauth.service
$ systemctl start glauth.service
```

# Construction package Debian

_Pour information_

```
cd v2
mkdir -p debian/source
```

**debian/compat**

```
echo 11 > debian/compat
```

**debian/control**

```
Source: glauth
Maintainer: YvesAgo <yves+github@yvesago.net>
Build-Depends: debhelper (>= 11)

Package: glauth
Architecture: amd64
Depends: adduser
Description: glauth v2 ldap server
 Go-lang LDAP Authentication (GLAuth) is a secure, easy-to-use, LDAP server
 https://github.com/glauth/glauth
```

**debian/changelog**

```
$ dch --create -v 2.1.0 --package glauth

glauth (2.1.0) UNRELEASED; urgency=medium

  * Initial release.

 -- yvesago <yves+githu\@yvesago.net>  Tue, 01 Mar 2022 21:53:26 +0100 

```

**debian/glauth.install**

```
debian/sample-simple.cfg etc/glauth

```

**debian/glauth.postinst**

```
#!/bin/sh

set -e

case "$1" in
    configure)
        adduser --system --disabled-password --disabled-login --home /var/empty \
                --no-create-home --quiet --force-badname --group glauth
	chmod 750 /etc/glauth
	chown glauth:glauth /etc/glauth
	chown glauth:glauth /etc/glauth/*
	chmod 640 /etc/glauth/*
        ;;
esac

#DEBHELPER#

exit 0
```

**debian/glauth.service**

```
[Unit]
Description=Ldap Glauth v2
After=network.target

[Service]
Type=simple
User=glauth
ExecStart=/usr/bin/nohup /usr/bin/glauth -c /etc/glauth/sample-simple.cfg

[Install]
WantedBy=multi-user.target
```

**debian/rules**

```
#!/usr/bin/make -f

DISTRIBUTION = "static"
VERSION = 2.1.0
PACKAGEVERSION = $(VERSION)-0~$(DISTRIBUTION)0

%:
	dh $@ --with quilt

override_dh_auto_build:
override_dh_auto_test:
override_dh_installdocs:
override_dh_auto_install:
	$(MAKE) install DESTDIR=debian/glauth

override_dh_installsystemd:
	dh_installsystemd --name=glauth

override_dh_gencontrol:
	dh_gencontrol -- -v$(PACKAGEVERSION)
```

**debian/patches/fix-makefile-glauth.patch**

```
$ dquilt new fix-makefile-glauth
* dquilt add Makefile
```

```
--- a/Makefile
+++ b/Makefile
@@ -1,3 +1,7 @@
+
+export GOROOT = /snap/go/current
+export GOBIN = /snap/go/current/bin/go
+
 VERSION=$(shell bin/linuxamd64/glauth --version)

 GIT_COMMIT=$(shell git rev-list -1 HEAD )
@@ -43,6 +47,8 @@
 # Setup commands to always run
 setup: getdeps format

+install: setup linuxamd64
+
 #####################
 # Subcommands
 #####################
@@ -53,13 +59,13 @@

 # Get all dependencies
 getdeps:
-       go get -d ./...
+       ${GOBIN} get -d ./...

 updatetest:
        ./scripts/ci/integration-test.sh

 format:
-       go fmt
+       ${GOBIN} fmt

 devrun:
        go run ${BUILD_FILES} -c sample-simple.cfg
@@ -68,7 +74,8 @@
        mkdir -p bin/$@ && GOOS=linux GOARCH=386 go build ${TRIM_FLAGS} -ldflags "${BUILD_VARS}" -o bin/$@/glauth ${BUILD_FILES} && cd bin/$@ && sha256sum glauth > glauth.sha256

```

```
$ dquilt refresh
```


**Build**

```
$ debuild -us -uc -b

# verif
$ dpkg-deb -c ../glauth_2.1.0-0~static0_amd64.deb

# install
$ sudo dpkg -i ../glauth_2.1.0-0~static0_amd64.deb
```
