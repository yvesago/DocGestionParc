# Installation des clients


## Distribution des images

Ubuntu 22.04

```
$ mkdir -p /var/www/images
$ cd /var/www/images
$ wget http://releases.ubuntu.com/jammy/ubuntu-22.04.3-desktop-amd64.iso

$ mkdir -p /var/tftpboot/_iso/ubuntu_22.04/Desktop/
$ mount /var/www/images/ubuntu-22.04.3-desktop-amd64.iso /mnt/

# copie du noyau vmlinuz et du mini système de démarrage initrd 
$ cp /mnt/casper/{vmlinuz,initrd} /var/tftpboot/_iso/ubuntu_22.04/

# copie de la clé sur disque
$ cp -a /mnt/. /var/tftpboot/_iso/ubuntu_22.04/Desktop/

```

L'image de Ubuntu Desktop, depuis 20.04, a besoin de 8G de mémoire pour être installable depuis la mémoire.

pour des machines avec moins de RAM, l'astuce est de démarrer avec l'installation serveur
```
$ cd /var/www/images
$ wget https://releases.ubuntu.com/22.04/ubuntu-22.04.3-live-server-amd64.iso
$ umount /mnt
$ mount /var/www/images/ubuntu-22.04.3-live-server-amd64.iso /mnt/

$ cp /mnt/casper/vmlinuz /var/tftpboot/_iso/ubuntu_22.04/vmlinuz-serv
$ cp /mnt/casper/initrd /var/tftpboot/_iso/ubuntu_22.04/initrd-serv

```

PrimTux 7
```
$ mkdir -p /var/tftpboot/_iso/primtux_20.04/Desktop/
$ cd /var/tftpboot/_iso/primtux_20.04/
$ wget https://sourceforge.net/projects/primtux/files/Distribution/PrimTux7-Ubuntu-20.04.3-amd64.iso/download -O PrimTux7-Ubuntu-20.04.3-amd64.iso

$ umount /mnt
$ mount /var/www/images/PrimTux7-Ubuntu-20.04.3-amd64.iso /mnt/
# copie de la clé sur disque
$ cp -a /mnt/. /var/tftpboot/_iso/primtux_20.04/Desktop/

```

PrimTux est basé sur Ubuntu 20.04 mais n'a pas prévu une installation réseau.

Il faut légèrement modifier l'image

```
# affichage au démarrage
$ vi /var/tftpboot/_iso/primtux_20.04/Desktop/.disk/info
# remplacer "Ubuntu" par "PrimTux 7"

# fix kernel access
chmod 644 /var/tftpboot/_iso/primtux_20.04/Desktop/casper/vmlinuz
# fix casper uuid
cp /var/tftpboot/_iso/ubuntu_20.04/Desktop/.disk/casper-uuid-generic /var/tftpboot/_iso/primtux_20.04/Desktop/.disk/


# configuration par defaut
$ vi /var/tftpboot/_iso/primtux_20.04/Desktop/preseed/ubuntu.seed
# changement de mot de passe
# ajout du proxy cache de mises à jours
d-i     mirror/http/proxy string http://proxy.local.lan:9999/
```


## Boot réseau

### Installation PXE
```
$ apt install syslinux-common
$ apt install pxelinux
$ cd /var/tftpboot
$ cp /usr/lib/PXELINUX/pxelinux.0 .
$ cp /usr/lib/syslinux/memdisk .
$ cp /usr/lib/syslinux/modules/bios/* .
```

### PXE et téléchargement tftp sur le réseau local
```
$ vi /etc/dnsmasq.d/lan

# ajout tftp et pxe
## Telechargement simple avec tftp
enable-tftp
tftp-root=/var/tftpboot
## On initialise le service PXE
dhcp-boot=pxelinux.0
## On initialise le menu du service PXE
pxe-prompt="Veuillez faire votre choix :"
pxe-service=x86PC, "Boot depuis le disque local", 30
pxe-service=x86PC, "Interface PXE", pxelinux

$ systemctl restart dnsmasq
```

### Configuration choix PXE

Le menu PXE propose d'installer Ubuntu ou PrimTux

Les 2 distributions peuvent aussi être testées en **live** : rien n'est installé sur le disque du client. 

```
$ mkdir /var/tftpboot/pxelinux.cfg
$ vi /var/tftpboot/pxelinux.cfg/default
DEFAULT menu.c32
MENU MARGIN 0
MENU ROWS -9
MENU TABMSG
MENU TABMSGROW -3
MENU CMDLINEROW -3
MENU HELPMSGROW -4
MENU HELPMSGENDROW -1
MENU COLOR SCREEN 30;47
MENU COLOR BORDER 30;47
MENU COLOR TITLE 30;47
MENU COLOR SCROLLBAR 30;47
MENU COLOR SEL 37;40
MENU COLOR UNSEL 30;47
MENU COLOR CMDMARK 30;47
MENU COLOR CMDLINE 30;47
MENU COLOR TABMSG 37;40
MENU COLOR DISABLED 37;40
MENU COLOR HELP 37;40
MENU TITLE Serveur d'installation PXE
LABEL hdt
 MENU LABEL ^Hardware Detection Tool
 KERNEL hdt.c32
LABEL reboot
 MENU DEFAULT
 MENU LABEL Reboot
 COM32 reboot.c32
LABEL ubuntu_nfs
  MENU LABEL Ubuntu 22.04 ^Live (NFS)
  KERNEL _iso/ubuntu_22.04/vmlinuz
  INITRD _iso/ubuntu_22.04/initrd
  APPEND modprobe.blacklist=floppy root=/dev/ram0 ramdisk_size=1500000 ip=dhcp locale=fr_FR bootkbd=fr console-setup/layoutcode=fr console-setup/variantcode=nodeadkeys nfsroot=192.168.200.1:/var/tftpboot/_iso/ubuntu_22.04/Desktop netboot=nfs ro boot=casper splash systemd.mask=tmp.mount fsck.mode=skip --
LABEL ubuntu_inst
  MENU LABEL Ubuntu 22.04 ^Install (HTTP)
  KERNEL _iso/ubuntu_22.04/vmlinuz-serv
  INITRD _iso/ubuntu_22.04/initrd-serv
  APPEND modprobe.blacklist=floppy root=/dev/ram0 ramdisk_size=1500000 ip=dhcp autoinstall url=http://www.local.lan/images/ubuntu-22.04.3-live-server-amd64.iso  ds=nocloud-net;s=http://www.local.lan/images/ubuntu_22.04/ cloud-config-url=/dev/null
LABEL primtux_nfs
  MENU LABEL ^PrimTux7 20.04 Live (NFS)
  KERNEL _iso/ubuntu_20.04/vmlinuz
  INITRD _iso/ubuntu_20.04/initrd
  APPEND modprobe.blacklist=floppy root=/dev/ram0 ramdisk_size=1500000 ip=dhcp netboot=nfs boot=casper nfsroot=192.168.200.1:/var/tftpboot/_iso/primtux_20.04/Desktop splash fsck.mode=skip
LABEL primtux_inst
  MENU LABEL Prim^Tux7 Install 20.04 (NFS)
  KERNEL _iso/ubuntu_20.04/vmlinuz
  INITRD _iso/ubuntu_20.04/initrd
  APPEND modprobe.blacklist=floppy root=/dev/ram0 ramdisk_size=1500000 ip=dhcp netboot=nfs boot=casper nfsroot=192.168.200.1:/var/tftpboot/_iso/primtux_20.04/Desktop file=/cdrom/preseed/ubuntu.seed fsck.mode=skip only-ubiquity



```
## Configuration automatique

Ubuntu est installé depuis l'image serveur. La configuration est alors réalisée avec les outils de ``cloud-init`` : un simple fichier texte '``user-data``, distribué par http, permet d'ajouter et configurer les machines clientes 

```
$ mkdir /var/www/images/ubuntu_22.04/

$ touch /var/www/images/ubuntu_22.04/vendor-data
$ echo "instance-id: jammy-autoinstall" > /var/www/images/ubuntu_22.04/meta-data
$ vi /var/www/images/ubuntu_22.04/user-data
```

[user-data](user-data) est documenté.


L'installation démarre avec l'installation d'une version serveur élémentaire, la création de fichier de configuration et la création de 7 scripts d'installation de la version Desktop. Ces scripts sont lancés au cours du premier démarrage. Si le téléchargement de certains paquets échoue sur une liaison réseau à petit débit ou instable, il suffit de relancer les scripts en échec. Sur une petite liaison à 2Mb (3G ou 4G) 1,5Go de données (hors cache) sont téléchargées pendant un peu moins de 3H.


Test des erreurs de syntaxe
```
$ cloud-init schema --config-file  /var/www/images/ubuntu_22.04/user-data
Valid cloud-config: /var/www/images/ubuntu_22.04/user-data
```

## Primtux

L'installation de Primtux est en mode manuel : on peut choisir de l'installer sur tout le disque ou sur l'espace libre ``/dev/sda3``

Si la double installation est choisi. On peut ensuite choisir le système de démarrage par défaut :

```
$ vi /etc/default/grub

GRUB_DEFAULT=4
GRUB_DISTRIBUTOR="Primtux"

$ update-grub
```
