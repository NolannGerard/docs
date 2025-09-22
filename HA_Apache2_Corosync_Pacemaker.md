HA Site Web sur Apache2 avec Corosync & Pacemaker ¶ Introduction ¶
Installation & Configuration Dans un premier temps il est nécéssaire
d'installer les paquets requis. Nous allons utiliser corosync, pacemaker
et crmsh

apt update apt install corosync pacemaker crmsh Avant tout on va générer
la clé qui permetra à nos node de communiquer entre eux :

corosync-keygen Cette clé est sauvegarder ici /etc/corosync/authkey.
Dans notre cas nous allons cloner notre VM un peu plus tard donc la clé
sera présente mais dans d'autres types de configuration il vous sera
peut-être nécéssaire de la déposer sur vos serveurs.

Ensuite pour configurer tout ça il est préférable de sauvegarder d'abord
la configuration par défaut si on commet une erreur.

mv /etc/corosync/corosync.conf /etc/corosync/corosync.conf.ori Désormais
on créer notre fichier de configuration :

nano /etc/corosync/corosync.conf Et on le remplie avec la configuration
suivante :

totem { version: 2 cluster_name: cluster_web crypto_cipher: aes256
crypto_hash: sha1 clear_node_high_bit:yes } logging { fileline: off
to_logfile: yes logfile: /var/log/corosync/corosync.log to_syslog: no
debug: off timestamp: on logger_subsys { subsys: QUORUM debug: off } }
quorum { provider: corosync_votequorum expected_votes: 2 two_nodes: 1 }
nodelist { node { name: serv1 nodeid: 1 ring0_addr: 172.16.0.10 } node {
name: serv2 nodeid: 2 ring0_addr: 172.16.0.11 } } service { ver: 0 name:
pacemaker } Copy Dans ce fichier de conf il faut adapter la partie
nodelist en remplaçant les noms des serveurs et leurs IP associés

nodelist { node { name: SRV-WEB3 nodeid: 1 ring0_addr: 172.16.0.15 }
node { name: SRV-WEB4 nodeid: 2 ring0_addr: 172.16.0.20 } } Copy Pour
vérifier votre configuration vous pouvez utilisez cette comande :

corosync-cfgtool -s Maintenant on va allez un peu plus loin en utilisant
la commande :

crm_verify -L -V Elle va vous renvoyer une erreur mais pas de panique on
va résoudre ça !

crm configure property stonith-enabled=false On refait la vérification :

crm_verify -L -V Voilà l'erreur est réglé.

Maintenant on va donner comme instruction le fait d'ignorer le fait
qu'il faut que plus de 60% des nodes du cluster soit UP.

Car dans notre cas on a uniquement 2 Serveurs.

crm configure property no-quorum-policy="ignore" Après ça on peut faire
un crm configure show pour vérifier que tout est bien pris en compte

Maintenant on peut donc cloner notre VM. On change son nom et son IP par
rapport avec ce qu'on avait prévu dans notre fichier de conf

Désormais en faisant la commande crm_mon ou crm status vous devriez voir
que les deux serveurs son ONLINE !

¶ Configuration des ressources « IPFailover » et « serviceWeb » On va
créer une IP Virtuelle qui sera attribué à notre groupe de serveur. En
fait l'utilisateur utilisera cette IP pour accéder à notre site et
derrière en fonction de la disponnibilité ce sera soit le SRV-WEB3 ou le
SRV-WEB4 qui répondra.

Commande pour l'IP virtuelle : crm configure primitive IPFailover
ocf:heartbeat:IPaddr2 params ip=172.16.0.50 cidr_netmask=24 nic=ens33
iflabel=VIP

Dans votre cas adaptez les infos suivantes :

ip=XXX.XXX.XXX.XXX cidr_netmask=XX nic=XXXXX Si tout c'est bien passé
lorsque vous faites un crm_mon vous devez voir qu'une nouvelle ressource
à été ajouté.

Compléter votre vérification en faisant un ip a

failovervip.png

Maintenant on va défifinir le serveur principal de notre cluster. Dès
qu'il sera re-disponnible tout re-basculera dessus automatiquement.

On fait donc cette commande : crm configure location Prefer-IPFailover
IPFailover INFINITY: serv1 //ou// crm resource move IPFailover SRV-WEB3

Maintenant passsons au test. On va définir le SRV-WEB3 comme DOWN donc
tout va basculer sur le SRV-WEB4.

Pour le mettre en veille on fait un crm node standby

Après un peu de temps on fait un crm status qui nous affiche le SRV-WEB3
en standby et le SRV-WEB4 en ONLINE.

Et sur le site on est sur le SRV-WEB4

websitesrvweb4test1.png

Ensuite si vous réactiver le SRV-WEB3 en faisant un crm node online le
basculement se fait pour retourner sur le SRV-WEB3.

websitesrvweb4test1.png

Félicitations vous avez réussi à configurer le Failover IP !

Désormais on va lier avec le failover la gestion du service Web Apache2

crm configure primitive serviceWeb lsb:apache2 op monitor interval=60s
op start interval=0 timeout=60s op stop interval=0 timeout=60s commit
quit Maintenant lorsque le SRV-WEB3 sera actif le service apache2 sera
actif mais sur le SRV-WEB4 il passera en inactif. Et inversement.

¶ Configuration de la réplication des bases de données SUR LE SRV-WEB3
(MAITRE)

Dans la console MariaDB :

grant replication slave on *.* to 'replicateur' @'172.16.0.20'
identified by 'Btssio2017';

Ensuite editez le fichier : /etc/mysql/maridb.conf.d/50-server.cnf

#bind address 127.0.0.1 log_error = /var/log/mysql/error.log log_bin =
/var/log/mysql/mysql-bin.log server-id = 3 expire_logs_days = 10
max_binlog_size = 100M binlog_do_db = gsb_valide Copy Puis on créer le
dossier pour stocker les logs que l'on vient d'activer :

mkdir /var/log/mysql chmod -R 777 /var/log/mysql systemctl restart
mariadb.service

SUR LE SRV-WEB4 (ESCLAVE)

On édite le même fichier : /etc/mysql/maridb.conf.d/50-server.cnf

#bind address 127.0.0.1 log_error = /var/log/mysql/error.log server-id =
4 expire_logs_days = 10 max_binlog_size = 100M master-retry-count = 20
replicate-do-db = gsb_valide Copy systemctl restart mariadb.service

SUR LE SRV-WEB3 (MAITRE)

Dans la console MariaDB : SHOW MASTER STATUS;

SUR LE SRV-WEB4 (ESCLAVE)

GRANT ALL PRIVILEGE ON *.* TO 'userGsb'@'localhost' stop slave; change
master to master_host='172.16.0.15', master_user='replicateur',
master_password='Btssio2017', master_log_file='mysql-bin.000001',
master_log_pos=328; start slave; show slave status `\G

L`{=tex}es valeurs master_log_file & master_log_pos sont définit en
fonction du résultat du SHOW MASTER STATUS qui a été fait sur le master.

¶ Validation par un test SUR LE SRV-WEB3 (MAITRE)

Faite une modification sur d'un mot de passe d'un utilisateur par
exemple : update gsb_valide.Visiteur set mdp='toto' where login='agest';

SUR LE SRV-WEB4 (ESCLAVE)

Afficher les donnnées qui ont été modfifier sur le Master pour voir si
la réplication à bien eu lieu :

sqlreplitest1.png

¶ Configuration Master to Master Pour pousser la configuration on va
faire en sorte que nos deux serveurs soit esclaves de l'autres et
maitres. Comme ça peut import où aura été fait la modification elle sera
automatiquement répliqué sur l'autre.

SUR LE SRV-WEB3

Dans la console MariaDB

GRANT ALL PRIVIVILEGES ON *.* TO 'userGsb'@'localhost' Et on modifier le
fichier de configuration : /etc/mysql/mariadb.conf.d/50-server.cnf

replicate-do-db = gsb_valide master-retry-count = 20 log-slave-updates
Copy systemctl restart mariadb

SUR LE SRV-WEB4

Dans la console MariaDB : grant replication slave on *.* to
'replicateur'@'172.16.0.15' identified by 'Btssio2017';

Et on modifie le fichier de configuration :
/etc/mysql/mariadb.conf.d/50-server.cnf

replicate-do-db = gsb_valide master-retry-count = 20 log-slave-updates
binlog_do_db = gsb_valide Copy systemctl restart mariadb

SUR LE SRV-WEB3

Dans la console MariaDB : SHOW MASTER STATUS;

SUR LE SRV-WEB4

Dans la console MariaDB : stop slave; change master to
master_host='172.16.0.15', master_user='replicateur',
master_password='Btssio2017', master_log_file='mysql-bin.000001',
master_log_pos=328; start slave; SHOW SLAVE STATUS `\G

S`{=tex}HOW MASTER STATUS;

SUR LE SRV-WEB3

Dans la console MariaDB :

stop slave; change master to master_host='172.16.0.15',
master_user='replicateur', master_password='Btssio2017',
master_log_file='mysql-bin.000001', master_log_pos=328; start slave;
SHOW SLAVE STATUS `\G

¶`{=tex} Validation par un test Refaire les test que l'on a fait pour le
Master To Esclave mais cette fois ci on peut le répéter dans les deux
sens.

SRV-WEB3 --\> SRV-WEB4 SRV-WEB4 --\> SRV-WEB3 ¶ Intégration de la
réplication SQL dans le cluster crm configure primitive serviceMySQL
ocf:heartbeat:mysql params socket=/var/run/mysqld/mysqld.sock crm
configure clone cServiceMySQL serviceMySQL crm status

crmstatusclonesql.png

Commentaires
