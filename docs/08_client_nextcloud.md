# 08 - Client Nextcloud PC et Android

## Installation du client Nextcloud sur PC

### Installation

Telecharger et installer le client officiel Nextcloud pour Windows.

Lancer le client puis renseigner :

```text
URL du serveur :
https://homelab-pi.xxxxxxxxx.ts.net
```

Se connecter avec :

```text
Utilisateur : admin-cloud
Mot de passe : ********
Code 2FA (TOTP)
```

Le code TOTP n'est demande que lors de la premiere connexion ou lors d'une reconnexion importante.

Le client recoit ensuite un jeton d'authentification et ne redemande pas le code a chaque synchronisation.

---

### Choix du dossier local

Modification du dossier de synchronisation :

```text
F:\Nextcloud
```

au lieu du dossier par defaut :

```text
C:\Users\<Utilisateur>\Nextcloud
```

Avantages :

- ne pas remplir le disque systeme ;
- stockage sur un SSD dedie ;
- gestion plus simple des donnees.

---

### Mode de synchronisation

Option choisie :

```text
Synchronize everything from server
```

Fonctionnement :

```text
F:\Nextcloud
        <=>
Nextcloud (Raspberry Pi)
```

Toute modification est synchronisee dans les deux sens :

- ajout ;
- modification ;
- suppression.

---

### Fichiers internes du client

Le client cree automatiquement :

```text
.sync_xxxxx.db
.sync_xxxxx.db-wal
```

Ces fichiers servent a gerer la synchronisation.

Ne pas les supprimer.

---

### Verification du fonctionnement

Creer un fichier :

```text
F:\Nextcloud\test.txt
```

Attendre la fin de la synchronisation.

Verifier son apparition :

```text
Interface web Nextcloud
-> Fichiers
```

Le fichier doit etre visible.

Le test valide :

```text
PC -> Raspberry Pi
```

---

### Utilisation quotidienne

Pour envoyer un fichier sur le serveur :

```text
Copie / Deplacement
->
F:\Nextcloud
```

Le client l'envoie automatiquement au Raspberry Pi.

Exemples :

```text
F:\Nextcloud\Photos
F:\Nextcloud\Documents
```

Toutes les donnees sont synchronisees automatiquement.

---

## Installation de Nextcloud sur Android

### Installation

Installer l'application officielle Nextcloud depuis :

```text
Google Play Store
```

ou

```text
F-Droid
```

---

### Connexion

Renseigner :

```text
Serveur :
https://homelab-pi.xxxxxxxxx.ts.net
```

Puis :

```text
Utilisateur
Mot de passe
Code TOTP
```

---

### Fonctionnement

Contrairement au PC :

```text
Le telephone ne telecharge pas tous les fichiers.
```

L'application affiche simplement le contenu du serveur.

Exemples :

```text
Photos
Documents
Cours
Videos
```

Les fichiers restent stockes sur le Raspberry Pi.

---

### Consultation des fichiers

Lorsqu'un fichier est ouvert :

```text
Telephone
->
Telechargement temporaire
->
Ouverture
```

Le fichier n'est pas conserve definitivement sur le telephone.

Android et l'application gerent automatiquement le cache.

Aucune suppression manuelle n'est necessaire.

---

### Modification d'un fichier

Exemple :

```text
document.txt
```

Processus :

```text
Telechargement
->
Modification
->
Reenvoi automatique vers Nextcloud
```

La version du serveur est mise a jour.

---

### Sauvegarde automatique des photos

Fonction disponible dans l'application :

```text
Parametres
-> Televersement automatique
```

Exemple :

```text
DCIM/Camera
```

Fonctionnement :

```text
Photo prise
->
Televersement automatique
->
Nextcloud sur Raspberry Pi
```

Les photos sont sauvegardees automatiquement sur le serveur.

---

## Resume du fonctionnement global

PC :

```text
Synchronisation complete
F:\Nextcloud <=> Raspberry Pi
```

Telephone :

```text
Consultation a la demande
Pas de copie complete des donnees
```

Photos :

```text
Telephone -> Raspberry Pi
(sauvegarde automatique)
```

Donnees :

```text
Raspberry Pi
->
SSD NVMe chiffre (LUKS)
->
Sauvegarde NAS
```

Acces :

```text
HTTPS
+
Tailscale
+
Authentification a deux facteurs (2FA)
```
