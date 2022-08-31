# Conventions d'écriture

_pour information_

**Les lignes qui commencent par ``$`` sont une commande depuis le terminal**

exemple:
```
$ vi /var/www/00-Conventions.md
```

------------------

**Avec ubuntu le premier utilisateur défini peut exécuter des commandes avec de privilèges d'administrateur (root).** 

Dans ce document toutes les commandes sont exécutées après que le premier utilisateur ait élevé ses privilèges avec la commande ``$ sudo -s``

exemple :
```
master@serv1:~$ sudo -s
[sudo] password for master: 
root@serv1:/home/master# 
```

-----------------

**Les fichiers sont édités avec ``vim`` (vi)**

``vi`` possède un mode consultation par défaut et un mode édition. La touche ``Esc`` permet de changer de mode. 

Quelques commandes de base doivent être connues :

Mode consultation (par défaut)
- ``:q`` permet de quitter ``vi``
- ``:q!`` forcer quitter 
- ``:wq`` enregistrer le fichier (write) et quitter
- ``i`` permet de passer en mode modification (insertion)
- ``/terme à chercher`` permet d'effectuer une recherche de "terme à chercher"
- ``n`` chercher l'occurrence suivante
- ``$`` aller en fin de ligne
- ``G`` aller en fin de document
- ``:1`` aller à la ligne 1
- ``u`` annule la dernière modification


Mode édition
- flèches, ``Fin`` permet de se déplacer dans le document
- ``Esc`` passer en mode consultation

