### 🔍 **Problème : La clé SSH (`id_rsa.pub`) n'a pas été générée**
L'erreur signifie que la commande `ssh-keygen` n'a pas généré les fichiers nécessaires (`~/.ssh/id_rsa` et `~/.ssh/id_rsa.pub`).

---

## ✅ **1. Générer la clé SSH manuellement sur chaque VM**
Sur chaque VM (`vm1`, `vm2`, `vm3`), exécute :
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```
📌 **Cela crée :**
- **`~/.ssh/id_rsa`** (Clé privée, ne pas la partager)
- **`~/.ssh/id_rsa.pub`** (Clé publique, à copier sur les autres machines)

Vérifie la création de la clé :
```bash
ls -l ~/.ssh/
```
Si les fichiers `id_rsa` et `id_rsa.pub` sont bien là, passe à l’étape suivante.

---

## ✅ **2. Copier la clé publique manuellement sur les autres machines**
Nous allons maintenant copier la clé publique de chaque machine vers les autres.

### 📌 **Depuis `vm1`, copier sa clé publique vers `vm2` et `vm3`**
1️⃣ **Récupérer la clé publique sur `vm1`** :
```bash
cat ~/.ssh/id_rsa.pub
```
📌 **Copie la sortie (exemple) :**
```
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA7v9... vagrant@ubuntu-vm1
```

2️⃣ **Ajoute cette clé sur `vm2` et `vm3`** :
Sur **`vm2`** :
```bash
vagrant ssh vm2
nano ~/.ssh/authorized_keys
```
📌 **Colle la clé de `vm1` à la fin du fichier**, puis **sauvegarde (`CTRL+X`, `Y`, `ENTER`)**.

Sur **`vm3`** :
```bash
vagrant ssh vm3
nano ~/.ssh/authorized_keys
```
📌 **Colle la clé de `vm1` à la fin du fichier**, puis **sauvegarde (`CTRL+X`, `Y`, `ENTER`)**.

3️⃣ **Applique les permissions correctes sur `vm2` et `vm3`** :
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
sudo systemctl restart ssh
```

---

### 📌 **Depuis `vm2`, copier sa clé publique vers `vm1` et `vm3`**
1️⃣ **Récupérer la clé publique sur `vm2`** :
```bash
cat ~/.ssh/id_rsa.pub
```
📌 **Copie la sortie et ajoute-la manuellement sur `vm1` et `vm3`** :
Sur **`vm1`** :
```bash
vagrant ssh vm1
nano ~/.ssh/authorized_keys
```
Colle la clé de `vm2` à la fin.

Sur **`vm3`** :
```bash
vagrant ssh vm3
nano ~/.ssh/authorized_keys
```
Colle la clé de `vm2` à la fin.

3️⃣ **Applique les permissions sur `vm1` et `vm3`** :
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
sudo systemctl restart ssh
```

---

### 📌 **Depuis `vm3`, copier sa clé publique vers `vm1` et `vm2`**
1️⃣ **Récupérer la clé publique sur `vm3`** :
```bash
cat ~/.ssh/id_rsa.pub
```
📌 **Copie la sortie et ajoute-la manuellement sur `vm1` et `vm2`** :
Sur **`vm1`** :
```bash
vagrant ssh vm1
nano ~/.ssh/authorized_keys
```
Colle la clé de `vm3` à la fin.

Sur **`vm2`** :
```bash
vagrant ssh vm2
nano ~/.ssh/authorized_keys
```
Colle la clé de `vm3` à la fin.

3️⃣ **Applique les permissions sur `vm1` et `vm2`** :
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
sudo systemctl restart ssh
```

---

## ✅ **3. Tester les connexions SSH entre les machines**
### 📌 **Depuis `vm1`, tester la connexion SSH sans mot de passe**
```bash
ssh vagrant@192.168.33.12  # Connexion à vm2
ssh vagrant@192.168.33.13  # Connexion à vm3
```
### 📌 **Depuis `vm2`, tester la connexion SSH sans mot de passe**
```bash
ssh vagrant@192.168.33.11  # Connexion à vm1
ssh vagrant@192.168.33.13  # Connexion à vm3
```
### 📌 **Depuis `vm3`, tester la connexion SSH sans mot de passe**
```bash
ssh vagrant@192.168.33.11  # Connexion à vm1
ssh vagrant@192.168.33.12  # Connexion à vm2
```
📌 **Si tout fonctionne, c'est bon ! 🎉**

---

## ✅ **4. Automatiser la génération et copie des clés SSH dans le `Vagrantfile`**
Ajoute ce script au `Vagrantfile` pour que **chaque VM génère et copie automatiquement sa clé publique vers les autres** :
```ruby
      vm.vm.provision "shell", inline: <<-SHELL
        # Génération de la clé SSH si elle n'existe pas
        if [ ! -f ~/.ssh/id_rsa ]; then
          ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
        fi

        # Ajouter la clé publique dans authorized_keys
        cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
        chmod 600 ~/.ssh/authorized_keys
        chmod 700 ~/.ssh

        # Copier la clé publique sur les autres VMs
        for target in #{vms.keys.join(" ")}; do
          target_ip=#{vms['vm1']} # Valeur temporaire
          case $target in
            vm1) target_ip=#{vms['vm1']} ;;
            vm2) target_ip=#{vms['vm2']} ;;
            vm3) target_ip=#{vms['vm3']} ;;
          esac
          if [ "$target_ip" != "#{ip}" ]; then
            ssh-keyscan -H $target_ip >> ~/.ssh/known_hosts
            cat ~/.ssh/id_rsa.pub | ssh vagrant@$target_ip "cat >> ~/.ssh/authorized_keys"
          fi
        done
      SHELL
```
Ensuite, **applique les changements** :
```bash
vagrant reload --provision
```

---

## 🎯 **Résumé des étapes**
1. ✅ **Générer une clé SSH sur chaque machine** (`ssh-keygen -t rsa -b 4096`).
2. 📋 **Copier la clé publique de chaque machine et l'ajouter dans `~/.ssh/authorized_keys` des autres**.
3. 🔒 **Appliquer les permissions correctes (`chmod 600 ~/.ssh/authorized_keys`)**.
4. 🚀 **Tester `ssh vagrant@192.168.33.X` depuis chaque machine**.
5. 🔄 **Automatiser avec le `Vagrantfile` et `vagrant reload --provision`**.

💡 **Essaie ces solutions et dis-moi si SSH fonctionne maintenant ! 🚀**