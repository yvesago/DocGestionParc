# Installation serveur

1. Page de download: https://ubuntu.com/download/server
   * choisir «Get Ubuntu Server 20.04 LTS»
2. téléchargement iso : ubuntu server 20.04 
   * _Pour créer une clé usb bootable :_ `usb-creator-gtk`
3. Accès BIOS : (5x F1 pour Lenovo)
   * choisir USB comme première solution de boot
4. Choix d'installation :
   * utiliser le nouveau installeur, sinon il aura des problèmes de clavier
   * stockage lvm
     * laisser 100G pour le système principal
     * création ``homeremote-lv`` sur les xxxG restants, point de montage ``/home/remote``
   * (si un proxy est déjà disponible) : ``http://proxy.local.lan:9999``
   * installer serveur ssh
   * pas de paquet spécifique
5. Définition identifiant / mot de passe:
   * ex: master / ``***SEC***``
6. Premières connexions:
   * `server$ hostname -I` : IP locale du server
   * `distant$ ssh master@ip` : connexion depuis une machine distante

## Mises à jours automatiques

La première opération est la configuration les mises à jours automatiques

```
$ vi /etc/apt/apt.conf.d/10periodic
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
```

```
$ vi /etc/apt/apt.conf.d/50unattended-upgrades

Unattended-Upgrade::Remove-Unused-Kernel-Packages "true";

Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

Unattended-Upgrade::Automatic-Reboot "true";

Unattended-Upgrade::Automatic-Reboot-Time "02:00";

```

### traces de vérification
logs dans `/var/log/unattended-upgrades/`


## Éléments manquants dans l'installation server par défaut

* définition time zone

```
$ timedatectl list-timezones
$ timedatectl set-timezone Europe/Paris
```

* synchro timezone

```
$ apt install ntpdate
```

## Problèmes éventuels

```
# contrôle des échecs de démarrage
$ systemctl -t service

# ajouter  --timeout=10
$ vi /lib/systemd/system/systemd-networkd-wait-online.service
ExecStart=/usr/lib/systemd/systemd-networkd-wait-online --timeout=10
```
