### 🔍 **Problème : "Permission denied" lors du `scp` de `client-key.pem`**
L'erreur vient du fait que **`client-key.pem`** a des **permissions restreintes (`chmod 0400`)**, ce qui empêche **même l’utilisateur `vagrant`** de le copier.

---

## ✅ **Solution : Modifier temporairement les permissions avant de copier**
Sur **`vm1`** (`192.168.33.11`), exécute ces commandes pour **autoriser temporairement la lecture** :
```bash
sudo chmod 644 /etc/docker/certs/client-key.pem
```

Puis, depuis **`vm2`** (`192.168.33.12`), retente la copie :
```bash
scp vagrant@192.168.33.11:/etc/docker/certs/client-key.pem .
```
Après la copie, **restaure les permissions sécurisées sur `vm1`** :
```bash
sudo chmod 0400 /etc/docker/certs/client-key.pem
```

---

## ✅ **Solution alternative : Copier en tant que root**
Si `vagrant` n’a pas les droits pour accéder aux fichiers, tu peux utiliser `scp` en `sudo` :

Sur **`vm2`**, exécute :
```bash
sudo scp vagrant@192.168.33.11:/etc/docker/certs/client-key.pem .
```
Si cela ne fonctionne pas, utilise **`sudo` côté `vm1`** pour l'envoyer :
```bash
sudo scp /etc/docker/certs/client-key.pem vagrant@192.168.33.12:~
```
Puis, sur **`vm2`**, place le fichier au bon endroit :
```bash
sudo mv ~/client-key.pem /etc/docker/certs/
sudo chmod 0400 /etc/docker/certs/client-key.pem
```

---

## 🎯 **Résumé des solutions**
| **Problème** | **Solution** |
|-------------|-------------|
| `scp: Permission denied` | **1️⃣ Modifier temporairement les permissions (`chmod 644`)** |
| `scp toujours bloqué` | **2️⃣ Utiliser `sudo scp` pour copier en tant que root** |

💡 **Teste ces solutions et dis-moi si tu arrives à copier `client-key.pem` ! 🚀**