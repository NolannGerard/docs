Installation de GLPI sur un Conteneur Proxmox VE (PVE)

Bienvenue dans ce guide d’installation de GLPI sur un conteneur Debian 12 sous Proxmox VE.
Ce tutoriel vous accompagne étape par étape pour mettre en place GLPI, un gestionnaire de parc informatique et de helpdesk.

📌 Prérequis

Proxmox VE avec un conteneur Debian 12

Ressources minimales :

40 Go de stockage

1 CPU

2 Go de RAM

512 Mo de swap

Accès internet configuré (réseau & DNS)

⚙️ Mise à jour du système
apt update && apt upgrade -y

📦 Installation des dépendances
apt install apache2 php mariadb-server -y
apt install php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-ldap -y

🔐 Sécurisation de MariaDB
mysql_secure_installation


Change root password : Y

Remove anonymous users : Y

Disallow root login remotely : Y

Remove test database : Y

Reload privilege tables : Y

🗄️ Création de la Base de Données
CREATE DATABASE companyGLPI;
GRANT ALL PRIVILEGES ON companyGLPI.* TO 'Adminglpi'@'localhost' IDENTIFIED BY 'Password123!';
FLUSH PRIVILEGES;
EXIT;

⬇️ Téléchargement & Installation de GLPI
cd /tmp
wget https://github.com/glpi-project/glpi/releases/download/10.0.12/glpi-10.0.12.tgz
tar -xzvf glpi-10.0.12.tgz -C /var/www/
chown -R www-data:www-data /var/www/glpi/

📂 Organisation des répertoires
mv /var/www/glpi/config /etc/glpi
mv /var/www/glpi/files /var/lib/glpi
mkdir /var/log/glpi
chown www-data:www-data /var/log/glpi

⚙️ Configuration de GLPI

/var/www/glpi/inc/downstream.php :

<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}
?>


/etc/glpi/local_define.php :

<?php
define('GLPI_VAR_DIR', '/var/lib/glpi');
define('GLPI_LOG_DIR', '/var/log/glpi');
?>

🌍 Configuration Virtual Host Apache
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/glpi.company.infra.conf
nano /etc/apache2/sites-available/glpi.company.infra.conf


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

⚡ Activation du site
nano /etc/php/8.2/apache2/php.ini
# Modifier : session.cookie_httponly = 1

a2ensite glpi.company.infra.conf
a2dissite 000-default.conf
a2enmod rewrite
systemctl restart apache2

🖥️ Installation via l’interface Web

Sélectionnez la langue → OK

Acceptez la licence → Accepter

Choisissez Installer

Vérifiez la compatibilité → Continuer

Configurez la base SQL :

Serveur : localhost

Utilisateur : Adminglpi

Mot de passe : Password123!

Base : companyGLPI

🔑 Première Connexion

Nom d’utilisateur : glpi

Mot de passe : glpi

⚠️ Changez immédiatement les mots de passe des utilisateurs par défaut.

🧹 Nettoyage

Supprimez le fichier d’installation :

rm /var/www/glpi/install/install.php

🎉 Félicitations !

GLPI est maintenant installé et fonctionnel 🚀
Vous pouvez commencer à gérer votre parc informatique via l’interface web.

🔗 Liens utiles

GLPI Project

Documentation officielle

IT-Connect (tutoriels GLPI)
