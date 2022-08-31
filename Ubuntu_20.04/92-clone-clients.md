Clonage des clients
================

Si le réseau ne permet pas de télécharger les paquets, une machine master peut être clonée par plusieurs clients avec [Clonezilla](https://clonezilla.org/)

## Installation serveur

```
$ wget "https://osdn.net/frs/redir.php?m=nchc&f=clonezilla%2F77477%2Fclonezilla-live-3.0.1-8-amd64.iso" -O clonezilla-live-3.0.1-8-amd64.iso

$ mkdir /var/tftpboot/_iso/clonezilla
$ mount -o loop clonezilla-live-3.0.1-8-amd64.iso /mnt/
$ cp /mnt/live/{initrd.img,vmlinuz,filesystem.squashfs} /var/tftpboot/_iso/clonezilla
$ umount /mnt 
```

### Ajouter une option de boot à syslinux
```
$ vi /var/tftpboot/pxelinux.cfg/default
...
LABEL clone
  MENU LABEL ^Clone Disk
  KERNEL _iso/clonezilla/vmlinuz
  INITRD _iso/clonezilla/initrd.img
  APPEND modprobe.blacklist=floppy boot=live username=user union=overlay config components noswap edd=on nodmraid locales=fr_FR.UTF-8 keyboard-layouts=fr ocs_live_run="ocs-live-general" ocs_live_extra_param="" ocs_live_batch=no net.ifnames=0 nosplash noprompt ocs_postrun="hostname localhost" -p true fetch=tftp://192.168.200.1/_iso/clonezilla/filesystem.squashfs fsck.mode=skip
```

### Création d'un dossier nfs sur le serveur
pour enregistrer les informations du disque à cloner

```
$ /home/remote/partimag
```

## Préparation du master

nettoyage des caches de packages
```
$ apt-get clean
```
le nom de machine ``localhost`` sera réécrit au démarrage de chaque nouvelle machine
```
$ hostname localhost
```
installation de **dvdcss**
```
$ dpkg-reconfigure --terse libdvd-pkg
```

## Usage : clone d'un master vers de multiples machines

_Le mode réseau multicast permet de diffuser le master sur de multiples clients_

1. Démarrage pxe de tous les clients
  - attendre sur le menu "Start Clonezilla"


2. Démarrage pxe du master
  - suivre les menus
```
 Start Clonezilla > lite-server > netboot > auto-detect > 
 avec copie nfs (v3) sur 192.168.200.1 et le dossier ``/home/remote/partimag``
 > Begginer > from device > disque > sda > nofsck > -p true
```
 - copie pendant une 10aine de minutes des informations du disque sur ``/home/remote/partimag``

 - "Entrée" jusqu'au message "En attente de la connexion des clients"
noter l'IP du master


3. Clients
  - suivre les menus
```
Start Clonezilla > lite-client > IP du master
```

4. Nettoyage

Lorsque toutes les machines sont installées le dossier ``/home/remote/partimag`` peut être vidé
```
$ rm -rf /home/remote/partimag/*
```

## Usage : image d'un master sur le serveur

_à utiliser pour un très petit nombre de machines, pour éviter la saturation des ressources serveur_

- Pré requis : Exclure l'image de 4.6G des sauvegardes :
```
$ vi ~/.autorestic.yml
global:
  forget:
    keep-daily: 10
    keep-weekly: 52
  options:
    backup:
      exclude:
      - '*.nope'
      - '*.swp'
      - '*.iso'
      - '/home/remote/partimag/*
```

- **master:** Choisir ``device-image`` de Clonezilla pour enregistrer (``savedisk``) l'image en nfs sur le serveur dans le dossier ``/home/remote/partimag/``
- **client:** Choisir ``device-image`` puis ``restoredisk``
