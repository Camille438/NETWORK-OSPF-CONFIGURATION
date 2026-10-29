# Configuration OSPF et redistribution de routes statiques

## Description
Ce projet présente la configuration d'OSPF sur des routeurs Cisco, la redistribution de routes statiques dans OSPF et les commandes de vérification. Les fichiers `.pkt` ont été réalisés avec Cisco Packet Tracer.

## 1. Configuration de base d'OSPF

```
Router> enable
Router# configure terminal
Router(config)# hostname R1

Router(config)# interface g0/0
Router(config-if)# ip address 10.0.12.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface loopback 0
Router(config-if)# ip address 10.255.0.1 255.255.255.255
Router(config-if)# exit

Router(config)# router ospf 1
Router(config-router)# router-id 1.1.1.1
Router(config-router)# network 10.0.12.0 0.0.0.3 area 0
Router(config-router)# network 10.255.0.1 0.0.0.0 area 0
Router(config-router)# passive-interface g0/1
Router(config-router)# end
Router# write memory
```

- `router-id` : identifiant unique du routeur
- `network ... area` : active OSPF sur les interfaces concernées
- `passive-interface` : empêche l'envoi de Hello vers les LAN

## 2. Aires Stub et Totally Stubby

```
! Aire Stub : sur TOUS les routeurs de l'aire
Router(config-router)# area 10 stub

! Aire Totally Stubby : no-summary uniquement sur l'ABR
Router(config-router)# area 20 stub no-summary
```

## 3. Redistribution des routes statiques dans OSPF

Sur le routeur qui porte les routes statiques (ASBR) :

```
Router(config)# ip route 192.168.100.0 255.255.255.0 10.0.12.2
Router(config)# ip route 0.0.0.0 0.0.0.0 10.0.12.2

Router(config)# router ospf 1
Router(config-router)# redistribute static subnets
Router(config-router)# default-information originate
Router(config-router)# end
```

- `redistribute static subnets` : injecte les routes statiques dans OSPF (sans `subnets`, les sous-réseaux sont ignorés)
- `default-information originate` : diffuse la route par défaut aux autres routeurs
- Les routes redistribuées apparaissent avec le code **O E2** dans la table de routage

Option : choisir le type de métrique

```
Router(config-router)# redistribute static subnets metric 20 metric-type 1
```

## 4. Commandes de vérification

```
show ip ospf neighbor
show ip ospf interface brief
show ip protocols
show ip route
show ip route ospf
show ip ospf database
show running-config | section ospf
ping 10.255.0.1
traceroute 10.255.0.1
```

| Commande | Rôle |
|----------|------|
| `show ip ospf neighbor` | Vérifie les adjacences (état FULL) |
| `show ip ospf interface brief` | Affiche l'aire et le coût de chaque interface |
| `show ip route ospf` | Affiche les routes apprises par OSPF |
| `show ip ospf database` | Affiche la LSDB (base de données d'état de liens) |

## 5. Résultats attendus
- Toutes les adjacences sont à l'état **FULL**
- Les routes inter-aires apparaissent en **O IA**
- Les routes statiques redistribuées apparaissent en **O E2**
- La route par défaut apparaît en **O*E2** (ou **O*IA** dans une aire stub)
- Les pings entre les LAN réussissent

## Fichiers du dépôt
- `4_labs_OSPF_avances_non_corriges.pdf` : sujet des labs
- `PROJET LAB 1.pkt` : Lab 1 (Packet Tracer)
- `PROJET LAB 2.pkt` : Lab 2 (Packet Tracer)

## Outils
Cisco Packet Tracer, Cisco IOS, OSPF

## Auteur
Camille438
