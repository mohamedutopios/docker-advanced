### 🔥 **Mise en place complète de la communication sécurisée entre `vm1` et `vm2` via Docker TLS**  
🎯 **Objectif :** Permettre à `vm2` de contrôler Docker sur `vm1` via **TLS sécurisé**.

---

## ✅ **1. Configuration de `vm1` (Docker Host)**
Nous allons configurer `vm1` pour écouter sur TCP avec TLS.

### **📌 1.1 Créer un dossier pour les certificats**
Sur **`vm1`**, exécute :
```bash
sudo mkdir -p /etc/docker/certs
cd /etc/docker/certs
```

### **📌 1.2 Générer le certificat d'autorité (CA)**
Nous allons créer un **Certificate Authority (CA)** qui va signer les certificats serveur et client.
```bash
sudo openssl genrsa -aes256 -out ca-key.pem 4096
sudo openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```
📌 **Important :** Lors de la commande `openssl req`, entre **un nom clair pour l’autorité**, comme `"Docker TLS Authority"`.

---

### **📌 1.3 Générer le certificat serveur pour `vm1`**
1️⃣ **Créer une clé privée pour le serveur (`vm1`)** :
```bash
sudo openssl genrsa -out server-key.pem 4096
```

2️⃣ **Créer une demande de signature (CSR) pour `vm1`** :
```bash
sudo openssl req -subj "/CN=192.168.33.11" -sha256 -new -key server-key.pem -out server.csr
```

3️⃣ **Signer le certificat serveur avec le CA** :
```bash
sudo openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem
```

---

### **📌 1.4 Configurer Docker sur `vm1` pour utiliser TLS**
Ajoute cette configuration dans `/etc/docker/daemon.json` :
```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2376"],
  "tls": true,
  "tlsverify": true,
  "tlscacert": "/etc/docker/certs/ca.pem",
  "tlscert": "/etc/docker/certs/server-cert.pem",
  "tlskey": "/etc/docker/certs/server-key.pem"
}
```
✅ **Redémarre Docker pour appliquer les changements :**
```bash
sudo systemctl restart docker
```
📌 **Vérifie que Docker écoute bien sur TCP/TLS :**
```bash
sudo netstat -tulnp | grep dockerd
```
📌 **Sortie attendue :**
```
tcp6       0      0 :::2376                 :::*                    LISTEN      <dockerd PID>
```

---

## ✅ **2. Génération des certificats client sur `vm1`**
Nous allons maintenant créer les certificats pour `vm2`, puis les copier sur `vm2`.

### **📌 2.1 Générer une clé privée pour `vm2`**
```bash
sudo openssl genrsa -out client-key.pem 4096
```

### **📌 2.2 Créer une demande de signature (CSR)**
```bash
sudo openssl req -subj '/CN=client' -new -key client-key.pem -out client.csr
```

### **📌 2.3 Signer le certificat client avec le CA**
```bash
sudo openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem
```

### **📌 2.4 Vérifier que la clé privée correspond au certificat client**
```bash
openssl rsa -noout -modulus -in client-key.pem | openssl md5
openssl x509 -noout -modulus -in client-cert.pem | openssl md5
```
📌 **Les deux valeurs MD5 doivent être identiques !**

---

## ✅ **3. Copier les certificats sur `vm2`**
Sur **`vm1`**, copie les fichiers sur **`vm2`** :
```bash
scp /etc/docker/certs/ca.pem vagrant@192.168.33.12:~
scp /etc/docker/certs/client-key.pem vagrant@192.168.33.12:~
scp /etc/docker/certs/client-cert.pem vagrant@192.168.33.12:~
```


En cas de probleme : 

Sur **`vm2`**, place les fichiers au bon endroit :
```bash
sudo mkdir -p /etc/docker/certs
sudo mv ~/ca.pem ~/client-key.pem ~/client-cert.pem /etc/docker/certs/
```

### **📌 3.1 Vérifier et appliquer les bonnes permissions**
```bash
sudo chmod 0400 /etc/docker/certs/client-key.pem
sudo chmod 0444 /etc/docker/certs/client-cert.pem /etc/docker/certs/ca.pem
```

---

## ✅ **4. Tester la connexion TLS depuis `vm2`**
### **📌 4.1 Définir les variables d'environnement**
Sur **`vm2`**, exécute :
```bash
export DOCKER_HOST="tcp://192.168.33.11:2376"
export DOCKER_TLS_VERIFY=1
export DOCKER_CERT_PATH="/etc/docker/certs"
```
### **📌 4.2 Tester `docker info`**
```bash
docker info
```
📌 **Si tout fonctionne, tu verras les informations de Docker distant (`vm1`) 🎉.**

---

## ✅ **5. Résolution des erreurs fréquentes**
| **Erreur** | **Solution** |
|------------|-------------|
| `tls: certificate required` | Vérifier `export DOCKER_CERT_PATH` et exécuter `docker --tlsverify ...` |
| `tls: private key does not match public key` | Régénérer `client-key.pem` et `client-cert.pem` sur `vm1` |
| `docker: command not found` | Installer Docker sur `vm2` |
| `scp: Permission denied` | Vérifier SSH et ajouter la clé publique dans `~/.ssh/authorized_keys` |

---

## ✅ **6. Bonus : Utiliser `docker context` pour simplifier la connexion**
Tu peux créer un **contexte Docker sécurisé** pour éviter d'exporter les variables à chaque session.

### **📌 6.1 Créer un `docker context`**
Sur **`vm2`**, exécute :
```bash
docker context create remote \
  --docker "host=tcp://192.168.33.11:2376,ca=/etc/docker/certs/ca.pem,cert=/etc/docker/certs/client-cert.pem,key=/etc/docker/certs/client-key.pem"
```

### **📌 6.2 Utiliser le contexte `remote`**
```bash
docker context use remote
```

### **📌 6.3 Tester sans exporter les variables**
```bash
docker info
docker ps
docker run -d --name test-container alpine sleep 3600
```
📌 **Si ça fonctionne, tu peux gérer Docker à distance plus facilement.**

---

## 🎯 **Résumé des étapes**
| **Étape** | **Commandes principales** |
|-----------|----------------------------|
| **Créer un CA sur `vm1`** | `openssl genrsa -aes256 -out ca-key.pem 4096` |
| **Créer un certificat serveur sur `vm1`** | `openssl genrsa -out server-key.pem 4096` |
| **Configurer `daemon.json` sur `vm1`** | `"tls": true, "tlsverify": true, "hosts": ["tcp://0.0.0.0:2376"]` |
| **Créer un certificat client sur `vm1`** | `openssl genrsa -out client-key.pem 4096` |
| **Copier les certificats vers `vm2`** | `scp ca.pem client-key.pem client-cert.pem vagrant@192.168.33.12:~` |
| **Configurer `vm2` pour TLS** | `export DOCKER_HOST="tcp://192.168.33.11:2376"` |
| **Tester `docker info`** | `docker info` |
| **Créer un contexte Docker** | `docker context create remote ...` |

🚀 **Félicitations, `vm2` peut maintenant gérer Docker sur `vm1` en toute sécurité via TLS !** 🎉  
Si tu veux plus de configurations (firewall, proxy, logs...), dis-moi ! 😊