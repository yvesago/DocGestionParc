# Certificats SSL

Certificat auto signé de 10 ans

_**let's encrypt** est un bon choix mais nécessite un renouvellement tous les 3 mois_

Les certificats auto-signés permettent d'envisager d'éventuelles coupures d'accès à Internet de plus de 3 mois.


## Certificat unique

_solution simple mais évolutions difficiles_

```
$ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout server.local.lan.key -out server.local.lan.crt -subj "/CN=server.local.lan/OU=local.lan" \
  -addext "subjectAltName=DNS:ldap,DNS:proxy,DNS:nfs,DNS:www,DNS:*.local.lan"
```



``basicConstraints=CA:true`` serai indispensable pour Android 11 
mais pas utilisable avec les navigateurs !!!


### installation

```
$ cp server.local.lan.crt /etc/ssl/certs
$ cp server.local.lan.key /etc/ssl/private/


$ chmod 640 /etc/ssl/private/server.local.lan.key

$ addgroup ssl-cert
$ chown root:ssl-cert /etc/ssl/private/server.local.lan.key

$ chmod g+rx /etc/ssl/private/
$ chown root:ssl-cert /etc/ssl/private/
```


## Certificat wifi spécifique pour freeradius

Depuis Android 11, l'accès au réseau wifi a été renforcé. La validation d'un certificat est devenu impérative.

Lors de la configuration de l'accès au réseau avec ``wpa2`` (nous utilisons ici un accès ``TTLS+PAP``), l'option ``Domaine`` doit être équivalente au sujet du certificat. Ici ``radius.local.lan``

Le certificat public ``radius.local.lan.crt`` doit avoir l'extension ``basicConstraints = critical, CA:TRUE``.

Ce certificat doit être enregistré sur chaque appareil Android comme un "Certificat Wi-Fi"

Généralement dans les menus :
Paramètres > Sécurité > Paramètres avancés > Chiffrement et identifiants > Installer un certificat > Certificat Wi-Fi

Ce certificat doit ensuite être sélectionné dans l'option "Certificat CA"

L'option "État du certificat en ligne" peut conserver la valeur "Ne pas valider"


```
$ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout radius.local.lan.key -out radius.local.lan.crt -subj "/CN=radius.local.lan" \
  -addext "basicConstraints = critical, CA:TRUE"
```

### installation

```
$ cp radius.local.lan.crt /etc/ssl/certs
$ cp radius.local.lan.key /etc/ssl/private/


$ chmod 640 /etc/ssl/private/radius.local.lan.key
$ chown :ssl-cert /etc/ssl/private/radius.local.lan.key
```

## PKI

Une infrastructure de gestion de clés (PKI) va permettre de définir un certificat public global du domaine.

Les certificats de services peuvent être ajoutés ou modifiés.

_documentation complète Ubuntu : https://ubuntu.com/server/docs/security-certificates_

```
$ mkdir /etc/ssl/CA
$ mkdir /etc/ssl/newcerts
```

### définition des numéros de série
```
$ sh -c "echo '01'" > /etc/ssl/CA/serial
$ touch /etc/ssl/CA/index.txt
```

### options par défaut
```
$ vi /etc/ssl/openssl.cnf
# quelques changements
[ CA_default ]
dir             = /etc/ssl              # Where everything is kept
database        = $dir/CA/index.txt     # database index file.
certificate     = $dir/certs/localcacert.pem # The CA certificate
serial          = $dir/CA/serial        # The current serial number
private_key     = $dir/private/localcakey.pem# The private key
default_days    = 3650

# decommenter ligne suivante pour les AltNames
copy_extensions = copy

# passer en optionnel
[ policy_match ]
countryName             = optional
stateOrProvinceName     = optional
organizationName        = optional


[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = LN
countryName_min                 = 2
countryName_max                 = 2

stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     =

localityName                    = Locality Name (eg, city)

0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = Local


```

### création de la clé privée et du certificat public
```
$ openssl req -new -x509 -extensions v3_ca -keyout localcakey.pem -out localcacert.pem -days 3650

Generating a RSA private key
......................................................................................+++++
.............................................................................+++++
writing new private key to 'localcakey.pem'
Enter PEM pass phrase: ***SEC***
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [LN]:
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) [Local]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:master@local.lan
```

``localcakey.pem`` doit rester strictement confidentielle ``***SEC***``

``localcacert.pem`` peut être diffusé

**installation**
```
$ cp localcakey.pem /etc/ssl/private/
$ cp localcacert.pem /etc/ssl/certs/
$ cp localcacert.pem /var/www/
```

Le certificat [http://www.local.lan/localcacert.pem](http://www.local.lan/localcacert.pem) peut ensuite être enregistré dans les profils de certificats d'autorité de Firefox


### Création de requête de CSR

**avec création de la clé privée ``server.local.lan.key``**
```
$ openssl req -new -newkey rsa:4096 -nodes \
  -keyout server.local.lan.key -out server.csr -subj "/CN=server.local.lan/OU=local.lan" \
  -addext "subjectAltName=DNS:ldap,DNS:proxy,DNS:nfs,DNS:www,DNS:*.local.lan"

# lecture
$ openssl req -text -noout -verify -in server.csr
```

**signature (validation) de la demande**
```
$ openssl ca -in server.csr -config /etc/ssl/openssl.cnf
```

**certificat public ``server.local.lan.crt``**
```
$ cp /etc/ssl/newcerts/01.pem server.local.lan.crt

# visualisation
$ openssl x509 -text -in server.local.lan.crt
...
Validity
     Not Before: Aug  9 14:09:43 2022 GMT
     Not After : Aug  6 14:09:43 2032 GMT
Subject: OU = local.lan, CN = server.local.lan
...
```

**installation**
```
$ cp server.local.lan.key /etc/ssl/private/
$ cp server.local.lan.crt /etc/ssl/certs/
```

``server.local.lan.key`` doit rester confidentielle ``***SEC***``

``server.local.lan.key`` et ``server.local.lan.crt`` peuvent être utilisés par les services pour chiffrer les flux

