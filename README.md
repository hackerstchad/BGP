# 🌐 BGP — Border Gateway Protocol

<img width="1248" height="832" alt="OIG2" src="https://github.com/user-attachments/assets/35e8d549-0405-4690-8d41-d99e061a019f" />

---

[GUIDE PRATIQUE] (https://github.com/hackerstchad/BGP/blob/main/GUIDE_BGP_WINDOWS.md)
<img width="1248" height="832" alt="OIG2 tSS3uerrxcGsKO" src="https://github.com/user-attachments/assets/d4eb3c6f-f819-4227-a25f-2ed7d15e37d2" />


## Table des matières

1. [Qu'est-ce que BGP ?](#quest-ce-que-bgp)
2. [Historique et créateurs](#historique-et-créateurs)
3. [Versions du protocole](#versions-du-protocole)
4. [Types de BGP](#types-de-bgp)
5. [Fonctionnement interne](#fonctionnement-interne)
6. [Messages BGP](#messages-bgp)
7. [Attributs BGP](#attributs-bgp)
8. [Algorithmes de sélection](#algorithmes-de-sélection)
9. [Sécurité BGP](#sécurité-bgp)
10. [Resource Definition et templates](#resource-definition-et-templates)
11. [Commandes et configuration](#commandes-et-configuration)
12. [Outils de monitoring et de débogage](#outils-de-monitoring-et-de-débogage)
13. [Cas d'usage réels](#cas-dusage-réels)
14. [Bonnes pratiques](#bonnes-pratiques)
15. [RFCs et ressources](#rfcs-et-ressources)
16. [Glossaire](#glossaire)

---

## 1. Qu'est-ce que BGP ?

**BGP (Border Gateway Protocol)** est le protocole de routage externe utilisé pour échanger des informations de routage entre différents systèmes autonomes (AS) sur Internet. Il est considéré comme **le protocole principal de l'Internet moderne** car il permet à des milliers de réseaux indépendants de communiquer entre eux.

BGP est un protocole :
- **Path-vector** : il annonce les chemins complets (séquence d'AS) vers les destinations.
- **Stateful** : il maintient des sessions TCP persistantes entre voisins.
- **Incrementiel** : seules les modifications de routage sont annoncées après la synchronisation initiale.
- **Policy-driven** : les administrateurs peuvent fortement influencer les décisions de routage.

### Pourquoi BGP est indispensable ?

Sans BGP, chaque réseau serait une île isolée. BGP permet à un opérateur télécom, à un datacenter, à un FAI ou à une entreprise multinationale d'annoncer ses préfixes IP aux autres réseaux, et de recevoir en retour les routes vers le reste du monde.

### Caractéristiques clés

| Caractéristique | Description |
|-----------------|-------------|
| Type de protocole | Path-vector, EGP ( Exterior Gateway Protocol ) |
| Transport | TCP port 179 |
| Fiabilité | Élevée grâce à TCP et au mécanisme de keepalive |
| Évolutivité | Supporte plus de 900 000 routes IPv4 dans la table globale |
| Flexibilité | Très nombreux attributs et communautés pour le traffic engineering |

---

## 2. Historique et créateurs

### Origines

BGP a été créé à la fin des années 1980 pour remplacer **EGP (Exterior Gateway Protocol)**, devenu obsolète face à la croissance exponentielle d'Internet et aux besoins de routage plus complexes.

### Auteurs principaux

- **Kirk Lougheed** (Cisco) et **Yakov Rekhter** (IBM) sont les co-auteurs originels de la RFC 1105, première version de BGP.
- **Yakov Rekhter** est souvent considéré comme le "père de BGP" pour son rôle déterminant dans la conception de BGP-4.

### Chronologie

| Année | Événement |
|-------|-----------|
| 1989 | Publication de BGP-1 dans la RFC 1105 |
| 1990 | BGP-2 (RFC 1163) |
| 1991 | BGP-3 (RFC 1267) |
| 1994 | BGP-4 (RFC 1654), puis RFC 1771, standardisé dans RFC 4271 |
| 2000+ | Extensions : MP-BGP, MPLS/VPN, BGP Flowspec, BGPsec |

### Anecdote

Lors d'une réunion IETF, Lougheed et Rekhteraient rédigé la première version de BGP sur deux serviettes en papier dans un hôtel. Cette anecdote illustre l'ingéniosité et la rapidité avec lesquelles BGP a été conçu pour répondre à un besoin urgent.

---

## 3. Versions du protocole

### BGP-1 (RFC 1105)

- Première version.
- Supporte uniquement les réseaux classful (A, B, C).
- Pas de CIDR (Classless Inter-Domain Routing).

### BGP-2 (RFC 1163)

- Amélioration du format des messages.
- Meilleure gestion des mises à jour.

### BGP-3 (RFC 1267)

- Ajout de mécanismes de contrôle plus robustes.
- Toujours classful.

### BGP-4 (RFC 4271)

- **Version actuelle et dominante.**
- Support du CIDR et du VLSM (Variable Length Subnet Masking).
- Agrégation des routes (route summarization).
- Attributs étendus pour MPLS, VPN, IPv6 (via MP-BGP).
- Capable de transporter des NLRI pour plusieurs familles d'adresses.

### MP-BGP (Multiprotocol BGP)

Défini dans la RFC 4760, MP-BGP étend BGP-4 pour transporter des routes autres qu'IPv4 unicast, notamment :
- IPv6 unicast/multicast
- VPNv4 / VPNv6 (MPLS L3VPN)
- L2VPN
- Flowspec
- Labeled Unicast

---

## 4. Types de BGP

### eBGP — External BGP

- Établi entre deux routeurs appartenant à **des AS différents**.
- Utilisé pour échanger des routes entre opérateurs, FAI, entreprises, datacenters.
- Par défaut, eBGP modifie le TTL à 1 (sauf avec `ebgp-multihop`).

### iBGP — Internal BGP

- Établi entre deux routeurs du **même AS**.
- Propagation des routes apprises via eBGP à l'intérieur de l'AS.
- Nécessite une topologie full-mesh ou l'utilisation de route reflectors/confederations.

### Confédérations BGP

- Découpage d'un grand AS en plusieurs sous-AS.
- Simplifie la gestion iBGP.
- Défini dans la RFC 5065.

### Route Reflectors

- Permettent d'éviter le full-mesh iBGP.
- Un route reflector reçoit les routes d'un client et les réfléchit aux autres clients.
- Cluster-ID et originator-ID pour éviter les boucles.

---

## 5. Fonctionnement interne

### Établissement d'une session BGP

1. **Idle** : état initial.
2. **Connect** : tentative de connexion TCP sur le port 179.
3. **Active** : écoute active si la connexion échoue.
4. **OpenSent** : message OPEN envoyé.
5. **OpenConfirm** : message OPEN reçu et accepté.
6. **Established** : session active, échange de UPDATE, NOTIFICATION, KEEPALIVE.

### Messages BGP

| Type | Numéro | Description |
|------|--------|-------------|
| OPEN | 1 | Initialisation de la session, négociation des paramètres |
| UPDATE | 2 | Annonce ou retrait de routes |
| NOTIFICATION | 3 | Erreur fatale, fermeture de la session |
| KEEPALIVE | 4 | Maintien de la session |
| ROUTE-REFRESH | 5 | Demande de réenvoi des mises à jour |

### Tables BGP

- **Adj-RIB-In** : routes reçues d'un voisin avant politique.
- **Loc-RIB** : routes sélectionnées après politique et best path.
- **Adj-RIB-Out** : routes annoncées aux voisins après politique sortante.

---

## 6. Attributs BGP

### Attributs Well-known mandatory

| Attribut | Description |
|----------|-------------|
| AS_PATH | Liste des AS traversés |
| NEXT_HOP | Adresse IP du prochain saut |
| ORIGIN | Origine de la route (IGP, EGP, INCOMPLETE) |

### Attributs Well-known discretionary

| Attribut | Description |
|----------|-------------|
| LOCAL_PREF | Préférence locale en sortie de l'AS |
| ATOMIC_AGGREGATE | Indique qu'une route a été agrégée |

### Attributs Optional transitive

| Attribut | Description |
|----------|-------------|
| COMMUNITY | Tag pour regrouper des routes |
| AGGREGATOR | Routeur ayant créé l'agrégat |

### Attributs Optional non-transitive

| Attribut | Description |
|----------|-------------|
| MED | Metric utilisée pour influencer l'entrée dans l'AS |
| ORIGINATOR_ID | ID du routeur à l'origine d'une route réfléchie |
| CLUSTER_LIST | Liste des clusters traversés |

---

## 7. Algorithme de sélection du meilleur chemin BGP

Cisco utilise traditionnellement les critères suivants, dans l'ordre :

1. **Highest Weight** (Cisco-proprietary, local au routeur)
2. **Highest LOCAL_PREF**
3. **Locally originated** (routes générées localement)
4. **Shortest AS_PATH**
5. **Lowest ORIGIN type** (IGP < EGP < INCOMPLETE)
6. **Lowest MED**
7. **eBGP over iBGP**
8. **Lowest IGP metric to NEXT_HOP**
9. **Oldest route**
10. **Lowest Router ID**
11. **Shortest CLUSTER_LIST**
12. **Lowest neighbor IP address**

---

## 8. Sécurité BGP

### Menaces principales

- **Route hijacking** : annonce frauduleuse de préfixes.
- **Route leaks** : propagation involontaire de routes.
- **Man-in-the-middle** sur les sessions BGP.
- **DDoS** via la manipulation du routage.

### Mécanismes de protection

| Mécanisme | Description |
|-----------|-------------|
| MD5/TCP-AO | Authentification des sessions BGP |
| Prefix filtering | Filtrage des préfixes annoncés/reçus |
| RPKI | Resource Public Key Infrastructure, validation des ROAs |
| BGPsec | Signature cryptographique des chemins AS_PATH |
| IRRDB | Internet Routing Registry Database |
| Max-prefix limits | Limitation du nombre de routes reçues |
| Graceful shutdown | Basculement en douceur |

---

## 9. Resource Definition et templates

### YAML : Définition d'un voisin BGP

```yaml
bgp_neighbors:
  - neighbor: 203.0.113.1
    remote_as: 64500
    description: "Upstream Provider A"
    password: "S3cur3P@ss"
    soft_reconfiguration: inbound
    route_map_in: ACCEPT_DEFAULT_AND_CUSTOMER
    route_map_out: ANNOUNCE_CUSTOMER_ROUTES
    prefix_limit: 1000000
    afi_safi:
      - ipv4_unicast
      - ipv6_unicast
```

### JSON : Définition d'une policy BGP

```json
{
  "route_map": {
    "name": "PREFER_PEER_A",
    "entries": [
      {
        "sequence": 10,
        "match": { "as_path": "AS-PATH-FROM-A" },
        "set": { "local_preference": 200 },
        "action": "permit"
      }
    ]
  }
}
```

---

## 10. Commandes et configuration

### Cisco IOS / IOS-XE

```cisco
router bgp 65000
 bgp router-id 192.0.2.1
 neighbor 203.0.113.1 remote-as 64500
 neighbor 203.0.113.1 description Upstream-Provider-A
 neighbor 203.0.113.1 password S3cur3P@ss
 neighbor 203.0.113.1 soft-reconfiguration inbound
 !
 address-family ipv4 unicast
  neighbor 203.0.113.1 activate
  neighbor 203.0.113.1 route-map IN-FROM-UPSTREAM in
  neighbor 203.0.113.1 route-map OUT-TO-UPSTREAM out
  network 198.51.100.0 mask 255.255.255.0
 exit-address-family
```

### Juniper JunOS

```junos
protocols {
    bgp {
        group UPSTREAMS {
            type external;
            neighbor 203.0.113.1 {
                description "Upstream Provider A";
                peer-as 64500;
                authentication-key "$9$S3cur3P@ss";
                import IN-FROM-UPSTREAM;
                export OUT-TO-UPSTREAM;
            }
        }
    }
}
```

### Bird

```bird
protocol bgp upstream_a {
    description "Upstream Provider A";
    local as 65000;
    neighbor 203.0.113.1 as 64500;
    password "S3cur3P@ss";
    ipv4 {
        import filter in_from_upstream;
        export filter out_to_upstream;
    };
}
```

### FRRouting

```frr
router bgp 65000
 neighbor 203.0.113.1 remote-as 64500
 neighbor 203.0.113.1 description Upstream-Provider-A
 address-family ipv4 unicast
  neighbor 203.0.113.1 activate
  neighbor 203.0.113.1 route-map IN-FROM-UPSTREAM in
 exit-address-family
```

### Commandes de vérification

| Commande | Description |
|----------|-------------|
| `show ip bgp summary` | État des sessions BGP |
| `show ip bgp neighbors` | Détails des voisins |
| `show ip bgp` | Table BGP IPv4 |
| `show bgp ipv6 unicast` | Table BGP IPv6 |
| `show ip route bgp` | Routes BGP dans la FIB |
| `clear ip bgp * soft in` | Soft reset inbound |
| `clear ip bgp * soft out` | Soft reset outbound |

---

## 11. Outils de monitoring et de débogage

| Outil | Usage |
|-------|-------|
| BGPStream | Collecte et analyse des données BGP historiques et temps réel |
| RouteViews | Projets de collecte de tables de routage publiques |
| RIPE RIS | Collecte de routes pour la recherche et le monitoring |
| IRR Explorer | Validation des objets IRR/RPKI |
| BGPlay | Visualisation de l'évolution des annonces BGP |
| PeeringDB | Base de données des interconnexions réseau |

---

## 12. Cas d'usage réels

- **Interconnexion FAI** : échange de routes entre fournisseurs d'accès.
- **Multi-homing** : connexion d'une entreprise à plusieurs opérateurs.
- **CDN et anycast** : diffusion mondiale de contenu avec des annonces géographiques.
- **DDoS mitigation** : annonce de routes pour rediriger le trafic attaqué vers un scrubbing center.
- **Cloud on-ramp** : connexion directe à AWS, Azure, GCP via Direct Connect, ExpressRoute, Cloud Interconnect.

---

## 13. Bonnes pratiques

1. Toujours filtrer les préfixes reçus et annoncés.
2. Utiliser RPKI et BGPsec dès que possible.
3. Limiter le nombre de routes reçues par voisin.
4. Documenter chaque session avec un `description`.
5. Utiliser des communautés pour le traffic engineering.
6. Monitorer les changements de routes avec des outils comme BGPStream.
7. Configurer l'authentification TCP-AO ou MD5 sur les sessions.
8. Prévoir un plan de graceful shutdown.

---

## 14. RFCs et ressources essentielles

| RFC | Titre |
|-----|-------|
| RFC 4271 | A Border Gateway Protocol 4 (BGP-4) |
| RFC 4760 | Multiprotocol Extensions for BGP-4 |
| RFC 4272 | BGP Security Vulnerabilities Analysis |
| RFC 4456 | BGP Route Reflection |
| RFC 5065 | Autonomous System Confederations for BGP |
| RFC 6793 | BGP Support for Four-Octet AS Number Space |
| RFC 7938 | Use of BGP for Routing in Large-Scale Data Centers |
| RFC 8205 | BGPsec Protocol Specification |
| RFC 6811 | BGP Prefix Origin Validation |
| RFC 7313 | Enhanced Route Refresh Capability for BGP-4 |

---

## 15. Glossaire

| Terme | Définition |
|-------|------------|
| AS | Autonomous System — réseau autonome identifié par un numéro AS |
| NLRI | Network Layer Reachability Information — les préfixes annoncés |
| RIB | Routing Information Base — base d'informations de routage |
| FIB | Forwarding Information Base — table de transfert |
| ROA | Route Origin Authorization — objet RPKI |
| IRR | Internet Routing Registry — base de données de politiques de routage |
| Peering | Échange direct de trafic entre deux réseaux |
| Transit | Achat de connectivité vers tout Internet |

---

## 16. Conclusion

BGP est bien plus qu'un simple protocole de routage : c'est le **système nerveux de l'Internet mondial**. Sa robustesse, sa flexibilité et son évolutivité en font un outil indispensable pour les opérateurs, les entreprises et les architectes réseau. Cependant, sa sécurité reste un enjeu majeur, et l'adoption de RPKI, BGPsec et des bonnes pratiques de filtrage est essentielle pour garantir la résilience d'Internet.

---

**Créateurs originels : Kirk Lougheed & Yakov Rekhter**

**Première publication : RFC 1105, 1989**

**Version actuelle : BGP-4 (RFC 4271)**

**Licence : Protocole ouvert, standardisé par l'IETF**

---

Auteur 

HACKERS_TCHAD
