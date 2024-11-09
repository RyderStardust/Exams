# Fiche 1 : Systèmes Distribués

## Concepts et notions de base

**Définition d'un système distribué** :
<br>
Un système distribué consiste en un réseau de plusieurs ordinateurs (noeuds) interconnectés qui agissent ensemble pour atteindre un objectif commun.

Les noeuds partagent des ressources et communiquent via un réseau pour répartir la charge de travail et augmenter la tolérance aux pannes.


**Caractéristiques essentielles** :
- **Évolutivité** : Capacité à ajouter des noeuds pour gérer plus de traffic ou de données, souvent en augmentant les ressources horizontalement (ajout de nouveaux serveurs).

- **Tolérance aux pannes** : Utilisation de la réplication et de la redondance pour permettre au système de rester fonctionnel même si un ou plusieurs noeuds tombent en panne.

- **Cohérence des données** : Maintien de la synchronisation entre les copies de données dans un système où chaque noeud peut stocker une version des données.

- **Latence et bande passante** : Considérations de vitesse de transfert et délais de réponse entre les noeuds; importantes pour choisir le protocole de communication.



## Modèles d'architecture en systèmes distribués

- **Client-Serveur** : Un modèle où un serveur centralisé fournit des services ou des ressources aux clients.

- **Peer-to-peer** : Modèle décentralisé où tous les noeuds sont égaux, chaque noeud servant à la fois de client et de serveur.

- **Microservices** : Une application est divisée en plusieurs petits services indépendants, chacun gérant une fonctionnalité précise.



## Communication inter-noeuds : protocoles et types

- **TCP** : Protocole fiable qui garantit l'arrivée et l'ordre des paquets ; approprié pour des applications nécessitant des communications exactes et ordonnées.

- **UDP** : Protocole plus rapide mais moins fiable, utilisé pour des applications où quelques pertes de paquets sont acceptables.

- **gRPC** : Protocole utilisant HTTP/2 pour des communications efficaces entre services, souvent pour des microservices, et supportant les appels de procédure distante (RPC) avec des données structurées.



## Tolérance aux pannes et cohérence des données

- **Tolérance aux pannes** : Les systèmes distribués utilisent la redondance pour garantir que les données et services restent disponibles même en cas de panne.

**Modèles de cohérence des données** :

- **Cohérence forte** : Les clients voient les mises à jour instantanément après qu'elles ont été effectuées.

- **Cohérence éventuelle** : Les clients voient les mises à jour après un certain délai, généralement quand le système a eu le temps de synchroniser les données.
