# Gestion web

Utilisation de `glauth` comme serveur ldap léger. Les comptes sont conservés dans un fichier à plat.

La gestion avec interface web est réalisée avec <https://github.com/yvesago/glauth-ui-light>

## Installation

### Téléchargement

<https://github.com/yvesago/glauth-ui-light/releases>

```
$ wget https://github.com/yvesago/glauth-ui-light/releases/download/v1.4.3/glauth-ui-light_1.4.3-0.static0_amd64.deb

$ dpkg -i glauth-ui-light_1.4.3-0.static0_amd64.deb
```


### Configuration

```
# vi /etc/glauth-ui/glauth-ui.cfg
appname = "glauth-ui-light"
appdesc = "Gestion des utilisateurs et groupes"
dbfile = "/etc/glauth/sample-simple.cfg"
port = "0.0.0.0:8080"

# maskotp = true


# authentification utilisateurs des clients
defaulthomedir = "/home/remote"
defaultloginshell = "/bin/bash"

[sec]
  # random secrets for CSRF token
  csrfrandom = "lkg367khsqda" ***SEC***
  trustedproxies = ["127.0.0.1","::1"]

[ssl]
  crt = "/etc/ssl/certs/server.local.lan.crt"
  key = "/etc/ssl/private/server.local.lan.key"

[logs]
  path = "/var/log/glauth-ui/"


[locale]
  lang = "fr"
  path = "/etc/glauth-ui/locales/"
  langs = ["en-US","fr-FR"]

[passpolicy]
  min = 2
  max = 24
  allowreadssha256 = true

[cfgusers]
  start = 5000
  gidadmin = 5501
  gidcanchgpass = 5503 # le groupe users peut changer son mot de passe

[cfggroups]
  start = 5500
```

## Vérification

firefox https://serv1:8080

par défaut ``johndoe/dogood`` est administrateur

supprimer les comptes de démo, renommer ``johndoe`` et changer les mots de passe

```
$ systemctl status glauth-ui-light
```

## Problèmes connus

**ATTENTION** le compte ``serviceuser`` va être utilisé par les services d'authentification, il doit conserver le bloc ``[[users.capabilities]]``
```
[[users]]
  name = "serviceuser"
  passsha256 = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  primarygroup = 5502
  uidnumber = 5003
  loginShell = "/bin/false"
  [[users.capabilities]]
    action = "search"
    object = "*"
```
Les modifications du comptes ``serviceuser`` peuvent effacer ce bloc. Il faut alors l'ajouter manuellement dans ``/etc/glauth/sample-simple.cfg``

Un changement de mot de passe de ``serviceuser`` doit être répercuté sur les services d'authentification ``radius`` et sur les services ``sssd`` de **tous** les clients.

## Logs

```
$ ll -rt logs/
$ tail -f logs/app.20aammdd
```

## Compilation sur autre machine

_Pour information, par curiosité_


```

$ git clone https://github.com/yvesago/glauth-ui-light.git
$ cd glauth-ui-light
$ make 


# déploiement
$ scp build/linux/glauth-ui master@serv1:/home/app/glauth-ui
$ scp -pr locales master@serv1:/home/app/glauth-ui/locales
```
#### Démarrage automatique

```
$ vi /etc/systemd/system/glauthui.service

---------------
[Unit]
Description= Ldap Glauth UI
After=network.target

[Service]
Type=simple
#User=glauth
ExecStart=nohup /home/app/glauth-ui/glauth-ui -c /home/app/glauth-ui/conf/webconfig.cfg 

[Install]
WantedBy=multi-user.target
---------------

$ systemctl enable glauthui.service
$ systemctl start glauthui.service
```

