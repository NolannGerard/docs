# Installation de GLPI sur un Conteneur Proxmox VE (PVE)

## 📑 Table des Matières
- [Prérequis](#-prérequis)
- [Mise à jour du système](#️-mise-à-jour-du-système)
- [Installation des dépendances](#-installation-des-dépendances)
- [Sécurisation de MariaDB](#-sécurisation-de-mariadb)
- [Création de la Base de Données](#️-création-de-la-base-de-données)
- [Téléchargement & Installation de GLPI](#️-téléchargement--installation-de-glpi)
- [Organisation des répertoires](#-organisation-des-répertoires)
- [Configuration de GLPI](#-configuration-de-glpi)
- [Configuration Virtual Host Apache](#-configuration-virtual-host-apache)
- [Activation du site](#-activation-du-site)
- [Installation via l’interface Web](#️-installation-via-linterface-web)
- [Première Connexion](#-première-connexion)
- [Nettoyage](#-nettoyage)
- [Félicitations](#-félicitations)
- [Liens utiles](#-liens-utiles)

---

Bienvenue dans ce guide d’installation de **GLPI** sur un conteneur Debian 12 sous **Proxmox VE**.  
Ce tutoriel vous accompagne étape par étape pour mettre en place GLPI, un gestionnaire de parc informatique et de helpdesk.

---

## 📌 Prérequis

- Proxmox VE avec un conteneur Debian 12  
- Ressources minimales :
  - 40 Go de stockage
  - 1 CPU
  - 2 Go de RAM
  - 512 Mo de swap
- Accès internet configuré (réseau & DNS)

---

## ⚙️ Mise à jour du système

```bash
apt update && apt upgrade -y
```
##📦 Installation des dépendances
```bash
apt install apache2 php mariadb-server -y
apt install php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-ldap -y

```
##🔐 Sécurisation de MariaDB
```bash
mysql_secure_installation
```

### Change root password : Y

### Remove anonymous users : Y

### Disallow root login remotely : Y

### Remove test database : Y

Reload privilege tables : Y
##🗄️ Création de la Base de Données
