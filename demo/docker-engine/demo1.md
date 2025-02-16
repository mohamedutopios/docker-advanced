# Configuration Personnalisée avec Docker Machine

Docker Machine permet de créer et configurer des hôtes Docker sur différentes plateformes, qu'elles soient locales (VirtualBox, VMware) ou distantes (AWS, Azure). Ce document propose plusieurs configurations locales avec VirtualBox, en incluant des options variées pour personnaliser les hôtes Docker.

## 1. Création d'un hôte Docker local

### 1.1. Création d'une machine virtuelle Docker sur VirtualBox
Pour créer une machine virtuelle avec VirtualBox, utilisez la commande suivante :

```sh
docker-machine create --driver virtualbox --virtualbox-no-vtx-check my-docker-host

```

- `--driver virtualbox` : Utilise VirtualBox comme hyperviseur.
- `my-docker-host` : Nom de la machine.

### 1.2. Personnalisation des ressources

Vous pouvez personnaliser les ressources CPU, mémoire et disque :

```sh
docker-machine create \
  --driver virtualbox \
  --virtualbox-cpu-count 2 \
  --virtualbox-memory 4096 \
  --virtualbox-disk-size 20000 \
  my-docker-host
```

- `--virtualbox-cpu-count 2` : Alloue 2 CPU.
- `--virtualbox-memory 4096` : Alloue 4 Go de RAM.
- `--virtualbox-disk-size 20000` : Crée un disque de 20 Go.

### 1.3. Configuration d'un réseau personnalisé

Pour attribuer une adresse IP fixe à la VM :

```sh
docker-machine create \
  --driver virtualbox \
  --virtualbox-hostonly-cidr 192.168.99.1/24 \
  --virtualbox-dhcp=false \
  --virtualbox-static-ip 192.168.99.100 \
  my-docker-host
```

- `--virtualbox-hostonly-cidr` : Définit un sous-réseau pour la machine.
- `--virtualbox-static-ip` : Spécifie une IP fixe.

### 1.4. Utilisation d'une clé SSH personnalisée

```sh
docker-machine create \
  --driver virtualbox \
  --virtualbox-ssh-key ~/.ssh/id_rsa \
  my-docker-host
```

- `--virtualbox-ssh-key` : Spécifie la clé SSH pour la connexion.

## 2. Accès et gestion des hôtes Docker

### 2.1. Liste des hôtes Docker

```sh
docker-machine ls
```
Affiche toutes les machines, leur statut, IP et driver utilisé.

### 2.2. Se connecter via SSH

```sh
docker-machine ssh my-docker-host
```
Accède à la machine via SSH.

### 2.3. Configurer le client Docker

Avant d'exécuter des commandes Docker sur l'hôte distant, configurez l'environnement :

```sh
eval $(docker-machine env my-docker-host)
```

Après cela, toutes les commandes Docker seront exécutées sur la machine distante.

## 3. Suppression et nettoyage des machines Docker

### 3.1. Supprimer un hôte Docker

```sh
docker-machine rm my-docker-host
```

### 3.2. Supprimer toutes les machines Docker

```sh
docker-machine rm $(docker-machine ls -q)
```

### 3.3. Récupérer les logs d'un hôte Docker

```sh
docker-machine logs my-docker-host
```

## Conclusion

Docker Machine offre une grande flexibilité pour créer et gérer des machines Docker localement. En utilisant VirtualBox, il est possible d'allouer des ressources, de configurer un réseau, et de sécuriser les accès via SSH. Ces options permettent de personnaliser l'environnement selon les besoins du projet.

