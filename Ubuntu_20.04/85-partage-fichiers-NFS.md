Partage de fichiers - NFS
==========================



```
$ apt install nfs-kernel-server
```

Autorisations d'accès par réseau
```
$ vi /etc/hosts.allow
rpcbind: 192.168.200. 127.0.0.1
```
_pour un meilleur niveau de sécurité, il faudra utiliser NFS v4 et activer kerberos_

## Lecture simple

Dossiers partagés
```
$ vi /etc/exports
/var/tftpboot/_iso/ 192.168.200.0/24(ro,sync,no_subtree_check,root_squash)
/home/share/ 192.168.200.0/24(ro,no_subtree_check,no_root_squash)

$ exportfs -a
```

test
```
$ systemctl status rpcbind
...

$ systemctl status nfs-mountd
...

$ systemctl status nfs-idmapd
...

$ showmount -e 127.0.0.1
Export list for 127.0.0.1:
/var/tftpboot/_iso  192.168.200.0/24
/home/share         192.168.200.0/24
```

pb connu : si ``rpc.mountd`` et ``nfs.idmapd`` ne démarrent pas au boot
```
$ apt-get install --reinstall nfs-kernel-server nfs-common
```

### Clients

Pour information : la configuration des clients est automatisée à l'installation


```
$ apt install nfs-common
$ showmount -e 192.168.200.1
Export list for 127.0.0.1:
/var/tftpboot/_iso  192.168.200.0/24
/home/share         192.168.200.0/24
```

**Montage manuel**
```
$ mount -t nfs 192.168.200.1:/home/share /mnt/nfs/share
```

**Automount pour tout utilisateur**

L'accès au dossier /mnt/nfs/share réalise le montage

_le nom du fichier doit correspondre avec le point de montage_
```
$ vi /etc/systemd/system/mnt-nfs-nfsshare.mount
    [Unit]
    Description = nfs mount for nfsfiles

    [Mount]
    What=nfs:/home/share
    Where=/mnt/nfs/nfsshare
    Type=nfs
    Options=defaults
    TimeoutSec=5

    [Install]
    WantedBy=multi-user.target


$ vi /etc/systemd/system/mnt-nfs-nfsshare.automount
    [Unit]
    Description=nfs automount for nfsfiles

    [Automount]
    Where=/mnt/nfs/nfsshare
    TimeoutIdleSec=0

    [Install]
    WantedBy=multi-user.target

$ systemctl enable mnt-nfs-nfsshare.mount
$ systemctl enable mnt-nfs-nfsshare.automount
$ systemctl start mnt-nfs-nfsshare.mount
$ systemctl start mnt-nfs-nfsshare.automount
```


## Dossier partagé en écriture pour tous les utilisateurs

sur le serveur
```
$ vi /etc/exports
# ajout
/home/remote/ 192.168.200.0/24(rw,sync,nohide,no_subtree_check,no_root_squash)

$ exportfs -a
```

Sur le client
```
$ mkdir -p /mnt/nfs/remote
$ chmod 1777 /mnt/nfs/remote 
```

_il faut créer les scripts de démarrage mnt-nfs-remote.mount et mnt-nfs-remote.automount_

Tous les utilisateurs peuvent écrire dans /mnt/nfs/remote

L'accès aux fichiers est défini par le mask ou individuellement par chaque utilisateur.

**Par défaut tous les utilisateurs peuvent lire tous les fichiers.**

Il est recommandé de conserver les secrets avec ``keepassxc`` pour les mots de passe ou ``zulucrypt`` pour des dossiers chiffrés


