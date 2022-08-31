# Sécurité

## Objectifs de sécurité

Les objectifs sont pédagogiques et la facilité d'installation dans un point d'accès mal maitrisé. Par exemple : box personnelle, accès partagé 4G.

Ce type d'installation est destiné à un **petit groupe de confiance** : école primaire, association, évènement ponctuel, ...


Limitations :
 - le partage NFS permet des accès en lecture de tous les utilisateurs
 - certains secrets sont accessibles (sous forme chiffrée) depuis les fichiers d'installation automatique
 - la possibilité d'installation d'un deuxième système client donne accès, aux administrateurs, aux données de l'autre système enregistré sur le même disque
 - les utilisateurs doivent expressément enregistrer leur données sur le dossier partagé du serveur
 - il n'y a pas de mécanisme de serveur de secours

Contre-mesures :
 - les données personnelles doivent être chiffrées avec un container de mot de passe et/ou des dossiers chiffrés
 - les sauvegardes externes sur clé usb ou disque externe doivent être gérées
 - un serveur de secours cloné doit synchroniser régulièrement les comptes ldap (fichier ``/etc/glauth/sample-simple.cfg``) et les données personnelles dans ``/home/remote``



## Secrets

### Critiques

La perte ou la divulgation peut empêcher l'accès, diffuser des informations confidentielles ou détruire les systèmes.

**Mots de passe administrateurs**
 - mot de passe administrateur du serveur lors de l'[installation du serveur](10-Install-serveur.md)
 - mot de passe administrateur client dans [cloud-init](user-data) ou pour [PrimTux](70-Acces-utilisateurs.md)

**Sauvegardes**
 - clés d'accès  ou ``/root/.autorestic.yml`` ou clé de secours de [restic](50-Sauvegardes.md)

**PKI**
 - mot de passe de [signature](15-certificats.md)

### Importants

La perte peut nuire au fonctionnement.

Un mot de passe oublié peut être changé par l'administrateur.

La divulgation peut donner accès à des informations sensibles.

**Accès à distance**
 - clé d'[accès à distance](65-remote-control.md)

**Accès ldap**
 - création ou modification des droits d'accès des utilisateurs depuis l'[interface web](33-comptes-web.md) ou directement depuis le fichier des comptes [ldap](32-comptes-ldap.md)

**Administrateur utilisateurs ldap**
 - mot de passe du compte ``serviceuser`` défini dans ldap et diffusé sur les clients avec [cloud-init](70-Acces-utilisateurs.md)

### Secondaires

La perte peut bloquer le fonctionnement

Le mot de passe peut facilement être changé par l'administrateur

La divulgation n'a pas ou de faible conséquences.

**Liaison freeradius hostapd**
Le mot de passe n'est utilisable que localement

**Secret web contre CSRF**
Utilisable uniquement dans le cadre improbable d'une attaque CSRF contre l'[application web de gestion des utilisateurs](33-comptes-web.md)

