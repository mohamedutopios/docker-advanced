# 1 - Le service Docker

## Rappels : images, conteneurs,...
- Définition d'une image Docker
- Définition d'un conteneur Docker
- Comment une image devient un conteneur

## Configurer le démon `Dockerd`
- Configuration des paramètres du démon
- Fichiers de configuration
- Options de démarrage

## Variables d'environnement : `DOCKER_HOST`
- Qu'est-ce que `DOCKER_HOST` ?
- Comment configurer `DOCKER_HOST` ?

## L'option socket pour les accès réseau
- Utilisation du socket Docker (`/var/run/docker.sock`)
- Sécurisation de l'accès au socket

# 2 - Création d'un registry privé

## Principe d'un registry
- Qu'est-ce qu'un Docker Registry ?
- Différence entre un registry public et privé

## Création d'un registry
- Étapes pour créer un registry Docker privé

## Création d'un registry sécurisé
- Utilisation du protocole HTTPS
- Générer un certificat SSL pour Docker Registry

## Création d'un registry authentifié
- Ajout d'une authentification basique
- Configurer un fichier `htpasswd`

## Configuration d'un client Docker et les certificats
- Ajouter des certificats SSL
- Configurer Docker pour se connecter à un registry privé sécurisé

# 3 - Docker Machine

## Présentation et installation
- Qu'est-ce que Docker Machine ?
- Installer Docker Machine sur différentes plateformes

## Configuration personnalisée
- Configuration des hôtes Docker sur Docker Machine
- Personnalisation des machines virtuelles Docker

## Création de machines virtuelles
- Commandes pour créer une machine virtuelle avec Docker Machine

## Utilisation de Docker-machine
- Gérer des hôtes Docker à l'aide de Docker Machine
- Commandes utiles pour Docker Machine

# 4 - Fonctionnalités avancées du réseau et du stockage

## Rappels du réseau Docker
- Introduction au réseau Docker
- Types de réseaux Docker : bridge, host, overlay, etc.

## Fonctionnalités avancées : bridge, overlay, host, macvlan
- Explication des réseaux Docker : 
  - `bridge`
  - `overlay`
  - `host`
  - `macvlan`

## Rappel des volumes Docker
- Qu'est-ce qu'un volume Docker ?
- Types de volumes : locaux, partagés, et en réseau

## Les volumes distribués et les plugins
- Utilisation de volumes distribués : NFS, GlusterFS, etc.
- Installer et configurer des plugins de stockage

## Stockage et conteneurs : gestion des ressources
- Gestion des ressources (CPU, mémoire) dans Docker
- Utilisation des volumes pour le stockage persistant

# 5 - Docker Swarm

## Architecture de Swarm
- Présentation de Docker Swarm
- Rôles des nœuds : manager et worker

## Installation du cluster et administration des nœuds
- Étapes d'installation d'un cluster Docker Swarm
- Ajouter et gérer des nœuds dans le cluster

## L'option `cluster-advertise`
- Utilisation de l'option `--cluster-advertise` dans Swarm

## Le principe des services globaux et répartis
- Différences entre services globaux et répartis
- Comment utiliser ces services dans un cluster Swarm

## Administration des services Docker Swarm
- Créer, configurer et gérer des services dans Swarm

## Scalabilité et load balancing
- Gestion de la scalabilité avec Docker Swarm
- Répartition de la charge (load balancing) dans Swarm

## Les labels, les contraintes et les préférences
- Utilisation des labels et contraintes dans Swarm
- Personnaliser les services avec des préférences

## Réseaux et volumes
- Configuration avancée des réseaux et volumes dans Docker Swarm

## Rappels de Docker Compose
- Introduction à Docker Compose pour gérer des applications multi-conteneurs

## Administration des Stacks
- Déployer et administrer des stacks dans Swarm avec Docker Compose

## Personnalisation d'un réseau overlay
- Personnaliser un réseau overlay dans Docker Swarm

## Réseau et load balancing
- Optimiser le réseau et le load balancing dans Docker Swarm

## Les mises à jour et rollback
- Gérer les mises à jour et faire des rollback dans Swarm

## La sécurité - autolock
- Configurer l'option `autolock` pour sécuriser les nœuds Swarm

## Maintenance d'un nœud du cluster
- Planifier et effectuer la maintenance d'un nœud dans Swarm

## Sauvegarde et restauration d'un cluster
- Sauvegarder et restaurer un cluster Docker Swarm

# 6 - La sécurité Docker

## Principes : analyse des risques, les types de dangers, les mécanismes de protection
- Comprendre les risques et les protections dans Docker

## Sécuriser l'hôte
- Sécuriser l'hôte Docker (pare-feu, mises à jour, etc.)

## Sécuriser Docker Engine
- Configurer Docker Engine pour une meilleure sécurité

## Sécuriser les images Notary
- Utilisation de Docker Content Trust et Notary pour sécuriser les images

## Sécuriser l'application Dockérisée
- Sécuriser les applications exécutées dans des conteneurs

## Sécuriser le cluster Swarm
- Sécuriser un cluster Docker Swarm

## L'isolation avec les namespaces
- Utilisation des namespaces pour isoler les conteneurs

## Limiter les ressources avec les Cgroups
- Gestion des ressources avec les Cgroups pour améliorer la sécurité

## Administrer les Docker Secrets
- Stocker et gérer des informations sensibles avec Docker Secrets

# 7 - Les logs et le monitoring Docker

## La gestion des logs
- Gérer et configurer les logs dans Docker

## Audits (Docker Bench, ...)
- Utilisation de Docker Bench pour la sécurité
- Autres outils d'audit de sécurité pour Docker

## Outils de supervision : Prometheus, ...
- Configuration de la supervision avec Prometheus et Grafana
