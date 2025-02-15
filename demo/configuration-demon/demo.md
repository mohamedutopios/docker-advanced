Configurer le **démon Docker (`dockerd`)** permet de personnaliser le fonctionnement du moteur Docker selon divers besoins : sécurité, performances, réseau, stockage, logs, etc.

---

## 📌 **1. Localisation du fichier de configuration**
Le démon Docker utilise un fichier JSON pour sa configuration :
- 📍 **Emplacement par défaut** : `/etc/docker/daemon.json`
- 🔄 **Redémarrer Docker après modification** :  
  ```bash
  sudo systemctl restart docker
  ```

Si le fichier n'existe pas, vous pouvez le créer :
```bash
sudo touch /etc/docker/daemon.json
```

---

## 🔥 **2. Exemples détaillés de configurations pour `daemon.json`**

### 1️⃣ **Configurer le socket d'écoute**
Par défaut, Docker écoute sur `unix:///var/run/docker.sock`.  
👉 **Autoriser l'accès via TCP (ex. : `192.168.1.100:2375`)**
```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://192.168.1.100:2375"]
}
```
**🚨 Sécurité** : Activer TCP expose Docker à distance ! Utilisez TLS si nécessaire.

---

### 2️⃣ **Activer TLS pour sécuriser les connexions**
Utilisation de certificats pour sécuriser l'accès distant (remplacez `/path/to/certs` par vos chemins de certificats) :
```json
{
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/path/to/certs/ca.pem",
  "tlscert": "/path/to/certs/server-cert.pem",
  "tlskey": "/path/to/certs/server-key.pem"
}
```
Cela force le client Docker à utiliser **des certificats valides** pour se connecter.

---

### 3️⃣ **Configurer le stockage des images et conteneurs**
Définir un **back-end de stockage** pour Docker (`overlay2`, `btrfs`, `aufs`, `zfs`, `devicemapper`, etc.).
```json
{
  "storage-driver": "overlay2",
  "data-root": "/var/lib/docker"
}
```
📌 **`overlay2`** est recommandé pour **les noyaux Linux récents**.

---

### 4️⃣ **Limiter la consommation de ressources**
Vous pouvez restreindre l'utilisation CPU et mémoire pour tous les conteneurs en définissant des limites globales.
```json
{
  "default-shm-size": "512m",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 32000
    }
  }
}
```
- `default-shm-size`: Définit la mémoire partagée `/dev/shm` par défaut.
- `default-ulimits`: Limite le nombre de fichiers ouverts par conteneur.

---

### 5️⃣ **Configurer le système de logs**
Docker propose plusieurs pilotes de logs : `json-file`, `syslog`, `journald`, `fluentd`, `gelf`, `awslogs`, etc.

👉 **Exemple : Activer la rotation des logs avec `json-file`**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5"
  }
}
```
- **`max-size`** : Chaque fichier de log fait au maximum **50 Mo**.
- **`max-file`** : Docker garde un maximum de **5 fichiers** de logs.

👉 **Exemple : Envoyer les logs vers `syslog`**
```json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://192.168.1.50:514",
    "tag": "{{.Name}}"
  }
}
```
- **`syslog-address`** : Adresse d’un serveur **Syslog** distant.
- **`tag`** : Utilisation du **nom du conteneur** comme identifiant de log.

---

### 6️⃣ **Configurer le réseau**
Personnalisation des sous-réseaux Docker **par défaut**.
```json
{
  "bip": "192.168.1.1/24",
  "default-address-pools": [
    {
      "base": "192.168.100.0/24",
      "size": 24
    }
  ],
  "fixed-cidr": "192.168.1.128/25"
}
```
- **`bip`** : Définit l’IP de l’interface `docker0`.
- **`default-address-pools`** : Plages IP attribuées aux nouveaux réseaux.
- **`fixed-cidr`** : Restreint la plage des IP attribuées aux conteneurs.

---

### 7️⃣ **Activer l’IPv6**
Si votre réseau supporte **IPv6**, activez-le dans Docker :
```json
{
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```
- **`ipv6`** : Active IPv6 dans Docker.
- **`fixed-cidr-v6`** : Définit un sous-réseau IPv6.

---

### 8️⃣ **Configurer les règles de sécurité**
👉 **Désactiver la modification des `iptables` par Docker** (⚠️ seulement si vous gérez manuellement `iptables`) :
```json
{
  "iptables": false
}
```

👉 **Activer le mode “live restore”** (permet de redémarrer Docker sans arrêter les conteneurs) :
```json
{
  "live-restore": true
}
```

👉 **Restreindre le nombre de requêtes API** pour limiter les attaques DoS :
```json
{
  "api-cors-header": "*",
  "max-concurrent-downloads": 3,
  "max-concurrent-uploads": 2
}
```
- **`api-cors-header`** : Autorise toutes les origines CORS (`*`).
- **`max-concurrent-downloads`** : Limite à **3** téléchargements simultanés d’images.
- **`max-concurrent-uploads`** : Limite à **2** envois simultanés.

---

## 🎯 **3. Appliquer la configuration et vérifier**
Une fois le fichier `/etc/docker/daemon.json` modifié, appliquez les changements :
```bash
sudo systemctl restart docker
```

Vérifiez que la configuration est bien prise en compte :
```bash
docker info
```

---

## ✅ **4. Exemple complet de `/etc/docker/daemon.json`**
Voici un exemple combinant **plusieurs paramètres** pour une **configuration optimale** :

```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://192.168.1.100:2375"],
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/path/to/certs/ca.pem",
  "tlscert": "/path/to/certs/server-cert.pem",
  "tlskey": "/path/to/certs/server-key.pem",
  "storage-driver": "overlay2",
  "data-root": "/var/lib/docker",
  "default-shm-size": "512m",
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 32000
    }
  },
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5"
  },
  "bip": "192.168.1.1/24",
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64",
  "iptables": false,
  "live-restore": true,
  "max-concurrent-downloads": 3,
  "max-concurrent-uploads": 2
}
```

---

## 🚀 **Conclusion**
- **Sécurité** : Utilisation de TLS et de restrictions API.
- **Stockage optimisé** : `overlay2` recommandé.
- **Logs gérés efficacement** : Rotation avec `json-file` ou `syslog`.
- **Réseau personnalisé** : IPv6 activé et pool d’adresses configuré.
- **Haute disponibilité** : `live-restore` empêche l’arrêt brutal des conteneurs.

🔥 Avec cette configuration, votre Docker Daemon est **optimisé, sécurisé et prêt pour la production** ! 🚀