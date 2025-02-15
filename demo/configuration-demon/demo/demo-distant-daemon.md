### ✅ **Tester l'accès à Docker via TCP (`tcp://192.168.1.100:2375`)**

#### 📌 **1. Configuration du démon Docker pour écouter sur TCP**
Dans **la VM où Docker est installé** (`vm1` ou `vm2`), modifiez le fichier de configuration :
```bash
sudo nano /etc/docker/daemon.json
```
Ajoutez :
```json
{
  "hosts": ["unix:///var/run/docker.sock", "tcp://0.0.0.0:2375"]
}
```
📌 **Explication** :
- `unix:///var/run/docker.sock` → Maintient l'accès local via le socket Unix.
- `tcp://0.0.0.0:2375` → Écoute sur **toutes les interfaces** pour permettre un accès distant.

🛑 **⚠️ Important :** Cette configuration expose Docker sans authentification. En **production**, utilisez TLS !

#### 📌 **2. Redémarrer Docker pour appliquer les changements**
```bash
sudo systemctl restart docker
```

#### 📌 **3. Vérifier que Docker écoute sur le bon port**
Exécutez :
```bash
ss -tlnp | grep dockerd
```
Sortie attendue :
```
LISTEN  0  128  0.0.0.0:2375  0.0.0.0:*  users:(("dockerd",pid,fd))
```
Si le port **2375** apparaît, Docker est bien en écoute.

---

## 🚀 **Tester l'accès distant à Docker via TCP**
Vous pouvez tester depuis **une autre machine** ou **une autre VM** (`vm3`).

### **Test 1 : Vérifier avec `curl`**
Depuis une autre VM (`vm3`), testez si Docker répond :
```bash
curl -X GET http://192.168.33.11:2375/version
```
Si Docker répond, vous verrez un JSON contenant la version du daemon :
```json
{
  "Version": "20.10.17",
  "ApiVersion": "1.41",
  ...
}
```

### **Test 2 : Utiliser Docker CLI à distance**
Sur **n'importe quelle autre VM**, définissez la variable d'environnement :
```bash
export DOCKER_HOST="tcp://192.168.33.11:2375"
```
Puis testez avec :
```bash
docker info
```
Si tout fonctionne, Docker affichera les détails du daemon distant.

### **Test 3 : Lancer un conteneur sur le daemon distant**
```bash
docker run -d --name test-nginx -p 8080:80 nginx
```
Puis vérifiez :
```bash
docker ps
```
Et testez depuis un navigateur :  
👉 **http://192.168.33.11:8080** doit afficher la page par défaut de **Nginx**.

---

## 🔥 **Sécuriser l'accès distant**
Sans sécurité, **Docker peut être attaqué** à distance ! ⚠️  
- **Désactivez le mode TCP si inutile** :  
  ```bash
  sudo nano /etc/docker/daemon.json
  ```
  Supprimez `"tcp://0.0.0.0:2375"`, puis :
  ```bash
  sudo systemctl restart docker
  ```
- **Utilisez TLS pour un accès sécurisé** (`tcp://192.168.1.100:2376` avec certificats).
- **Activez un pare-feu** (`ufw`, `iptables`) pour bloquer **les IP non autorisées**.

---

## 🎯 **Conclusion**
- 🎯 **Testez avec `curl`** : `curl http://192.168.33.11:2375/version`
- 🚀 **Utilisez `DOCKER_HOST`** : `export DOCKER_HOST="tcp://192.168.33.11:2375"`
- 🛑 **Sécurisez l'accès distant** avec **TLS ou firewall** en production.

💡 **Besoin d’aide pour sécuriser avec TLS ?** Dis-moi ! 🚀