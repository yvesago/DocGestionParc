Wikipedia
=========

### Téléchargement du serveur:
```
$ wget https://download.kiwix.org/release/kiwix-tools/kiwix-tools_linux-x86_64.tar.gz
$ tar zxvf kiwix-tools_linux-x86_64.tar.gz
$ ln -s kiwix-tools_linux-x86_64-3.5.0-2 kiwix

```

création d'un utilisateur ``kiwix``

```
$ mkdir /home/remote/wikipedia
$ useradd -d /home/remote/wikipedia -Mrs /sbin/nologin kiwix
```

### Téléchargement des données

création d'un dossier ``/home/remote/wikipedia`` et exclusion des sauvegardes

```
$ cd /home/remote/wikipedia

$ vi ~/.autorestic.yml

global:
  ...
  options:
    backup:
      exclude:
      ....
      - '/home/remote/partimag/*'
      - '/home/remote/wikipedia/*'


```

Téléchargement des fichiers ``.zim`` (ex: 37,31G  1,75MB/s    in 4h 5m)

```
$ wget https://download.kiwix.org/zim/wikipedia/wikipedia_fr_all_maxi_2023-06.zim
$ wget https://download.kiwix.org/zim/wikipedia/wikipedia_wo_all_maxi_2023-09.zim
```

```
$ ~master/kiwix/kiwix-manage library.xml add *.zim
```

Création du service de démarrage
```
$ vi /etc/systemd/system/kiwix.service

[Unit]
Description=Kiwix

[Service]
Type=simple
User=kiwix
ExecStart=/home/master/kiwix/kiwix-serve --library /home/remote/wikipedia/library.xml --address 127.0.0.1 --port 8081 --daemon
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

$ systemctl daemon-reload
$ systemctl enable kiwix

$ systemctl status kiwix

```
