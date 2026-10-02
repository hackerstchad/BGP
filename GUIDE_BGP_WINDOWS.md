# Guide Pratique BGP sous Windows

## 1. Outils disponibles sur Windows

### 1.1 Outils de vérification et diagnostic réseau natifs

| Outil | Commande | Utilité |
|-------|----------|---------|
| PowerShell | `Test-NetConnection` | Tester la connectivité TCP sur le port 179 (BGP) |
| PowerShell | `Get-NetRoute` | Voir les routes locales |
| Command Prompt | `route print` | Afficher la table de routage Windows |
| Command Prompt | `tracert` / `pathping` | Tracer les routes et détecter les pertes |
| Wireshark | Interface graphique | Capturer et analyser les paquets BGP |

### 1.2 Outils tiers recommandés

| Outil | Type | Description |
|-------|------|-------------|
| Wireshark | Analyseur de protocoles | Capture les messages BGP (OPEN, UPDATE, NOTIFICATION, KEEPALIVE) |
| GNS3 | Simulateur réseau | Émule des routeurs Cisco, Juniper, etc. sur Windows |
| EVE-NG Community | Simulateur / émulateur | Alternative à GNS3, plus moderne, nécessite VM |
| Cisco Packet Tracer | Simulateur Cisco | Gratuit pour étudiants, idéal pour apprendre BGP |
| BIRD (Windows port) | Routeur logiciel | Routeur BGP open source, utilisable en ligne de commande |
| ExaBGP | Routeur BGP Python | Peut s'installer via WSL ou Python natif |
| FRRouting | Suite de routage | Via WSL2 ou Docker Desktop sur Windows |
| Quagga / FRR | Routeur logiciel | Principalement Linux, mais fonctionne dans WSL2 |

## 2. Vérification BGP avec PowerShell

### 2.1 Tester si le port BGP (179) est accessible

```powershell
Test-NetConnection -ComputerName 192.168.1.1 -Port 179
```

Sortie attendue si le port est ouvert :

```
TcpTestSucceeded : True
```

### 2.2 Vérifier les routes locales

```powershell
Get-NetRoute -AddressFamily IPv4 | Where-Object { $_.DestinationPrefix -eq "0.0.0.0/0" }
```

### 2.3 Afficher la table de routage Windows

```cmd
route print
```

### 2.4 Vérifier les connexions TCP actives

```powershell
Get-NetTCPConnection -LocalPort 179
Get-NetTCPConnection -RemotePort 179
```

## 3. Wireshark : capturer et filtrer BGP

### 3.1 Installation

1. Télécharger Wireshark depuis https://www.wireshark.org/download.html
2. Installer avec Npcap (inclus).

### 3.2 Filtres utiles pour BGP

| Filtre | Description |
|--------|-------------|
| `bgp` | Affiche tous les paquets BGP |
| `bgp.type == 1` | Messages OPEN |
| `bgp.type == 2` | Messages UPDATE |
| `bgp.type == 3` | Messages NOTIFICATION |
| `bgp.type == 4` | Messages KEEPALIVE |
| `tcp.port == 179` | Tout le trafic TCP sur le port BGP |

## 4. Simuler BGP sur Windows

### 4.1 Cisco Packet Tracer

1. Télécharger sur https://www.netacad.com/courses/packet-tracer
2. Créer un réseau avec des routeurs ISR/2911.
3. Activer BGP :

```
Router> enable
Router# configure terminal
Router(config)# router bgp 65001
Router(config-router)# neighbor 192.168.1.2 remote-as 65002
Router(config-router)# network 10.0.0.0 mask 255.255.255.0
```

### 4.2 GNS3 sur Windows

1. Installer GNS3 + GNS3 VM (VirtualBox ou VMware).
2. Importer une image IOS (ex: c7200, c3745, IOSv).
3. Créer une topologie avec deux routeurs.
4. Configurer BGP de la même manière que sur du matériel réel.

### 4.3 EVE-NG

1. Installer EVE-NG Community sous VirtualBox/VMware Workstation.
2. Utiliser des images de routeurs (Cisco IOL, CSR1000v, Juniper vMX, etc.).
3. Créer un lab avec des liens entre routeurs et configurer BGP.

### 4.4 ExaBGP via WSL2 ou Python Windows

Installation :

```bash
pip install exabgp
```

Configuration minimale (`exabgp.conf`) :

```
neighbor 192.168.1.2 {
    router-id 192.168.1.1;
    local-address 192.168.1.1;
    local-as 65001;
    peer-as 65002;
    static {
        route 10.0.0.0/24 next-hop 192.168.1.1;
    }
}
```

Lancement :

```bash
exabgp exabgp.conf
```

### 4.5 FRRouting avec Docker Desktop

```powershell
docker pull frrouting/frr:latest
docker run -it --privileged --name frr-bgp frrouting/frr:latest
```

Puis configurer via `vtysh` :

```
configure terminal
router bgp 65001
 neighbor 192.168.1.2 remote-as 65002
 network 10.0.0.0/24
```

## 5. Configuration réelle d'un routeur BGP (exemples)

### 5.1 Cisco IOS

```
! Activer le routage BGP
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 203.0.113.2 remote-as 65002
 neighbor 203.0.113.2 description Peer-ISP

! Annoncer un réseau
 network 198.51.100.0 mask 255.255.255.0

! Route-map pour filtrer les routes sortantes
 neighbor 203.0.113.2 route-map EXPORT out

route-map EXPORT permit 10
 match ip address prefix-list MES-RESEAUX

ip prefix-list MES-RESEAUX seq 5 permit 198.51.100.0/24
```

Vérification :

```
show ip bgp summary
show ip bgp neighbors
show ip bgp
show ip route bgp
```

### 5.2 Juniper JunOS

```
set routing-options autonomous-system 65001
set routing-options router-id 1.1.1.1
set protocols bgp group EBGP-PEER type external
set protocols bgp group EBGP-PEER neighbor 203.0.113.2 peer-as 65002
set protocols bgp group EBGP-PEER export EXPORT-MES-RESEAUX

set policy-options policy-statement EXPORT-MES-RESEAUX term 1 from protocol direct
set policy-options policy-statement EXPORT-MES-RESEAUX term 1 from route-filter 198.51.100.0/24 exact
set policy-options policy-statement EXPORT-MES-RESEAUX term 1 then accept
set policy-options policy-statement EXPORT-MES-RESEAUX term default then reject
```

Vérification :

```
show bgp summary
show bgp neighbor
show route protocol bgp
```

### 5.3 FRRouting / BIRD

Exemple FRR (`/etc/frr/frr.conf`) :

```
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 203.0.113.2 remote-as 65002
 address-family ipv4 unicast
  network 198.51.100.0/24
 exit-address-family
```

## 6. Checklist de vérification BGP

- [ ] Connectivité IP entre les deux voisins OK
- [ ] Port TCP 179 autorisé dans les pare-feu
- [ ] AS local et AS distant correctement configurés
- [ ] Router-ID unique et valide
- [ ] Neighbor address joignable
- [ ] Pas de filtre route-map/prefix-list bloquant
- [ ] Routes à annoncer présentes dans la table de routage
- [ ] TTL / eBGP multihop configuré si nécessaire
- [ ] Authentification MD5 configurée des deux côtés si utilisée

## 7. Commandes de dépannage Windows complémentaires

```powershell
# Voir les interfaces réseau
Get-NetIPInterface

# Voir les adresses IP
Get-NetIPAddress

# Résoudre un nom DNS
Resolve-DnsName google.com

# Capturer réseau nativement (netsh)
netsh trace start capture=yes tracefile=c:\temp\capture.etl
netsh trace stop
```

## 8. Ressources supplémentaires

- RFC 4271 : A Border Gateway Protocol 4 (BGP-4)
- RFC 4456 : BGP Route Reflection
- RFC 1997 : BGP Communities Attribute
- RFC 4360 : BGP Extended Communities
- Cisco BGP documentation : https://www.cisco.com/c/en/us/support/ip/border-gateway-protocol-bgp/ 
- Juniper BGP documentation : https://www.juniper.net/documentation/en_US/junos/topics/concept/bgp-overview.html
- ExaBGP : https://github.com/Exa-Networks/exabgp
- FRRouting : https://frrouting.org/

---

Ce guide permet de vérifier, configurer et simuler BGP depuis un poste Windows, que ce soit en production, en lab, ou pour l'apprentissage.
