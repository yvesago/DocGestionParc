# wifi

Création d'un service de diffusion et d'accès wifi

```
$ apt install wireless-tools  # pour iwconfig, iwlist, ...
```

### identification support du mode AP (dépend du firmware)

```
$ iw phy0 info
  ...
  Supported interface modes
		 * IBSS
		 * managed
		 * AP      <==
		 * AP/VLAN
		 * monitor
		 * mesh point

$ iw dev
phy#0
	Interface wlan0

```

### firmware spécifique

si absence du mode AP

```
$ sudo apt install firmware-b43-installer
```

## Installation hostapd

```
$ apt install hostapd
```

# doc

https://doc.ubuntu-fr.org/hostapd

https://developer.toradex.com/knowledge-base/wi-fi-access-point-mode

# Configuration

```
$ vi /etc/hostapd/hostapd.conf

---------------
ssid=MonSSID
#interface=wlp3s0b1
# wlp3s0b1 renamed to wlan0 with netplan
interface=wlan0
macaddr_acl=0
#accept_mac_file=/etc/hostapd.accept
#deny_mac_file=/etc/hostapd.deny
ieee8021x=1
country_code=FR
hw_mode=g

logger_syslog=-1
logger_syslog_level=0

## WPA ##
wpa=2
wpa_key_mgmt=WPA-EAP
channel=1
wpa_pairwise=TKIP CCMP
wpa_ptk_rekey=600

## Radius ##
auth_server_addr=127.0.0.1
auth_server_port=1812
auth_server_shared_secret=***SEC***defini_dans_radius*** 
auth_algs=3
own_ip_addr=127.0.0.1
```



### activation/désactivation manuelle wifi

```
$ apt install rfkill
$ rfkill list
$ rfkill unblock 0
```

# Démarrage automatique

```
$ systemctl unmask hostapd 
$ systemctl enable hostapd 
$ systemctl start hostapd  
```

