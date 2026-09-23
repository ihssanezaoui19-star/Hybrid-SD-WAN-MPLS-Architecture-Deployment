# Méthodologie — WAN hybride MPLS et SD-WAN

> Travail académique en binôme dans EVE-NG. Ce document distingue les configurations vérifiées des objectifs. Les adresses et secrets de la maquette ne sont pas reproduits.

## 1. Poser la question et le périmètre

Une interconnexion dépendante d'un seul transport peut perdre l'accès aux ressources d'un autre site lors d'une panne. Le projet étudie une migration vers deux chemins, MPLS et Internet, avec sélection via FortiGate et tunnels IPsec. Le périmètre est virtuel ; les coûts opérateur et garanties de service ne sont pas mesurés.

## 2. Étudier les composants

Le rapport examine MPLS (transport et LDP), SD-WAN (membres, politiques, état des liens), BGP et OSPF, puis retient EVE-NG pour simuler les sites. Une notion expliquée dans l'étude ne signifie pas que chaque commande a été vérifiée en fonctionnement.

## 3. Dessiner le hub et les spokes

Le hub et trois sites distants sont représentés avec quatre FortiGate, quatre routeurs vIOS et des VPC. Les transports sont distingués des réseaux locaux et des tunnels. Le schéma public retire interfaces et adresses exactes.

![Topologie simplifiée et anonymisée](images/topologie-anonymisee.svg)

## 4. Configurer le transport sur les routeurs

Les captures montrent la configuration d'interfaces, de commandes MPLS/LDP et de voisinage BGP. Le contrôle attendu est que les interfaces et les routes nécessaires soient présentes avant d'ajouter les tunnels. Le rapport décrit OSPF dans l'état de l'art, sans démontrer son déploiement complet sur la topologie.

## 5. Définir zones et règles FortiGate

La configuration associe des interfaces à une zone SD-WAN, prépare des critères de santé de lien et définit des politiques pare-feu. La capture confirme une **étape de configuration**, pas un basculement réussi.

![Capture recadrée de la zone SD-WAN](images/capture-zone-sdwan.png)

Avant chaque changement, examiner les routes, l'ordre des règles et les accès autorisés : une règle trop large expose les postes ; une route erronée rompt la connectivité. Après application, contrôler le chemin et l'accès aux autres sites.

## 6. Préparer les tunnels IPsec

Les dernières sections du rapport décrivent `VPN-INET` et `VPN-MPLS`, leurs paramètres de phase 1, phase 2 et interfaces. Les valeurs d'authentification et secrets d'origine sont exclus. Certains choix cryptographiques sont propres à l'exercice et exigent une revue avant tout autre usage.

Pour valider chaque tunnel, il faudrait contrôler explicitement son état, les routes échangées et le trafic de bout en bout. Le rapport donne surtout des écrans de *configuration*, sans mesure indépendante de stabilité des deux tunnels.

## 7. Exécuter les tests documentés

La partie « Tests de connectivité » présente des réponses ICMP pour un VPC vers une passerelle, un VPC vers un routeur et des échanges entre routeurs. Elles prouvent l'accessibilité des cibles testées à cet instant, sans démontrer le chemin d'une application entre tous les LAN distants.

## 8. Examiner le basculement sans surestimer le résultat

Le secours MPLS/Internet est un **objectif**. Pour le valider, il resterait à enregistrer le chemin initial, couper un seul lien dans la maquette, observer le membre SD-WAN et les routes, mesurer l'interruption applicative, restaurer le lien et vérifier le retour. Le rapport ne contient pas ces mesures : aucune durée de convergence ou de disponibilité n'est revendiquée.

## Résultat pour un recruteur

Le travail illustre conception multi-sites, configuration de transport/routage, zone SD-WAN, tunnels IPsec de laboratoire et premiers tests de connectivité. La suite est une matrice de tests inter-sites et de panne reproductible, avec captures des routes, de l'état des tunnels et mesures chiffrées.
