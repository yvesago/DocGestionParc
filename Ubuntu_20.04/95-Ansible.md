Gestion de parc
======================

``Ansible`` permet de lancer des commandes depuis le serveur sur toutes les machines clientes allumées.

L'accès est possible suite à l'ajout, lors de l'installation avec ``cloud-init`` de la clé ssh publique de l'utilisateur ``root`` du serveur principal.

Le contenu de ``/root/.ssh/id_rsa.pub`` du serveur a été ajoutés au fichier ``/home/ubuntu/.ssh/authorized_keys`` de chaque machine.


## Installation serveur

```
$ apt install ansible


$ mkdir -p ~/ansible/inventaires
$ mkdir playbooks
```

### Configuration
```
$ vi ~/.ansible.cfg
[defaults]
inventory       = ~/ansible/inventaires/hosts
#stdout_callback = yaml

[privilege_escalation]
become = True
```

``become`` indique que les commandes seront exécutées sur le client avec les privilèges d'administrateur


## Création de la liste des machines


### En-tête du fichier ``host``
```
$ vi ~/ansible/inventaires/host

[machines:vars]
ansible_ssh_user = ubuntu
ansible_python_interpreter = /usr/bin/python3

[machines]
```

### Détection initiale des machines 

```
$ nmap -sP 192.168.200.0/24 | grep report | grep -v '(' >> ~/ansible/inventaires/hosts
```

Il faut ensuite corriger le fichier ``host`` : seule l'IP identifie les machines


## Usage

```
# liste des machines allumées
$ ansible machines -m ping

# lancement de la commande uptime (durée depuis le démarrage) sur toutes les machines
$ ansible machines -m shell -a uptime

# mise à jour immédiate des machines
$ ansible machines -m apt -a "upgrade=yes update_cache=yes cache_valid_time=86400" --become

# mise à jour immédiate des machines
$ ansible machines -m apt -a "upgrade=yes" --become
```

### Console

``ansible-console`` permet de se connecter simultanément sur toutes les machines allumées.

```
$ ansible-console machines

root@machines (19)[f:5]# list
root@machines (19)[f:5]# ping
root@machines (19)[f:5]# uptime

root@machines (19)[f:5]# shell apt-get install synaptic

root@machines (19)[f:5]# cd 192.168.200.xxx
root@machines (19)[f:5]# shell 
root@machines (19)[f:5]# ls /mnt/nfs/remote

root@machines (19)[f:5]# exit
```

### Playbook

``ansible-playbook`` permet de créer des fichiers de commandes


```
$ ansible-playbook -l machines playbooks/available-updates.yml
```

exemple: recherche des mises à jours disponibles

```
$ cat ../ansible/playbooks/available-updates.yml
---
- hosts: all
  tasks:
    - name: Update cache
      apt:
       update_cache: yes

    - name: Check upgradable
      command: apt list --upgradable
      register: updates
      async: 45
      poll: 5
    - debug: var=updates.stdout_lines

    - name: Check if a reboot is needed for Debian and Ubuntu boxes
      register: reboot_required_file
      stat: path=/var/run/reboot-required get_md5=no
      async: 45
      poll: 5

    - name: uptime
      command: uptime
      register: hostname
      async: 45
      poll: 5
    - debug: var=hostname.stdout_lines
```

exemple: mises à jours immédiates avec nettoyage des paquets orphelins

```
$ cat ../ansible/playbooks/upgrade.yml
---
- hosts: machines
  become: yes
  tasks:
    - name:
      apt: upgrade=yes force_apt_get=yes autoremove=yes autoclean=yes
      register: upgrade
    - debug: var=upgrade.stdout_lines
```


exemple: ajout d'une ligne dans un fichier de configuration (montage nfs)

```
$ cat ../ansible/playbook/add-fstab.yml
---
  - name: add to fstab
    hosts: all

    tasks:
      - name: "line insert"
        lineinfile:
           path: "/etc/fstab"
           line: "192.168.200.1:/home/remote /mnt/nfs/remote/  nfs      defaults    0       0"
           insertbefore: EOF
```
