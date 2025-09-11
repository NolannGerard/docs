# Installation de GLPI sur un Conteneur Proxmox VE (PVE)

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
<Directory /var/www/glpi/public>
    Require all granted
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ index.php [QSA,L]
</Directory>
```
La fin de Balise Directory doit être placé juste au dessus de la dernière balise VirtualHost.

##Activation du Site & du module Apache
```bash
a2ensite glpi.company.infra.conf
a2dissite 000-default.conf
a2enmod rewrite
systemctl restart apache2
```

## Configuration PHP
nano /etc/php/8.2/apache2/php.ini

Dans le fichier rechercher la ligne: session.cookie_httponly =

Et modifier la comme suit :

session.cookie_httponly = 1

##Redémarrage Apache
systemctl restart apache2
##Interface Web GLPI
#Installation Web Interface
Sélectionnez la langue souhaitée puis cliquez sur "OK".

Acceptez la licence en cliquant sur "Accepter".

Choisissez "Installer" pour débuter le processus d'installation.

Vérifiez la compatibilité de votre environnement avec l'exécution de GLPI et cliquez sur "Continuer".

#Connexion à la Base de Données
Remplissez les informations suivantes :

Serveur SQL : localhost
Utilisateur SQL : Adminglpi
Mot de Passe SQL : Password123!
