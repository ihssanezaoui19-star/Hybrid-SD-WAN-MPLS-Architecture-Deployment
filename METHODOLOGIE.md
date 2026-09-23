# Méthodologie — maquette WAN hybride MPLS et SD-WAN

> Projet académique réalisé en binôme dans EVE-NG. Les configurations et captures proviennent du rapport ; le document sépare étapes configurées, premiers tests ICMP et basculement encore à démontrer. Adresses et clé du laboratoire ne sont pas reprises.

## 1. Définir le besoin multi-sites

**Ce que nous cherchions :** relier un site central et trois sites distants avec deux chemins envisageables, MPLS et Internet, tout en protégeant les échanges. Le transport MPLS devait rester utile pour les flux prioritaires et la couche SD-WAN devait permettre de sélectionner un chemin selon l'état et la politique des liens.

**Pourquoi :** un second lien n'apporte rien si les routes, politiques ou tunnels empêchent son utilisation. Les coûts opérateur, SLA et performances réelles n'étaient pas étudiés dans cette maquette.

## 2. Étudier les technologies avant de construire

Le rapport compare MPLS, LDP, SD-WAN, tunnels IPsec et routage dynamique ; il décrit BGP et OSPF dans l'état de l'art. Nous avons retenu EVE-NG pour réunir routeurs, FortiGate et postes de test dans une topologie reproductible.

**Pourquoi :** séparer le transport, le routage et les politiques de sélection des liens permet de diagnostiquer une panne couche par couche. OSPF expliqué dans l'étude ne prouve pas son déploiement complet sur les équipements du projet.

## 3. Dessiner puis monter la topologie EVE-NG

**Réalisé :** représenter un hub, trois sites distants, quatre pare-feux FortiGate, quatre routeurs Cisco virtuels et des VPC. Les segments locaux, liaisons de transport et tunnels ont des fonctions distinctes dans le dessin.

**Pourquoi :** il faut savoir à quel équipement appartient chaque rôle pour comparer un trajet prévu à un trajet réellement testé. Le schéma public supprime adresses, interfaces exactes et identifiants.

![Topologie simplifiée et anonymisée](images/topologie-anonymisee.svg)

## 4. Préparer les routeurs et le transport

**Réalisé dans les extraits du rapport :** configurer les interfaces des routeurs, les éléments MPLS/LDP et des paramètres de voisinage BGP. Ces étapes établissent les bases du chemin de transport et de l'échange des préfixes dans le laboratoire.

**Pourquoi cet ordre :** si une interface ou une route manque, un tunnel IPsec et une politique SD-WAN ne peuvent pas compenser une connectivité de base absente. Le contrôle devrait porter sur l'état des interfaces, les voisins, les routes apprises et le chemin de paquets ; les captures ne montrent pas un audit complet de tous les équipements.

## 5. Préparer les politiques FortiGate et la zone SD-WAN

**Réalisé :** associer des interfaces ou tunnels à une zone SD-WAN, définir des règles de pare-feu et préparer la surveillance des liaisons. Une capture du rapport montre l'étape de configuration de la zone.

**Pourquoi :** la sélection de lien dépend de l'appartenance des membres à la zone et des politiques de circulation. Avant de modifier une règle, vérifier quels réseaux doivent communiquer ; après la modification, vérifier que les flux utiles passent toujours et que les autres ne sont pas autorisés par erreur.

![Capture recadrée de la configuration de zone SD-WAN](images/capture-zone-sdwan.png)

**Limite :** la capture ne prouve ni quelle liaison a été réellement choisie pour une application, ni un basculement automatique.

## 6. Préparer les tunnels IPsec sur les deux chemins

**Réalisé :** le rapport présente les paramètres de phase 1, phase 2 et interfaces des tunnels `VPN-INET` et `VPN-MPLS`. Les paramètres d'authentification et la clé d'essai du PDF ne sont pas repris ici.

**Pourquoi :** deux transports n'ont pas les mêmes garanties ; les tunnels servent à définir la communication inter-sites souhaitée. Il faudrait contrôler séparément l'état de chacun, les routes associées et une communication de bout en bout avant d'affirmer que les deux chemins sont opérationnels en toute circonstance.

**Précaution :** certaines propositions cryptographiques sont propres au TP et doivent être revues selon les normes en vigueur avant toute réutilisation. Le rapport contient principalement des écrans de configuration, sans mesure indépendante de stabilité de chaque tunnel.

## 7. Vérifier la connectivité qui figure dans le rapport

Les captures de la partie « Tests de connectivité » montrent des réponses ICMP entre un VPC et une passerelle, entre un VPC et un routeur, ainsi qu'entre des routeurs. Nous avons utilisé ces essais pour contrôler des segments de la topologie après les configurations.

**Ce que cela prouve :** les cibles répondent à cet instant. **Ce que cela ne prouve pas :** le chemin choisi pour chaque application, la communication entre tous les LAN distants, l'état de chaque tunnel en continu ou la qualité d'une bascule.

## 8. Définir la validation manquante pour le secours WAN

La démarche de test à compléter consiste à relever le chemin et l'application au départ, couper **un seul lien**, observer la zone SD-WAN, les routes et l'état des tunnels, mesurer l'interruption, puis restaurer le lien et vérifier le retour. Répéter pour l'autre chemin et consigner les échecs.

**Pourquoi :** une maquette capable de répondre aux ping peut encore perdre une session ou ne jamais utiliser le chemin de secours. Le rapport ne publie pas cette séquence ni de temps de convergence ; la haute disponibilité demeure donc une hypothèse d'architecture à tester.

## Résultat vérifiable

Conception d'un WAN virtuel multi-sites, premiers réglages de routage/transport, zone SD-WAN, préparation des tunnels et connectivité ICMP sur certains chemins. Les performances, le basculement et la disponibilité applicative restent à mesurer. Le PDF original, qui demeure dans ce dépôt, contient des détails de laboratoire à revoir avant diffusion.
