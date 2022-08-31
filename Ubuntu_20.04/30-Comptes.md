
Comptes serveurs
================

Comptes d'accès au serveur

_Pour information:_ le premier compte est suffisant pour effectuer toutes les opérations de gestion. Les utilisateurs seront gérés depuis ``ldap``. 

## Création d'un compte supplémentaire
```
$ adduser yves
Adding user `yves' ...
Adding new group `yves' (1001) ...
Adding new user `yves' (1001) with group `yves' ...
Creating home directory `/home/yves' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for yves
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n]
```

### autorisation sudo
```
$ usermod -aG sudo yves
```

## Compte system
ne permet pas de se connecter au serveur. Utilisé pour limiter les accès d'une application spécifique.
```
$ useradd --system --no-create-home --shell /bin/false appuser
```

## Suppression d'un compte
```
$ userdel appuser
```

