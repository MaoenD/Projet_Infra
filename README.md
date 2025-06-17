
# Rapport Technique – Infrastructure d’un Mini Extranet hébergé sur un VDS Contabo

## 1. Documentation d’Architecture

### 1.1 Présentation générale de l’infrastructure

Dans le cadre de la mise en place d’un mini extranet sécurisé à but pédagogique ou professionnel, une infrastructure virtualisée a été déployée sur un serveur VDS fourni par **Contabo**. Cette infrastructure repose sur des technologies open-source robustes et économiquement viables, avec un accent particulier mis sur la **sécurité**, la **centralisation des journaux**, la **modularité** des services et la **gestion du réseau**.

### 1.2 Composition de l’infrastructure

#### Matériel hôte :
- **Fournisseur** : Contabo (VDS)
- **OS hyperviseur** : Proxmox VE (présumé pour la gestion de `vmbr`)
- **Nombre de ponts réseau (bridges)** :
  - `vmbr0` – Interface WAN connectée à Internet
  - `vmbr1` – Interface DMZ (prévue pour extensions futures)
  - `vmbr2` – Interface LAN, interconnexion des services internes

#### Machines virtuelles déployées :

| Nom machine        | Rôle                  | OS         | Adresse IP LAN           | Remarques                        |
|--------------------|------------------------|------------|---------------------------|----------------------------------|
| **pfSense**        | Pare-feu, VPN, proxy, portail captif | pfSense 2.7.x | 192.168.1.1 (LAN) / 192.168.100.2 (WAN) | Accès Internet via NAT |
| **User**           | Poste utilisateur      | Debian     | 192.168.1.107 (DHCP)      | Aucun service, accès via Captive Portal |
| **Command Center** | Poste d'administration | Kali Linux | 192.168.1.100             | Accès complet (Wazuh, pfSense, Snort) |
| **Webmail**        | Service de messagerie  | Ubuntu     | 192.168.1.106             | Cowmail en HTTP |
| **Wazuh/Snort**    | SIEM + IDS             | Ubuntu     | 192.168.1.200             | Accès restreint, centralisation des logs |
| **Save**            | Sauvegarde et restauration| Ubuntu | 192.168.1.50              | Accès restreint et connexion SSH au webmail |

![image](https://github.com/user-attachments/assets/ae9eb749-5016-48a6-9e15-8a7a2243f366)

### 1.3 Architecture réseau logique

- Le **LAN** (vmbr2) regroupe l’ensemble des machines internes.
- La **DMZ** (vmbr1) est réservée pour de futurs services exposés (reverse proxy, services publics).
- Le **WAN** (vmbr0) est relié à l’IP publique du VDS. Le **pfSense** assure la sortie Internet via NAT avec la passerelle 192.168.100.1.

🔒 **Routage / NAT :**  
Un mappage NAT depuis l’IP publique du VDS vers l’interface WAN de pfSense (192.168.100.2) est réalisé pour assurer la connectivité extérieure.

### 1.4 Services & Protocoles déployés

| Service             | Localisation       | Description |
|---------------------|--------------------|-------------|
| **DHCP**            | pfSense            | Attribue les adresses dynamiques sur le LAN |
| **SSH (port 2222)** | Toutes les machines| Pour l’accès distant sécurisé |
| **VPN (OpenVPN)**   | pfSense            | Authentification par user/mot de passe, accès au réseau LAN via tunnel |
| **Captive Portal**  | pfSense            | Redirection automatique pour authentification web (sauf serveurs et admin) |
| **Proxy Squid**     | pfSense            | Proxy transparent, aucune règle spécifique actuellement |
| **Cowmail**         | Webmail            | Serveur webmail interne, HTTP uniquement |
| **Wazuh**           | Wazuh/Snort        | Centralisation et visualisation des logs de sécurité |
| **Snort**           | pfSense (module)   | IDS temps réel, alertes remontées dans Wazuh |

### 1.5 Choix technologiques et justification

| Outil       | Justification |
|-------------|----------------|
| **pfSense** | Interface web intuitive, fonctionnalités avancées (VPN, Captive Portal, Proxy), open-source et bien documenté |
| **Wazuh**   | Centralisation des logs, tableau de bord graphique, intégration facile avec agents |
| **Snort**   | Puissance de détection IDS, intégration pfSense native |
| **Cowmail** | Léger, suffisant pour une messagerie interne sans domaine |
| **OpenVPN** | Support natif pfSense, facile à configurer, sécurisé |
| **Kali Linux** | Distribution idéale pour les tests d’intrusion et l’administration |

📝 **Absence de HTTPS pour le webmail :**  
En raison de l’absence de nom de domaine (et du coût associé), aucun certificat SSL n’a pu être généré. Le service reste accessible uniquement sur le réseau interne, limitant l’exposition au risque.

## 2. Documentation d’Exploitation

### 2.1 Utilisation du VPN

- Le fichier `.ovpn` est généré depuis l’interface pfSense
- Chaque utilisateur est identifié par un **login/mot de passe**
- Permet un accès complet aux ressources internes via tunnel sécurisé

```
openvpn --config fichier_client.ovpn
```

### 2.2 Utilisation de Wazuh / Snort

- Accès à Wazuh uniquement depuis le **Command Center**
- Tous les agents internes sont connectés (sauf pfSense)
- Les logs par défaut sont collectés (connexion SSH, sudo, accès réseau, erreurs, etc.)

```
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.12.0-1_amd64.deb && sudo WAZUH_MANAGER='192.168.1.200' dpkg -i ./wazuh-agent_4.12.0-1_amd64.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### 2.3 Configuration du proxy Squid

- Proxy déployé mais actuellement **sans règles de filtrage**
- À terme, peut permettre :
  - Blocage de sites
  - Caching HTTP
  - Suivi des connexions

### 2.4 Portail Captif

- Redirection automatique dès la connexion réseau
- Page personnalisée avec champs d’authentification
- Bypass des serveurs internes via MAC ou IP whitelist

### 2.5 Webmail (Cowmail)

- Interface web simple à l’URL : `http://webmail.projet.local`
- Aucune restriction d’accès sur le LAN
- Accessible également via VPN

### 2.6 Sauvegarde / restauration de Cowmail

**Sauvegarde :**
```
/usr/local/bin/backup_mailcow.sh

```
Automatisée via crontab tout les jours à 2h du matin
![image](https://github.com/user-attachments/assets/37d2fb6e-aaed-4821-9f59-9723ea79d678)

**Restauration :**
```
/usr/local/bin/restore_mailcow.sh
```

## 3. Répartition des Tâches

| Membre     | Tâches réalisées |
|------------|------------------|
| **Gabriel** | Déploiement, configuration et personnalisation du Captive Portal |
| **Guillaume** | Installation des VM, configuration réseau, déploiement des services (VPN, proxy, webmail, Wazuh, Snort), configuration NAT et filtrage, automatisation de la sauvegarde et restauration, documentation complète |

## 4. Bonnes pratiques mises en œuvre

- Séparation des interfaces réseau (WAN, LAN, DMZ)
- Restriction d’accès stricte aux services critiques (Snort, Wazuh, pfSense, backup)
- Journalisation centralisée de tous les événements sur wazuh
- Utilisation de ports non standards (SSH sur 2222)
- Accès à distance uniquement via VPN sécurisé
- Utilisation de noms locaux (`.local`) pour limiter l’exposition
- Préparation d’un plan d’extension via la DMZ
- Mise en place d'une sauvegarde tout les jours à 2h du matin
