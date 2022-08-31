# Comptes utilisateurs > radius

Usage de radius pour authentification wifi **TTLS/PAP** (authentification sur ldap).

L’authentification ldap est tunnelisée après un accès wpa 2 entreprise.

## Installation freeradius

```
$ apt install freeradius
$ apt install freeradius-ldap
```

## Configuration

### accès

```
$ vi /etc/freeradius/3.0/clients.conf
---------------
client localhost {
    ipaddr = 127.0.0.1
    secret = supersecretsecret  # cf /etc/hostpad/hostapd.conf
}
---------------
```

### activation ldap

```

$ cd /etc/freeradius/3.0/mods-enabled/
$ ln -s ../mods-available/ldap ./

$ vi /etc/freeradius/3.0/mods-enabled/ldap
---------------
ldap {
        server = 'ldaps://localhost'

        #  Port to connect on, defaults to 389, will be ignored for LDAP URIs.
        port = 3894

        #  Administrator account for searching and possibly modifying.
        #  If using SASL + KRB5 these should be commented out.
        identity = 'uid=serviceuser,dc=glauth,dc=com'
        password = ***SEC***defini_par_glauth***

        #  Unless overridden in another section, the dn from which all
        #  searches will start from.
        base_dn = 'dc=glauth,dc=com'

        ....
        tls {
            start_tls = no
            # le certificat autosigné est local
            require_cert    = 'never'

---------------
```

### logs sur une ligne dans syslog

```

$ vi /etc/freeradius/3.0/mods-available/linelog

add to log one line auth in syslog
---------------
linelog authlog {
         filename = syslog
         syslog_facility = local3
         escape_filenames = no
         permissions = 0600
         format = ""
         reference = "authlog.%{%{reply:Packet-Type}:-default}"
         authlog {
                 default = "Unknown packet type %{Packet-Type}"
                 Access-Accept = "Login OK: %{User-Name} (from SSID: %{Called-Station-SSID} client %{NAS-Identifier} port %{NAS-Port} cli %{Calling-Station-Id} %{Called-Station-Id}) Auth-Type:%{control:Auth-Type} Vlan:%{Tunnel-Private-Group-ID}"
                 Access-Reject = "Login incorrect(%{Module-Failure-Message}): %{User-Name} (from SSID: %{Called-Station-SSID} client %{NAS-Identifier} port %{NAS-Port} cli %{Calling-Station-Id}) %{Called-Station-Id} Auth-Type:%{control:Auth-Type}"
                 Access-Challenge = "Sent challenge: %{User-Name}"
         }
}
---------------

```

### site principal

```
$ vi /etc/freeradius/3.0/sites-enabled/default
---------------
authorize {
...
        eap {
                ok = return
        }

        if (!control:Auth-Type) {
                ldap

                if (ok && User-Password) {
                        update {
                        control:Auth-Type := LDAP
                        }
                }
        }
        expiration
       logintime
...
}

authenticate {
...
# décommenter:
#       Auth-Type LDAP {
#               ldap
#       }
...
}

post-auth {
    .....
    authlog
}


---------------
```

### site inner-tunnel

```

$ vi /etc/freeradius/3.0/sites-enabled/inner-tunnel
---------------
authorize {
   ....
        #  The ldap module reads passwords from the LDAP database.
        #-ldap
        ldap
        if ((ok || updated) && User-Password) {
            update {
                control:Auth-Type := ldap
            }
        }
...

authenticate {
# décommenter:
#       Auth-Type LDAP {
#               ldap
#       }

...
---------------
```

### eap
```
$ vi /etc/freeradius/3.0/mods-enabled/eap
eap {
    default_eap_type = ttls
...

    tls-config tls-common {
        #private_key_password = whatever
        #private_key_file = /etc/ssl/private/ssl-cert-snakeoil.key
        private_key_file = /etc/ssl/private/server.local.lan.key
    ...
        #certificate_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
        certificate_file = /etc/ssl/certs/server.local.lan.crt
    ...
        #ca_file = /etc/ssl/certs/ca-certificates.crt
        ca_file = /etc/ssl/certs/localcacert.pem
    ...
        #       disable_tlsv1_2 = no
        #       disable_tlsv1_1 = yes
        #       disable_tlsv1 = yes
    ...
        tls_min_version = "1.0"
        tls_max_version = "1.3"
    ...

    ttls {
    ...
    copy_request_to_tunnel = yes
    use_tunneled_reply = yes
    ...
    require_client_cert = no
    }
```

## Démarrage auto

à modifier pour démarrer après glauth

```
$ vi /lib/systemd/system/freeradius.service
---------------
[Unit]
Description=FreeRADIUS multi-protocol policy server
#After=network.target
After=network.target glauth.service
Wants=glauth.service
Documentation=man:radiusd(8) man:radiusd.conf(5) http://wiki.freeradius.org/ http://networkradius.com/doc/
StartLimitBurst=5
StartLimitIntervalSec=50

....
---------------

$ systemctl enable freeradius

```

## tests

```
# mode debug
$  freeradius -X &

$ radtest serviceuser mysecret 127.0.0.1 0 supersecretsecret
Sent Access-Request Id 189 from 0.0.0.0:48761 to 127.0.0.1:1812 length 81
	User-Name = "serviceuser"
	User-Password = "mysecret"
	NAS-IP-Address = 127.0.1.1
	NAS-Port = 0
	Message-Authenticator = 0x00
	Cleartext-Password = "mysecret"
Received Access-Accept Id 189 from 127.0.0.1:1812 to 127.0.0.1:48761 length 20

$ radtest johndoe dogood 127.0.0.1 0 supersecretsecret
...
Received Access-Accept ..

```

## logs

```
$ grep freeradius /var/log/syslog
```

### docs générales

https://hackernoon.com/how-to-secure-your-wifi-network-with-freeradius-94e0812a83bf

https://openschoolsolutions.org/freeradius-secure-wifi-network/

https://wiki.bigd.fr/index.php/Mise_en_place_d%27un_point_d%27acc%C3%A8s_WIFI_WPA2_avec_FreeRadius

https://it.izero.fr/linux-mise-en-place-dun-serveur-freeradius/

