### ✅ **Vérifier la configuration du logging dans Docker (`json-file`)**
Votre configuration dans `/etc/docker/daemon.json` :
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5"
  }
}
```
📌 **Explication :**
- **`log-driver`: `"json-file"`** → Docker utilise des fichiers JSON pour stocker les logs.
- **`max-size`: `"50m"`** → Chaque fichier de log est limité à **50 Mo**.
- **`max-file`: `"5"`** → Docker conserve **5 fichiers max**, puis supprime les plus anciens.

---

## 🔍 **1. Vérifier si Docker applique bien cette configuration**
Utilisez cette commande :
```bash
docker info | grep -i "Logging Driver"
```
📌 **Sortie attendue :**
```
Logging Driver: json-file
```
Si **ce n’est pas le cas**, redémarrez Docker :
```bash
sudo systemctl restart docker
```
Puis re-testez avec `docker info`.

---

## 🔥 **2. Vérifier les options de logs appliquées**
1️⃣ Exécutez cette commande :
```bash
docker inspect --format '{{ .HostConfig.LogConfig }}' <CONTAINER_ID>
```
Remplacez `<CONTAINER_ID>` par l’ID d’un conteneur actif.

📌 **Sortie attendue :**
```
{json-file map[max-size:50m max-file:5]}
```
Si les valeurs sont différentes, cela signifie que la configuration n’est **pas prise en compte** par ce conteneur.

---

## 🚀 **3. Tester la rotation des logs en pratique**
1️⃣ Lancez un conteneur qui génère beaucoup de logs :
```bash
docker run -d --name test-logs busybox sh -c 'while true; do echo "Test logs"; sleep 0.1; done'
```
2️⃣ Vérifiez où se trouvent les logs Docker :
```bash
docker inspect --format='{{.LogPath}}' test-logs
sudo cat $(docker inspect --format='{{.LogPath}}' test-logs)
```
Cela retourne un chemin comme :
```
/var/lib/docker/containers/<container_id>/<container_id>-json.log
```
3️⃣ Observez la taille des logs en direct :
```bash
watch -n 1 ls -lh /var/lib/docker/containers/<container_id>/
sudo watch -n 1 ls -lh /var/lib/docker/containers/d23*/
```
📌 **Au bout de 50 Mo, un nouveau fichier doit être créé**.

4️⃣ **Supprimez le conteneur après le test** :
```bash
docker rm -f test-logs
```

---

## 🎯 **Conclusion**
- ✅ Vérifiez avec `docker info | grep "Logging Driver"`.
- 🔍 Testez avec `docker inspect --format '{{ .HostConfig.LogConfig }}' <CONTAINER_ID>`.
- 🚀 Générez des logs et observez la rotation des fichiers.

💡 **Testez et dites-moi si les logs se comportent comme prévu ! 🚀**