# Documentation Utilisateur – Mini Extranet

## 1. Connexion au réseau interne

### 1.1 Connexion au vds
- connexion via lien fourni
- Entrer les credentials founis

### 1.2 Machine et Portail
- Se connecter à la machine user
- Une page d’authentification peut s'ouvrir en cliquant sur la page du navigateur.
- Saisir vos identifiants (fournis par l’administrateur).
- Une fois authentifié, vous accédez à Internet et aux services internes autorisés.

ℹEn cas de blocage, contactez l'administrateur réseau.

### 1.3 Création d’un compte utilisateur sur le Portail Captif (CommandCenter)
1. Accéder à pfSense via l’interface web : `https://192.168.1.1`
2. Aller dans **System > User Manager**
3. Cliquer sur **Add** pour créer un nouvel utilisateur
4. Remplir les champs :
   - Username
   - Password
   - Prénom/Nom (optionnel)
5. Enregistrer. L’utilisateur pourra se connecter via le portail captif.
---

## 2. Accès via VPN (OpenVPN)

### 2.1 Prérequis
- Obtenez le fichier de configuration `.ovpn` fourni par l’administrateur.
- Installer OpenVPN sur votre machine :
  - Windows : [OpenVPN GUI](https://openvpn.net/community-downloads/)
  - Linux : `sudo apt install openvpn`
  - macOS : Tunnelblick ou OpenVPN Connect

### 2.2 Connexion VPN
```bash
openvpn --config fichier_client.ovpn
```
- Entrez votre identifiant et mot de passe VPN.
- Une fois connecté, vous accédez au réseau interne comme si vous étiez localement connecté.

### 2.3 Création d’un compte utilisateur sur le VPN (CommandCenter)
1. Accéder à pfSense via l’interface web : `https://192.168.1.1`
2. Aller dans **System > User Manager**
3. Cliquer sur **Add** pour créer un nouvel utilisateur
4. Remplir les champs :
   - Username
   - Password
   - Prénom/Nom (optionnel)
   - Certificat
5. Enregistrer.
6. Exporter le fichier de configuration et transmettre les identifiants
---

## 3. Webmail (Cowmail)

### 3.1 Accès
- Ouvrir un navigateur web.
- Entrer l’adresse : `http://webmail.local` (disponible uniquement en local ou via VPN).

### 3.2 Connexion
- Saisissez votre identifiant et mot de passe.
- Vous accédez à votre boîte de réception, pouvez envoyer et recevoir des emails internes.

### 3.3 Création d’un compte utilisateur webmail (administrateur)
1. Se connecter à la machine webmail localement
2. Ajouter un nouvel utilisateur avec une boîte mail
3. L’utilisateur peut se connecter via Cowmail avec ses identifiants.

---

## 4. Utilisation du Proxy (Squid)

### 4.1 Fonctionnement
- Le proxy est transparent : il s’active automatiquement.
- Toute navigation web passe par le proxy pour des raisons de sécurité et de journalisation.

### 4.2 Restrictions possibles
- Certaines pages ou services peuvent être restreints (filtrage à venir).
- En cas de blocage, un message d’erreur ou une page personnalisée s’affichera.

---

## 5. Supervision et Sécurité (Wazuh / PFsense)

### 5.1 Accès réservé (CommandCenter uniquement)
- Interface : https://192.168.1.200
- Login/mot de passe requis.

### 5.2 Visualisation des alertes
- Affiche les logs, connexions SSH, activités système.
- Outils de filtrage, recherche, tableaux de bord dynamiques.

### 5.3 Accès réservé PFsense (CommandCenter uniquement)
- Interface : https://192.168.1.1
- Login /mot de passe requis.
---

## 6. Sauvegarde et restauration automatisées — Mailcow

Le système met en place une stratégie de sauvegarde et de restauration. Deux scripts sont utilisés :

- `/usr/local/bin/backup_mailcow.sh` : effectue la sauvegarde
- `/usr/local/bin/restore_mailcow.sh` : gère la restauration

### 6.1 Sauvegarde quotidienne automatisée

Le script de sauvegarde est exécuté automatiquement chaque jour à **03h00** via une tâche `cron`. Pour modifier l'heure de déclenchement, éditez la crontab avec :

```bash
crontab -e
```

Et modifiez la ligne suivante selon vos besoins :

```cron
0 3 * * * /usr/local/bin/backup_mailcow.sh
```

- Rotation automatique : suppression des fichiers de plus de **2 jours** sur le serveur distant

Tous les événements sont journalisés dans :

```
/var/log/mailcow_backup.log
```

### 6.2 Personnalisation du script de sauvegarde

Le script est situé ici :
```
/usr/local/bin/backup_mailcow.sh
```

Pour ajouter d'autres fichiers ou répertoires à sauvegarder :
1. Ouvrez le script avec un éditeur :
   ```bash
   sudo nano /usr/local/bin/backup_mailcow.sh
   ```
2. Ajoutez une ligne `tar` ou `scp` avec le chemin souhaité
3. Par exemple, pour sauvegarder `/opt/certificats` :
   ```bash
   tar -czf /tmp/certs.tar.gz /opt/certificats
   scp /tmp/certs.tar.gz backup1@192.168.1.201:/srv/backup1/autres/
   ```

### 6.3 Restauration manuelle avec confirmation

Le script de restauration est à lancer manuellement :
```bash
/usr/local/bin/restore_mailcow.sh
```

Il demande une **confirmation `[y/N]`** avant de procéder

Écrit tous les journaux dans :

```
/var/log/mailcow_restore.log
```

## 7. Bonnes Pratiques Utilisateur
- Ne partagez jamais vos identifiants.
- Déconnectez-vous des sessions VPN et Webmail après usage.
- Ne tentez pas d'accéder à des services restreints.
- Contactez l'administrateur en cas de doute ou problème technique.

---
