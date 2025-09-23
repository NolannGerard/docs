# HA Site Web sur Apache2 avec Corosync & Pacemaker

## Introduction
Nous allons mettre en place un cluster haute disponibilité (HA) basé sur :
- **Corosync** pour la communication entre nœuds,
- **Pacemaker** pour l’orchestration,
- **Apache2** pour le service web,
- **MariaDB** en réplication Master ↔ Master pour la base de données.

---

## 1. Installation & Configuration de Corosync

### Installation des paquets
```bash
apt update
apt install corosync pacemaker crmsh
```

### Génération de la clé de cluster
```bash
corosync-keygen
```
Cette clé est enregistrée dans `/etc/corosync/authkey`.  
En cas de cluster multi-serveurs, il faudra la copier manuellement sur chaque nœud.

### Sauvegarde de la configuration par défaut
```bash
mv /etc/corosync/corosync.conf /etc/corosync/corosync.conf.ori
```

### Création du fichier de configuration
```bash
nano /etc/corosync/corosync.conf
```

Exemple minimal :
```conf
totem {
    version: 2
    cluster_name: cluster_web
    crypto_cipher: aes256
    crypto_hash: sha1
    clear_node_high_bit: yes
}

logging {
    fileline: off
    to_logfile: yes
    logfile: /var/log/corosync/corosync.log
    to_syslog: no
    debug: off
    timestamp: on
    logger_subsys {
        subsys: QUORUM
        debug: off
    }
}

quorum {
    provider: corosync_votequorum
    expected_votes: 2
    two_nodes: 1
}

nodelist {
    node {
        name: SRV-WEB3
        nodeid: 1
        ring0_addr: 172.16.0.15
    }
    node {
        name: SRV-WEB4
        nodeid: 2
        ring0_addr: 172.16.0.20
    }
}

service {
    ver: 0
    name: pacemaker
}
```

---

## 2. Vérification du cluster

### Tester la configuration
```bash
corosync-cfgtool -s
crm_verify -L -V
```

En cas d’erreur **STONITH**, désactiver temporairement :
```bash
crm configure property stonith-enabled=false
```

Ignorer la règle de quorum (utile car seulement 2 nœuds) :
```bash
crm configure property no-quorum-policy="ignore"
```

Vérification :
```bash
crm_mon
```

---

## 3. Ressources Pacemaker

### IP virtuelle (Failover)
```bash
crm configure primitive IPFailover ocf:heartbeat:IPaddr2     params ip=172.16.0.50 cidr_netmask=24 nic=ens33 iflabel=VIP
```

Vérification :
```bash
ip a
crm_mon
```

### Définir un nœud préféré
```bash
crm configure location Prefer-IPFailover IPFailover INFINITY: SRV-WEB3
# ou bascule manuelle
crm resource move IPFailover SRV-WEB3
```

### Groupe Apache + IP
Pour garantir que l’IP et Apache basculent ensemble :
```bash
crm configure primitive serviceWeb lsb:apache2     op monitor interval=60s timeout=60s     op start interval=0 timeout=60s     op stop interval=0 timeout=60s

crm configure group grpWeb IPFailover serviceWeb
```

---

## 4. Réplication MariaDB

### Configuration Master (SRV-WEB3)
```sql
GRANT REPLICATION SLAVE ON *.* TO 'replicateur'@'172.16.0.20' IDENTIFIED BY 'Btssio2017';
```

Fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` :
```conf
log_error = /var/log/mysql/error.log
log_bin = /var/log/mysql/mysql-bin.log
server-id = 3
expire_logs_days = 10
max_binlog_size = 100M
binlog_do_db = gsb_valide
```

### Configuration Slave (SRV-WEB4)
```conf
log_error = /var/log/mysql/error.log
server-id = 4
expire_logs_days = 10
max_binlog_size = 100M
master-retry-count = 20
replicate-do-db = gsb_valide
```

Redémarrage :
```bash
systemctl restart mariadb
```

### Liaison Master → Slave
Sur le Master :
```sql
SHOW MASTER STATUS;
```

Sur le Slave :
```sql
STOP SLAVE;
CHANGE MASTER TO
  MASTER_HOST='172.16.0.15',
  MASTER_USER='replicateur',
  MASTER_PASSWORD='Btssio2017',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=328;
START SLAVE;
SHOW SLAVE STATUS\G
```

---

## 5. Réplication Master ↔ Master

Sur **SRV-WEB3** :
```sql
GRANT ALL PRIVILEGES ON *.* TO 'userGsb'@'localhost';
```
Dans `50-server.cnf` :
```conf
replicate-do-db = gsb_valide
master-retry-count = 20
log-slave-updates
```

Sur **SRV-WEB4** :
```sql
GRANT REPLICATION SLAVE ON *.* TO 'replicateur'@'172.16.0.15' IDENTIFIED BY 'Btssio2017';
```
Dans `50-server.cnf` :
```conf
replicate-do-db = gsb_valide
master-retry-count = 20
log-slave-updates
binlog_do_db = gsb_valide
```

Reprendre la même procédure avec `CHANGE MASTER TO` sur les deux serveurs.

---

## 6. Intégration MySQL au cluster

### Ressource Pacemaker pour MySQL
```bash
crm configure primitive serviceMySQL ocf:heartbeat:mysql     params socket=/var/run/mysqld/mysqld.sock     op monitor interval=20s timeout=10s
crm configure clone cServiceMySQL serviceMySQL
```

---

## 7. Tests de validation

### Test Failover Apache
- Mettre un nœud en standby :
  ```bash
  crm node standby SRV-WEB3
  ```
- Vérifier que l’IP et Apache passent sur SRV-WEB4 :
  ```bash
  crm status
  ```

### Test Réplication SQL
Sur SRV-WEB3 :
```sql
UPDATE gsb_valide.Visiteur SET mdp='toto' WHERE login='agest';
```

Sur SRV-WEB4 :
```sql
SELECT * FROM gsb_valide.Visiteur WHERE login='agest';
```

---

## Conclusion
Vous disposez maintenant :
- d’un **cluster HA Apache2** avec IP virtuelle,
- d’une **réplication SQL Master ↔ Master** intégrée au cluster Pacemaker.

Ce setup permet d’assurer la **disponibilité** et la **redondance** du service web et des bases de données.
