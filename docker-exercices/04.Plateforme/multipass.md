Dans cet exercice vous allez utiliser [Multipass](https://multipass.run) pour créer une machine virtuelle et installer Docker sur celle-ci.

## Quelques mots sur Multipass

Multipass est un utilitaire développé par Canonical, il permet de lancer des machines virtuelles Ubuntu facilement.

En fonction de l'OS, Multipass peut utiliser différents hyperviseurs:
- Hyper-V
- HyperKit
- KVM
- VirtualBox

En s'intégrant de manière native à ces hyperviseurs, il permet de démarrer des machines virtuelles très rapidement.

## Installation

Vous trouverez sur le site [https://multipass.run](https://multipass.run) la procédure d'installation de Multipass en fonction de votre OS.

![Multipass](./images/local/multipass.png)

## Commandes disponibles

La liste des commandes disponibles pour la gestion du cycle de vie des VMs peut être obtenue avec la commande suivante:

```
multipass --help
```

Nous verrons quelques unes de ces commandes dans la suite de cet exercice.

## Creation d'une machine virtuelle

La commande suivante permet de lancer une nouvelle machine virtuelle nommé *docker*

```
multipass launch -n docker
```

Pour obtenir des informations sur cette VM, il suffit d'utiliser la commande suivante:

```
multipass info docker
```

Vous obtiendrez un résultat similaire à celui ci-dessous:

```
Name:           docker
State:          Running
IPv4:           192.168.64.93
Release:        Ubuntu 20.04.2 LTS
Image hash:     04aabbbd5964 (Ubuntu 20.04 LTS)
Load:           0.08 0.16 0.08
Disk usage:     1.3G out of 4.7G
Memory usage:   138.9M out of 981.3M
```

Il est également possible de lister les VM existantes:

```
multipass list
```

## Installation de Docker

A l'aide de la commande suivante, lancez un shell dans la VM créée précédemment:

```
multipass shell docker
```

On obtient alors un shell avec l'utilisateur *ubuntu* qui est notamment dans le groupe *sudo*. Depuis ce shell, nous pouvons installer Docker avec la commande suivante:

```
curl -sSL https://get.docker.com | sh
```

Toujours depuis ce shell, ajoutez ensuite l'utilisateur *ubuntu* dans le groupe *docker* créé lors de l'installation. Cela permettra à cet utilisateur de communiquer avec le daemon Docker sans passer par l'utilisation de *sudo*.

```
sudo usermod -aG docker ubuntu
```

Note: il est nécessaire de sortir du shell et d'en lancer un nouveau pour que cela soit actif.

Vérifiez ensuite la version de Docker qui a été installé:

```
docker version
```

Multipass est un utilitaire très pratique pour manipuler des VMs en local, je vous conseille de le manipuler afin de voir les différentes fonctionnalités qu'il propose.

## Cleanup

La VM créée précedemment peut être supprimer facilement en utilisant la commande suivante:

```
multipass delete -p docker
```