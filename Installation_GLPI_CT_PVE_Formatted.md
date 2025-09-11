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
Unix switch : N

Change root password : Y

Remove anonymous users : Y

Disallow root login remotely : Y

Remove test database : Y

Reload privilege tables : Y


##🗄️Création de la Base de Données
```bash
mysql -u root -p
```
```bash
CREATE DATABASE companyGLPI;
GRANT ALL PRIVILEGES ON companyGLPI.* TO 'Adminglpi'@'localhost' IDENTIFIED BY 'Password123!';
FLUSH PRIVILEGES;
EXIT;

```
##⬇️ Téléchargement & Installation de GLPI
```bash
cd /tmp
```
```bash
wget https://github.com/glpi-project/glpi/releases/download/10.0.12/glpi-10.0.12.tgz
tar -xzvf glpi-10.0.12.tgz -C /var/www/
```
##Config permision
```bash
chown -R www-data:www-data /var/www/glpi/
```

##📂 Organisation des répertoires
```bash
mv /var/www/glpi/config /etc/glpi
mv /var/www/glpi/files /var/lib/glpi
```
Création des répertoires de logs
```bash
mkdir /var/log/glpi
chown www-data:www-data /var/log/glpi
```
##⚙️ Configuration de GLPI
```bash
/var/www/glpi/inc/downstream.php :
```
```bash
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
?>
```
```bash
nano /etc/glpi/local_define.php :
```

```bash
<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
?>
```
##🌍 Configuration Virtual Host Apache
```bash
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/glpi.company.infra.conf
nano /etc/apache2/sites-available/glpi.company.infra.conf
```
 Modifier ensuite le contenu comme suit :

ServerName glpi.company.infra

DocumentRoot /var/www/glpi/public

Et ajoutez :

```bash
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/glpi.company.infra.conf
nano /etc/apache2/sites-available/glpi.company.infra.conf
```
Exemple de configuration :
<VirtualHost *:80>
    ServerName glpi.company.infra
    DocumentRoot /var/www/glpi/public

    <Directory /var/www/glpi/public>
        Require all granted
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
    </Directory>
</VirtualHost>


