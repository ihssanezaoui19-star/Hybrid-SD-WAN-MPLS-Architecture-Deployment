# Architecture WAN hybride MPLS et SD-WAN

**Projet académique réalisé en binôme à l'ENSA Kénitra (2024–2025)** avec Chaima El-Achouri et Ihssane Zaoui, dans le module Technologies de réseaux WAN. Maquette EVE-NG, sans déploiement en entreprise.

## Problématique

Comment préparer une migration multi-sites où le MPLS reste disponible pour les flux prioritaires tandis qu'un lien Internet et une couche SD-WAN offrent un second chemin ? Le défi est d'articuler routage, isolation des sites et tunnels, puis de vérifier la connectivité avant de conclure à une éventuelle haute disponibilité.

## Ce que nous avons fait

| Étape | Travail décrit dans le rapport | Preuve ou résultat disponible |
| --- | --- | --- |
| Concevoir | Sites hub/spoke, quatre pare-feux FortiGate, quatre routeurs virtuels Cisco et postes VPC sur EVE-NG | Topologie et plan de laboratoire |
| Configurer le transport | Interfaces, cœur MPLS/LDP et commandes BGP sur les routeurs ; zone SD-WAN sur les FortiGate | Extraits de configuration et capture de la zone SD-WAN |
| Préparer les tunnels | Configurations IPsec `VPN-INET` et `VPN-MPLS`, interfaces et phases 1/2 | Captures de paramètres ; pas de métrique de disponibilité jointe |
| Tester | Tests ICMP entre postes, passerelles et routeurs de la maquette | Captures de réponses ping dans le rapport |

![Extrait recadré de la configuration d'une zone SD-WAN](images/capture-zone-sdwan.png)

## Résultat et limites

Le rapport montre une **topologie configurée et des échanges ICMP réussis sur les segments testés**. Il décrit une politique de secours envisagée entre MPLS et Internet, mais ne fournit pas de campagne reproductible de panne/basculement, de temps de convergence ni de mesure applicative. La haute disponibilité est donc **l'objectif d'architecture**, pas une performance démontrée. OSPF est expliqué dans l'état de l'art ; les extraits disponibles ne suffisent pas à confirmer sa configuration effective sur tous les équipements.

## Documentation

- [Méthodologie pas à pas, contrôles et suites à valider](METHODOLOGIE.md)
- [Schéma simplifié de la maquette](images/topologie-anonymisee.svg)

Le rapport académique d'origine reste dans ce dépôt. Il contient un plan d'adressage et un exemple de clé de laboratoire ; les extraits de cette présentation retirent ces paramètres. **Les propositions cryptographiques de cette maquette ne sont pas un modèle de production.**
