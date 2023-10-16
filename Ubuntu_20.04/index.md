---
title: Documentation
---
# Documentation

Installation d'un serveur Ubuntu 20.04 LTS pour le partage de connexions filaire et wifi. Gestion des utilisateurs et déploiement d'un parc de machines.

[Conventions d'écriture](00-Conventions.md)


### Plan
1. [Installation du serveur](10-Install-serveur.md)
   - [certificats](15-certificats.md)
2. [Réseau](20-Network.md)
   - [wifi](22-network-wifi.md)
   - [dhcp et dns](25-network-dhcp-dns.md)
   - [iptables](28-network-iptables.md)
3. [Comptes](30-Comptes.md)
   - [ldap](32-comptes-ldap.md)
   - [interface web](33-comptes-web.md)
   - [radius](35-comptes-radius.md)
4. [Proxy-cache des mises à jours](40-Proxy-cache-apt.md)
5. [Sauvegardes](50-Sauvegardes.md)
6. [Control](60-Control.md)
   - [control à distance](65-remote-control.md)
7. [Accès utilisateurs](70-Acces-utilisateurs.md)
8. [Partage de fichiers (HTTP)](80-Partage-fichiers-HTTP.md)
   - [Wikipedia local](81-Wikipedia-local.md)
   - [partage de fichiers (NFS)](85-partage-fichiers-NFS.md)
9. [Installation des machines clientes](90-Installation-clients.md)
   - **Ubuntu 22.04 Desktop** (PXE et cloud-init) et/ou **PrimTux 7** (PXE)
   - [clonage des machines clientes](92-clone-clients.md)
   - gestion du parc avec ``ansible`` **TODO**
10. [Sécurité](99-Securite.md)
