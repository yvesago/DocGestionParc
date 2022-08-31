Accès utilisateurs
==================

_Pour information_ : cette configuration est appliquée automatiquement lors de l'installation des clients.

L'usage n'est pas nécessaire pour l'accès aux serveurs. 

Accès des utilisateurs définis dans ldap (glauth)

``sssd`` est utilisé pour identifier les utilisateurs définis dans ldap

```
$ apt install sssd
```

## Configuration

_la connexion avec ldap est impérativement chiffrée_

```
$ vi /etc/sssd/sssd.conf
    [sssd]
    services = nss, pam
    config_file_version = 2
    domains = default

    [nss]

    [pam]
    offline_credentials_expiration = 60

    [domain/default]
    ldap_id_use_start_tls = True
    cache_credentials = True
    ldap_search_base = dc=glauth,dc=com
    id_provider = ldap
    auth_provider = ldap
    sudo_provider = none
    chpass_provider = none
    access_provider = ldap
    ldap_uri = ldaps://ldap:3894
    ldap_default_bind_dn = cn=serviceuser,dc=glauth,dc=com
    ldap_default_authtok = ***SEC***defini_dans_glauth***
    ldap_default_authtok_type = password
    ldap_tls_reqcert = demand
    ldap_tls_cacert = /etc/ssl/certs/server.local.lan.crt
    ldap_tls_cacertdir = /etc/ssl/certs
    ldap_search_timeout = 50
    ldap_network_timeout = 60
    ldap_access_order = filter
    ldap_access_filter = (objectClass=posixAccount)
    ldap_group_member = member
    ldap_schema = rfc2307bis
    enumerate = true
```

L'usage d'un certificat autosigné nécessite de préciser :

```
$ vi /etc/ldap/ldap.conf
TLS_CACERT      /etc/ssl/certs/server.local.lan.crt
```

Astuce pour récupérer en une ligne le certificat depuis le serveur
```
$ openssl s_client -connect ldap:3894 -showcerts < /dev/null | \
  openssl x509 -text | \
  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' \
   > /etc/ssl/certs/server.local.lan.crt
```

## Création du home sur le client

_potentiellement utilisable sur le serveur_

Si ``homedir`` est défini dans ldap pour chaque utilisateur

```
$ vi /etc/pam.d/common-session
...
session optional                    pam_sss.so  #installé par sssd
session required        pam_mkhomedir.so skel=/etc/skel/ umask=0022
...
```

