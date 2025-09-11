# # ## Installation GLPI sur CT PVE

/ / / 󰋜 S e r v i c e s G L P I g l p i
## Installation GLPI sur C T PVE
Procédure d'installation de GLPI
Bienv enue dans ce guide complet qui v ous accompagner a à tr avers l'installation étape par étape de GLPI, une solution de gestion des ser vices d' assistance et
d'inv entair e informatique. Suiv ez attentiv ement ces instructions pour déplo yer GLPI a vec succès sur v otre ser veur, simplifiant ainsi la gestion de v otre par c
informatique.
Plongeons dir ectement dans le pr ocessus, en suiv ant chaque étape pour assur er une installation pr opre et fonctionnelle de GLPI. 🌐🛠## Installation GLPI sur C T PVE
Introduction & Pr érequis
Prérequis
Pour ce guide v ous aur ez besoin d'utliser un coneteneur sous PVE. P our cela téléchar ger un template Debian 12 classique et utiliser le début du guide
sur l' Héber gement W eb a vec les C T de PVE 󰝗

Lors de la cr éation de v otre conteneur utiliser les r essour ces suiv antes :
```
bash
apt update && apt upgrade
```
mysql_secure_installation
Répondez aux questions selon les r ecommandations fournies.Dans ce tut o ce ser a la v ersion 10.0.12 qui ser a installé. 󰋼
40 Go de st ockage ▸
1 CPU ▸
2048 Mo de RAM ▸
512 Mo de SW AP ▸
Configur ation Réseau & DNS pour a voir internet au début ▸
## Installation GLPI
Mise à jour du système :
Installation des dépendances :
 apt install apache2 php mariadb-server
```
bash
apt install php-xml php-common php-json php-mysql php-mbstring php-curl php-gd php-intl php-zip php-bz2 php-imap php-ap
```
```
bash
apt install php-ldap1
```
2
3
Sécurisation de la Base de Donnéees MariaDB
Unix switch ? N ▸
Change r oot passwor d ? Y ▸

```
bash
mysql -u root -p
```
```
bash
cd /tmp
```
```
bash
tar -xzvf glpi-10.0.12.tgz -C /var/www/Remo ve anonymous users ? Y ▸
```
Dissalow r oot login r emotly ? Y ▸
Remo ve test database and access t o it ? Y ▸
Reload privileg tables now ? Y ▸
### Configuration de la Base de Données
```
bash
CREATE DATABASE companyGLPI;
```
```
bash
GRANT ALL PRIVILEGES ON companyGLPI.* TO Adminglpi@localhost IDENTIFIED BY 'Password123!';
```
```
bash
FLUSH PRIVILEGES;
```
```
bash
EXIT
```1
2
3
4
Si vous ne v ous r appelez plus de v otre User utiliser cette commande en v ous connectant en r oot.
󰝗SELECT User, Host, Password FROM mysql.user; 1
### Téléchargement & Installation de GLPI
```
bash
wget https://github.com/glpi-project/glpi/releases/download/10.0.12/glpi-10.0.12.tgz 1
```

```
bash
chown www-data:www-data /var/www/glpi/ -R
```
Et ajoutez y le contenu suiv ant.### Configuration des Permissions
### Organisation des Répertoires
```
bash
mv /var/www/glpi/config /etc/glpi
```
```
bash
mv /var/www/glpi/files /var/lib/glpi1
```
2
### Création des Répertoires de Logs
```
bash
mkdir /var/log/glpi
```
```
bash
chown www-data:www-data /var/log/glpi1
```
2
### Configuration GLPI
```
bash
nano /var/www/glpi/inc/downstream.php 1
```
<?php
define('GLPI_CONFIG_DIR', '/etc/glpi/');
if (file_exists(GLPI_CONFIG_DIR . '/local_define.php')) {
    require_once GLPI_CONFIG_DIR . '/local_define.php';
}1
2
3
4
5
```
bash
nano /etc/glpi/local_define.php 1
```

Modifier ensuite le contenu comme suit :
La fin de Balise Dir ectory doit êtr e placé juste au dessus de la dernièr e balise Vir tualHost.<?php
    define('GLPI_VAR_DIR', '/var/lib/glpi');
    define('GLPI_LOG_DIR', '/var/log/glpi');
?>1
2
3
4
### Configuration Virtual Host Apache
```
bash
cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/glpi.company.infra.conf
```
```
bash
nano /etc/apache2/sites-available/glpi.company.infra.conf1
```
2
ServerName glpi.company .infra ▸
DocumentRoot /v ar/www/glpi/public
Et ajoutez :▸
<Directory /var/www/glpi/public>
        Require all granted
        RewriteEngine On
        # Redirect all requests to GLPI router, unless file exists.
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php [QSA,L]
</Directory>1
2
3
4
5
6
7
8
9
Activation du Site & du module Apache

```
bash
nano /etc/php/8.2/apache2/php.ini
```
Dans le fichier r echer cher la ligne: session.cookie_httponly =
Et modifier la comme suit :
session.cookie_httponly = 1
```
bash
systemctl restart apache2a2ensite glpi.company.infra.conf
```
```
bash
a2dissite 000-default.conf
```
```
bash
a2enmod rewrite
```
```
bash
systemctl restart apache21
```
2
3
4
### Configuration PHP
[Recher che a vec Nano]
Pour r echer cher dans un document a vec l'éditeur nano utlisez le r accour ci CTRL + W
Et dans ce cas là r echer chez httponly󰋼
Redémarrage Apache
Inter face W eb GLPI
Installation Web Interface
Sélectionnez la langue souhaitée puis cliquez sur "OK". ▸
Acceptez la licence en cliquant sur " Accepter ". ▸
Choisissez "Installer " pour débuter le pr ocessus d'installation. ▸
Vérifiez la compatibilité de v otre envir onnement a vec l'exécution de GLPI et cliquez sur "Continuer ". ▸

Remplissez les informations suiv antes :Connexion à la Base de Données
Serveur SQL : localhost ▸
Utilisateur SQL : Adminglpi ▸
Mot de P asse SQL : P asswor d123! ▸

Test de connexion pour v alider les informations. Sélectionnez ensuite la base de données que v ous a vez cr éée pr écédemment.

"Initialisation BDD" pour pr épar er la base de données.
La base de données est initialisée a vec succès.Configuration Additionnelle

Décochez l' option "Statistiques d'usage " si v ous ne souhaitez pas par ticiper , puis cliquez sur "Continuer ". ▸

Cliquez sur "Continuer " pour ache ver l'installation.
Vous êtes maintenant pr êt à utiliser GLPI. Connectez-v ous en utilisant les identifiants suiv ants :## Finalisation de l'Installation
Première Connexion à GLPI
Nom d'utilisateur : glpi ▸
Mot de passe : glpi ▸

## Post-Installation
Passsword par défaut

Modifier les Mots de P asse par défaut de t ous les utilisateurs pr é-configur és sur GLPI
Cliquez sur l'utlisateur dans la barr e d'avertissement modifier et confirmer son nouv eau mot de passe et faites sauv egar der.
Répetez cette manipulation pour t ous les utilisateurs

Afin d' évitez une r éinstallation de notr e ser veur GLPI on v a supprimer le fichier PHP d'install
```
bash
rm /var/www/glpi/install/install.php
```
Fichier Install.php
Résultat

[Bien Joué ! ✔ ]
Félicitations ! V ous a vez maintenant configur é avec succès l'inter face web de GLPI. Commencez à explor er ses fonctionnalités pour une gestion
efficace de v otre par c informatique. 🚀🖥󰸞
[Liens 🔗 ]
https:/ /www .it-connect.fr/tag/glpi/ 󰏌
https:/ /fr .wikipedia.or g/wiki/Gestionnair e_Libr e_de_P ar c_Informatique 󰏌
https:/ /glpi-pr oject.or g/fr/ 󰏌󰋼
Propulsé par W i k i . j s

