Voici une version améliorée de votre **Vagrantfile** où chaque VM a un **hostname unique**, en plus de son adresse IP distincte.

---

### ✅ **Corrections et améliorations**
1. **Mise à jour de la box** :  
   - ⚠️ `hashicorp/bionic64` (**Ubuntu 18.04**) est **obsolète**.  
   - ✅ Remplacé par `ubuntu/jammy64` (**Ubuntu 22.04**).
   
2. **Ajout d'un hostname unique** pour chaque VM avec `vm.hostname = "nom-hostname"`.

3. **Ajout d'un provisionnement pour appliquer le hostname** dans `/etc/hostname` et `/etc/hosts` pour éviter les conflits réseau.

---

### 📌 **Vagrantfile corrigé avec des hostnames distincts**
```ruby
Vagrant.configure("2") do |config|
  # Définition de la première VM
  config.vm.define "vm1" do |vm1|
    vm1.vm.box = "ubuntu/jammy64"  # Utilisation d'Ubuntu 22.04 (LTS)
    vm1.vm.network "private_network", ip: "192.168.33.11"
    vm1.vm.hostname = "ubuntu-vm1"  # Définition du hostname

    vm1.vm.provision "shell", inline: <<-SHELL
      echo "ubuntu-vm1" | sudo tee /etc/hostname
      echo "192.168.33.11 ubuntu-vm1" | sudo tee -a /etc/hosts
      sudo hostnamectl set-hostname ubuntu-vm1
      
      # Installation de Docker
      curl -sSLf https://get.docker.com | sh
      sudo usermod -aG docker vagrant
    SHELL
  end

  # Définition de la deuxième VM
  config.vm.define "vm2" do |vm2|
    vm2.vm.box = "ubuntu/jammy64"
    vm2.vm.network "private_network", ip: "192.168.33.12"
    vm2.vm.hostname = "ubuntu-vm2"

    vm2.vm.provision "shell", inline: <<-SHELL
      echo "ubuntu-vm2" | sudo tee /etc/hostname
      echo "192.168.33.12 ubuntu-vm2" | sudo tee -a /etc/hosts
      sudo hostnamectl set-hostname ubuntu-vm2

      # Installation de Docker
      curl -sSLf https://get.docker.com | sh
      sudo usermod -aG docker vagrant
    SHELL
  end
end
```

---

### 📌 **Explication des changements**
- ✅ **`vm1.vm.hostname = "ubuntu-vm1"`** → Définit le hostname.
- ✅ **Modification de `/etc/hostname`** → Définit le hostname au démarrage.
- ✅ **Modification de `/etc/hosts`** → Ajoute un alias pour éviter les conflits DNS.
- ✅ **Commande `hostnamectl set-hostname`** → Applique immédiatement le changement de hostname.
- ✅ **Utilisation d’Ubuntu 22.04** (`ubuntu/jammy64`) pour éviter l'erreur **"end-of-life"** d’Ubuntu 18.04.

---

### 📌 **Commandes utiles après modification**
1️⃣ **Détruire les anciennes VM (si existantes)** :
```bash
vagrant destroy -f
```

2️⃣ **Lancer les nouvelles VM avec les hostnames mis à jour** :
```bash
vagrant up
```

3️⃣ **Vérifier l'état des VM** :
```bash
vagrant status
```

4️⃣ **Se connecter à `vm1` et vérifier le hostname** :
```bash
vagrant ssh vm1
hostname  # Doit afficher "ubuntu-vm1"
```

5️⃣ **Se connecter à `vm2` et vérifier le hostname** :
```bash
vagrant ssh vm2
hostname  # Doit afficher "ubuntu-vm2"
```

---

### ✅ **Résultat attendu**
- **Deux VM Ubuntu 22.04** avec des hostnames différents (`ubuntu-vm1`, `ubuntu-vm2`).
- **Deux IP distinctes** (`192.168.33.11`, `192.168.33.12`).
- **Docker installé sur chaque VM**.
- **Connexion possible entre les VM** en utilisant leurs hostnames (`ping ubuntu-vm2` depuis `vm1`).

🚀 **Maintenant, tes VM sont bien configurées pour un réseau privé propre et fonctionnel !** 🚀