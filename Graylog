Installation de Graylog 6.1 sur Debian (11/12)
Ce guide couvre l’installation complète de MongoDB 6.0, OpenSearch 2.x et Graylog 6.1 sur Debian.
Tout est prêt à l'emploi en une seule page markdown.

1. Prérequis et Configuration Système
bash
apt update && apt upgrade -y
apt install -y curl lsb-release ca-certificates gnupg2 pwgen wget apt-transport-https

timedatectl set-timezone Europe/Paris

cat <<EOF > /etc/systemd/timesyncd.conf
[Time]
NTP=ntp.univ-rennes2.fr
FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org 3.debian.pool.ntp.org
EOF
timedatectl set-ntp true
systemctl restart systemd-timesyncd
timedatectl timesync-status
2. Installation de MongoDB 6.0
bash
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | gpg --dearmor -o /usr/share/keyrings/mongodb-server-6.0.gpg

echo "deb [signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg] http://repo.mongodb.org/apt/debian bullseye/mongodb-org/6.0 main" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list

wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb
dpkg -i libssl1.1_1.1.1f-1ubuntu2.24_amd64.deb

apt update && apt install -y mongodb-org

systemctl daemon-reload
systemctl enable --now mongod
systemctl status mongod --no-pager
3. Installation d’OpenSearch 2.x
bash
curl -o- https://artifacts.opensearch.org/publickeys/opensearch.pgp | gpg --dearmor --batch --yes -o /usr/share/keyrings/opensearch-keyring

echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring] https://artifacts.opensearch.org/releases/bundle/opensearch/2.x/apt stable main" | tee /etc/apt/sources.list.d/opensearch-2.x.list

apt update
env OPENSEARCH_INITIAL_ADMIN_PASSWORD=Rootsio2017! apt install -y opensearch

cat <<EOF > /etc/opensearch/opensearch.yml
cluster.name: graylog
node.name: ${HOSTNAME}
path.data: /var/lib/opensearch
path.logs: /var/log/opensearch
discovery.type: single-node
network.host: 127.0.0.1
action.auto_create_index: false
plugins.security.disabled: true
EOF

cat <<EOF > /etc/opensearch/jvm.options
-Xms2g
-Xmx2g
EOF

sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | tee -a /etc/sysctl.conf

systemctl daemon-reload
systemctl enable --now opensearch
4. Installation de Graylog 6.1
bash
wget https://packages.graylog2.org/repo/packages/graylog-6.1-repository_latest.deb
dpkg -i graylog-6.1-repository_latest.deb
apt update && apt install -y graylog-server
5. Configuration de Graylog
bash
GRAYLOG_SECRET=$(pwgen -N 1 -s 96)
GRAYLOG_ROOT_SHA2=$(echo -n "Rootsio2017" | shasum -a 256 | awk '{print $1}')

sed -i "s/^password_secret =.*/password_secret = $GRAYLOG_SECRET/" /etc/graylog/server/server.conf
sed -i "s/^root_password_sha2 =.*/root_password_sha2 = $GRAYLOG_ROOT_SHA2/" /etc/graylog/server/server.conf
sed -i "s|^#root_timezone = UTC|root_timezone = Europe/Paris|" /etc/graylog/server/server.conf
sed -i "s|^#http_bind_address = 127.0.0.1:9000|http_bind_address = 0.0.0.0:9000|" /etc/graylog/server/server.conf
sed -i "s|^#elasticsearch_hosts = http://127.0.0.1:9200|elasticsearch_hosts = http://127.0.0.1:9200|" /etc/graylog/server/server.conf
6. Lancement du service Graylog
bash
systemctl daemon-reload
systemctl enable --now graylog-server
tail -f /var/log/graylog-server/server.log
Accédez à l’interface : http://<IP_DE_VOTRE_SERVEUR>:9000
Identifiants par défaut : admin / Rootsio2017

Changelog
Corrections typo et titres (Gaylog → Graylog)

Commandes redondantes supprimées/optimisées

Remplacement des nano par du scripting automatique (cat <<EOF & sed)

Ajout de vm.max_map_count (OpenSearch)

Fix libssl1.1 pour MongoDB sur Debian 12

Configuration automatisée des secrets et des principaux paramètres (server.conf)
