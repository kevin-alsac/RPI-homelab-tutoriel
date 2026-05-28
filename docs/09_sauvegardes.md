# Sauvegarde automatique Nextcloud vers NAS avec Cron

## Objectif

Mettre à jour automatiquement le NAS à partir des données Nextcloud stockées sur le Raspberry Pi.

Fonctionnement :

```text
Nextcloud (Pi)
        ↓
Montage temporaire du NAS
        ↓
Synchronisation rsync
        ↓
Création d'un log
        ↓
Démontage du NAS
```

Le NAS n'est monté que pendant la sauvegarde.

---

# Vérification du service Cron

Vérifier que le service est actif :

```bash
sudo systemctl status cron
```

Résultat attendu :

```text
active (running)
```

---

# Création du script de sauvegarde

Créer le dossier :

```bash
mkdir -p ~/scripts
```

Créer le script :

```bash
nano ~/scripts/backup-nas.sh
```

Contenu :

```bash
#!/bin/bash

DATE=$(date '+%Y-%m-%d_%H-%M-%S')

MOUNTPOINT="/mnt/backup-nas"
LOGDIR="/home/pi_admin/logs"
LOGFILE="$LOGDIR/backup-$DATE.log"

mkdir -p "$LOGDIR"
mkdir -p "$MOUNTPOINT"

echo "=== Début sauvegarde $(date) ===" >> "$LOGFILE"

# Démontage automatique à la fin
trap 'umount "$MOUNTPOINT" 2>/dev/null' EXIT

# Montage du NAS (IP FIXE)
mount -t cifs //192.168.1.69/data "$MOUNTPOINT" \
-o credentials=/root/.nas-credentials

if [ $? -ne 0 ]; then
    echo "ERREUR : impossible de monter le NAS" >> "$LOGFILE"
    exit 1
fi

echo "NAS monté avec succès" >> "$LOGFILE"

# Synchronisation Nextcloud -> NAS
rsync -a --delete \
"/srv/secure-data/docker-data/nextcloud/data/admin-cloud/files/" \
"$MOUNTPOINT"/ \
>> "$LOGFILE" 2>&1

if [ $? -eq 0 ]; then
    echo "Sauvegarde terminée avec succès" >> "$LOGFILE"
else
    echo "ERREUR : échec de la sauvegarde" >> "$LOGFILE"
fi

echo "=== Fin sauvegarde $(date) ===" >> "$LOGFILE"
```

Enregistrer :

```text
CTRL+O
Entrée
CTRL+X
```

Rendre le script exécutable :

```bash
chmod +x ~/scripts/backup-nas.sh
```

---

# Fichier d'identification NAS

Créer :

```bash
sudo nano /root/.nas-credentials
```

Contenu :

```text
username=UTILISATEUR_NAS
password=MOT_DE_PASSE_NAS
```

Sécuriser :

```bash
sudo chmod 600 /root/.nas-credentials
```

---

# Test manuel du script

Lancer :

```bash
sudo ~/scripts/backup-nas.sh
```

Vérifier les logs :

```bash
ls -lt ~/logs
```

Afficher le dernier log :

```bash
tail -50 $(ls -t ~/logs/*.log | head -1)
```

Résultat attendu :

```text
NAS monté avec succès
Sauvegarde terminée avec succès
=== Fin sauvegarde ...
```

---

# Création de la tâche Cron

Éditer la crontab root :

```bash
sudo crontab -e
```

Choisir :

```text
1
```

pour Nano si demandé.

Ajouter :

```cron
0 3 * * 6 /home/pi_admin/scripts/backup-nas.sh
```

Signification :

```text
0   = minute
3   = heure
*   = chaque jour du mois
*   = chaque mois
6   = samedi
```

Exécution :

```text
Tous les samedis à 03h00
```

Enregistrer :

```text
CTRL+O
Entrée
CTRL+X
```

---

# Vérifier la tâche enregistrée

Afficher la crontab :

```bash
sudo crontab -l
```

Résultat attendu :

```cron
0 3 * * 6 /home/pi_admin/scripts/backup-nas.sh
```

---

# Test Cron sans attendre le samedi

Modifier temporairement :

```bash
sudo crontab -e
```

Remplacer par :

```cron
*/5 * * * * /home/pi_admin/scripts/backup-nas.sh
```

Exécution :

```text
Toutes les 5 minutes
```

Attendre quelques minutes puis vérifier :

```bash
ls -lt ~/logs
```

Une fois le test terminé, remettre :

```cron
0 3 * * 6 /home/pi_admin/scripts/backup-nas.sh
```

---

# Désactiver temporairement la tâche

Éditer :

```bash
sudo crontab -e
```

Commenter la ligne :

```cron
# 0 3 * * 6 /home/pi_admin/scripts/backup-nas.sh
```

Cron ignorera cette ligne.

---

# Vérifications utiles

Voir les tâches root :

```bash
sudo crontab -l
```

Voir les montages actifs :

```bash
mount | grep backup-nas
```

Voir les processus rsync :

```bash
ps aux | grep rsync
```

Suivre un log en direct :

```bash
tail -f $(ls -t ~/logs/*.log | head -1)
```

Arrêter une synchronisation :

```bash
sudo pkill rsync
```

Démonter le NAS manuellement :

```bash
sudo umount /mnt/backup-nas
```

---

# Résultat final

Chaque samedi à 03h00 :

```text
Raspberry Pi
    ↓
Montage SMB du NAS
    ↓
Rsync des dossiers Nextcloud
    ↓
Création d'un log horodaté
    ↓
Démontage du NAS
```

Les dossiers synchronisés sont :

```text
Dossier Partagé
ileane
kevin
```

Le NAS conserve une copie miroir de ces dossiers grâce à :

```bash
rsync -a --delete
```
