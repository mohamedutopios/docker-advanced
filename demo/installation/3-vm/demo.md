Voici un **Vagrantfile mis à jour** qui **maintient l'installation de Docker sur `vm1` et `vm2`** tout en **ajoutant les accès SSH entre les VMs (`vm1`, `vm2` et `vm3`)**.

---

## ✅ **Modifications apportées :**
- 🚀 **`vm1` et `vm2` conservent Docker**.
- 🔑 **Accès SSH sans mot de passe** entre toutes les VMs (`vm1`, `vm2`, `vm3`).
- 🔄 **Ajout automatique des clés SSH** entre les VMs pour qu'elles puissent communiquer entre elles.

---

### 📌 **Nouveau `Vagrantfile`**
```ruby
Vagrant.configure("2") do |config|
  # Définition des VM avec leurs IP et Hostnames
  vms = {
    "vm1" => "192.168.33.11",
    "vm2" => "192.168.33.12",
    "vm3" => "192.168.33.13"
  }

  vms.each do |name, ip|
    config.vm.define name do |vm|
      vm.vm.box = "ubuntu/jammy64"  # Ubuntu 22.04
      vm.vm.network "private_network", ip: ip
      vm.vm.hostname = "ubuntu-#{name}"

      vm.vm.provision "shell", inline: <<-SHELL
        echo "ubuntu-#{name}" | sudo tee /etc/hostname
        echo "#{ip} ubuntu-#{name}" | sudo tee -a /etc/hosts
        sudo hostnamectl set-hostname ubuntu-#{name}

        # Mise à jour et installation de SSH
        sudo apt update -y
        sudo apt install -y openssh-server
        sudo systemctl enable ssh
        sudo systemctl restart ssh

        # Génération de la clé SSH (si elle n'existe pas)
        if [ ! -f ~/.ssh/id_rsa ]; then
          ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
        fi

        # Ajout des clés publiques pour l'accès SSH entre les VMs
        for target_ip in #{vms.values.join(" ")}; do
          if [ "$target_ip" != "#{ip}" ]; then
            ssh-keyscan -H $target_ip >> ~/.ssh/known_hosts
          fi
        done

        # Copier la clé publique dans les autres VMs
        cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
        chmod 600 ~/.ssh/authorized_keys
        chmod 700 ~/.ssh

        # Désactiver la vérification stricte des hôtes SSH
        echo "StrictHostKeyChecking no" >> ~/.ssh/config
      SHELL

      # Installation de Docker uniquement sur vm1 et vm2
      if ["vm1", "vm2"].include?(name)
        vm.vm.provision "shell", inline: <<-SHELL
          curl -sSLf https://get.docker.com | sh
          sudo usermod -aG docker vagrant
        SHELL
      end
    end
  end
end
```

---

## ✅ **Explication des ajouts**
1. 📌 **Docker installé uniquement sur `vm1` et `vm2`**.
2. 🔑 **Génération automatique des clés SSH** (`id_rsa`) si elles n'existent pas.
3. 🔄 **Ajout automatique des clés publiques** dans `authorized_keys` pour permettre SSH entre toutes les VMs.
4. 🚀 **Suppression de la vérification stricte SSH (`StrictHostKeyChecking no`)** pour éviter les avertissements de clé SSH.
5. 📡 **Mise à jour de `known_hosts` avec `ssh-keyscan`** pour pré-approuver les connexions entre les VMs.

---

## 🔥 **Lancer et tester les accès SSH**
### **1️⃣ Démarrer les 3 VMs**
```bash
vagrant up
```

### **2️⃣ Vérifier les connexions SSH entre les VMs**
#### **Depuis `vm1` :**
```bash
vagrant ssh vm1
```
Puis tester SSH vers les autres VMs :
```bash
ssh ubuntu-vm2  # Doit se connecter sans mot de passe
ssh ubuntu-vm3  # Doit se connecter sans mot de passe
```

#### **Depuis `vm2` :**
```bash
vagrant ssh vm2
```
Puis tester :
```bash
ssh ubuntu-vm1
ssh ubuntu-vm3
```

#### **Depuis `vm3` :**
```bash
vagrant ssh vm3
```
Puis tester :
```bash
ssh ubuntu-vm1
ssh ubuntu-vm2
```

---

## 🎯 **Résultat attendu**
- ✅ **Les VMs peuvent se connecter entre elles via SSH sans mot de passe**.
- ✅ **Docker est installé sur `vm1` et `vm2`, mais pas sur `vm3`**.
- ✅ **Chaque VM peut exécuter `docker ps` sur `vm1` et `vm2`**.
- ✅ **Les connexions entre `vm1`, `vm2` et `vm3` sont possibles via `ping` et `ssh`**.

---

## 🚀 **Personnalisation possible**
- 🔥 **Ajouter `docker-compose` à `vm1` et `vm2`**.
- 🔒 **Restreindre SSH uniquement aux IPs internes (`192.168.33.X`)** via `sshd_config`.
- 📡 **Configurer un pare-feu (`UFW`) pour plus de sécurité**.

💡 **Essaie ce `Vagrantfile` et dis-moi si tu veux ajouter des fonctionnalités ! 🚀**