maproller
=========

maproller

## URL
a.tilestream.maproller.org

b.tilestream.maproller.org

c.tilestream.maproller.org

maproller.org

maproller.org/api

maproller.org/_fti

cd ~/.ssh
ssh -i id_rsa root@5.101.96.147
apt-get update

#Firewall
sudo ufw status
sudo ufw allow from 188.178.222.138 to any port 22
sudo ufw allow www
sudo ufw enable

#Pakker
apt-get install git
apt-get install nodejs
apt-get install npm
apt-get install postgresql-9.3-postgis-2.1
apt-get install postgresql-server-dev-9.3
apt-get install python-dev
apt-get install python-pip
apt-get install gdal-bin
apt-get install python-gdal
apt-get install python-mapnik

#Python moduler
sudo pip install python-dateutil
sudo pip install pytz

#Tilemill2
git clone https://github.com/mapbox/tm2.git
cd tm2
npm install
node index.js --port=80

#Font
kopier matrikel font til /usr/share/fonts/matrikel
fc-cache -f -v
List alle fonte
python -c "from mapnik import FontEngine as e;print '\n'.join(e.instance().face_names())"

#timezone
dpkg-reconfigure tzdata
vælg europe og derefter copenhagen
hvis timezone forkert i postgres:
update ejerlav set published = published - interval '6 hour';

#postgresql
#https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
sudo -i -u postgres
createdb matrikel
psql -d matrikel -c "CREATE EXTENSION postgis;"
ctrl+d logout

http://localhost:3000/style/10/530/353.png?id=tmstyle:///Users/runetvilum/kommune.tm2
ftp.retrlines('MLSD')

#Upgrade
apt-get -u upgrade

#Check for hackere
cat auth.log | grep 'Accepted'

kopier fonte til ~/.tilemill/v2/font

-- ---------------------------------------------------------------------
-- converts mapnik's !scale_denominator! param to web mercator z
CREATE OR REPLACE FUNCTION public.z(scaledenominator numeric)
 RETURNS integer
 LANGUAGE plpgsql IMMUTABLE
AS $function$
begin
    -- Don't bother if the scale is larger than ~zoom level 0
    if scaledenominator > 600000000 then
        return 0;
    end if;
    return round(log(2,559082264.028/scaledenominator));
end;
$function$;

select ST_GeomFromGeoJSON(replace(trim(value,'"'),'\','')::json->>'geometry') from "db-7fab71c84cf84353a4475542c2013c39" where id=2
#######################
# BigCouch
#######################
apt-get update
apt-get upgrade
Hent nyeste erlang 17
wget http://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb
sudo dpkg -i erlang-solutions_1.0_all.deb
apt-get update
sudo apt-get install -y build-essential erlang-base-hipe erlang-dev erlang-manpages erlang-eunit erlang-nox erlang-reltool libicu-dev libmozjs185-dev libcurl4-openssl-dev git

git clone https://github.com/rebar/rebar.git
cd rebar
./bootstrap
cp rebar /usr/local/bin/.
cd ..
git clone https://github.com/apache/couchdb.git
cd couchdb
./configure
make
make install
adduser --system \
        --home /opt/couchdb \
        --no-create-home \
        --shell /bin/bash \
        --group --gecos \
        "CouchDB Administrator" couchdb
chown -R couchdb:couchdb /opt/couchdb

#Tilføj private ip adresse:
nano /opt/couchdb/etc/vm.args
-name couchdb@10.129.173.106


nano /opt/couchdb/etc/local.ini
[admins]
admin = rutv2327

alternativ
curl -X PUT 127.0.0.1:5984/_config/admins/admin -d '"rutv2327"'



nano /etc/init/couchdb.conf
# Upstart file at /etc/init/couchdb.conf
# CouchDB
 
start on runlevel [2345]
stop on runlevel [06]

script 
    exec sudo -i -u couchdb /opt/couchdb/bin/couchdb
end script

respawn
respawn limit 10 5


#Firewall
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F
sudo iptables -A INPUT -p tcp -s 188.178.222.138 --dport 22 -i eth0 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -d 188.178.222.138 --sport 22 -o eth0 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
sudo iptables -I OUTPUT -o eth0 -d 0.0.0.0/0 -j ACCEPT
sudo iptables -I INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i eth1 -j ACCEPT
sudo iptables -A OUTPUT -o eth1 -j ACCEPT
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP
sudo apt-get install iptables-persistent
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6

#couchdb-lucene
apt-get install openjdk-7-jdk
apt-get install maven2
wget https://github.com/rnewson/couchdb-lucene/archive/v1.0.1.tar.gz
tar -zxvf v1.0.1.tar.gz
cd couchdb-lucene-1.0.1
mvn
cp /root/couchdb-lucene-1.0.1/target/couchdb-lucene-1.0.1-dist.tar.gz /opt/.
cd /opt
tar -zxvf couchdb-lucene-1.0.1-dist.tar.gz
rm couchdb-lucene-1.0.1-dist.tar.gz
chown -R couchdb:couchdb couchdb-lucene-1.0.1/

nano /opt/couchdb/etc/local.ini
[httpd_global_handlers]
_fti = {couch_httpd_proxy, handle_proxy_req, <<"http://127.0.0.1:5985">>}
[httpd]
bind_address = 0.0.0.0

nano /etc/init/couchdb-lucene.conf
# Upstart file at /etc/init/couchdb-lucene.conf
# CouchDB-Lucene
 
start on runlevel [2345]
stop on runlevel [06]

script 
    exec sudo -i -u couchdb /opt/couchdb-lucene-1.0.1/bin/run
end script

respawn
respawn limit 10 5

nano /etc/logrotate.d/couchdb-lucene
/opt/couchdb-lucene-1.0.1/logs/*.log {
       weekly
       rotate 10
       copytruncate
       delaycompress
       compress
       notifempty
       missingok
}
#put anden node ind i den første node
curl -X PUT http://127.0.0.1:5986/nodes/couchdb@10.129.173.106 -d {}

#POSTFIX
apt-get install postfix
internet site
metadatabase.dk

Postfix is now set up with a default configuration.  If you need to make 
changes, edit
/etc/postfix/main.cf (and others) as needed.  To view Postfix configuration
values, see postconf(1).

After modifying main.cf, be sure to run '/etc/init.d/postfix reload'.

#Node.js
cd /root
mkdir node
cd node
mkdir couchdb-api
cd couchdb-api
apt-get install nodejs
apt-get install nodejs-legacy
apt-get install npm
npm install express-user-couchdb
node ./node_modules/express-user-couchdb/init http://admin:rutv2327@localhost:5986/_users
cp -R ./node_modules/express-user-couchdb/node_modules/ .
nano index.js
var couchUser = require('express-user-couchdb');
var express = require('express');
var app = express();
var path = require('path');
console.log(path.join(__dirname, 'templates'));
app.use(express.bodyParser());
// Required for session storage
app.use(express.cookieParser());
app.use(express.session({
    secret: 'rapport fra stedet'
}));
app.configure(function() {
    app.use(couchUser({
        users: 'http://admin:rutv2327@localhost:5986/_users',
        email: {
            from: 'noreply@metadatabase.dk',
            service: 'SMTP',
            SMTP: {
                service: 'localhost'
            },
            templateDir: path.join(__dirname, 'templates')
        },
        app: {
            name: 'Metadatabase'
        },
        adminRoles: ['admin'],
        verify: true

        //validateUser: function(data, cb) {...}
    }));
});
var server = app.listen(3000, function() {
    console.log('Listening on port %d', server.address().port);
});

npm install -g upstarter
upstarter
indtast:
node index.js

#Tilføj admin database
curl -X PUT http://admin:rutv2327@localhost:5984/admin
curl -X PUT http://admin:rutv2327@localhost:5984/admin/_security -d '{ "admins": { "names": [ "admin" ], "roles": [  ] }, "members": { "names": [ "admin" ], "roles": [  ] }}'
curl -X PUT http://admin:rutv2327@localhost:5984/www
curl -X PUT http://admin:rutv2327@localhost:5984/www/_security -d '{ "admins": { "names": [ "admin" ], "roles": [  ] }, "members": { "names": [  ], "roles": [  ] }}'
curl -X GET http://admin:rutv2327@localhost:5984/_all_dbs
#I bibliotekt RFS2/organization
erica push deploy

#Opret administrator
curl -X POST http://api.metadatabase.dk/api/user/signup -H "Accept: application/json" -H "Content-Type: application/json" -d '{"name": "rune@addin.dk", "password": "rutv2327", "roles": [], "email": "rune@addin.dk"}'

curl -X PUT http://admin:rutv2327@localhost:5986/_users/org.couchdb.user:rune@addin.dk -H "Accept: application/json" -H "Content-Type: application/json" -d '{"name": "rune@addin.dk", "password": "rutv2327", "roles": ["sys"], "type": "user", "verified":"2014-08-13T13:37:12.721Z", "timestamp":"2014-08-13T13:37:12.721Z"}' -v

curl -X POST http://localhost:5984/_session -H "Accept: application/json" -H "Content-Type: application/json" -d '{"name": "rune@addin.dk", "password": "rutv2327"}'

#Tilføj organizations database
curl -X PUT http://admin:rutv2327@localhost:5984/organizations
curl -X PUT http://admin:rutv2327@localhost:5984/organizations/_security -d '{ "admins": { "names": [ "admin" ], "roles": [  ] }, "members": { "names": [ ], "roles": [ ] }}'



##GlusterFS
#på alle servere
sudo apt-get install glusterfs-server
#på én server, opret forbindelse til anden server
gluster peer probe 10.129.173.106
#check status fra alle servere
gluster peer status
#opret bibliotek til at dele filer
mkdir /gluster-storage
#opret forbindelse
sudo gluster volume create volume1 replica 2 transport tcp 10.129.174.73:/gluster-storage 10.129.173.106:/gluster-storage force
mkdir /mnt/gluster
#server 10.129.174.73
mount -t glusterfs 10.129.174.73:/volume1 /mnt/gluster
#server 10.129.173.106
mount -t glusterfs 10.129.173.106:/volume1 /mnt/gluster
#automount
nano /etc/fstab
#server 10.129.174.73
10.129.174.73:/volume1 /mnt/gluster glusterfs defaults,_netdev 0 0
#server 10.129.173.106
10.129.173.106:/volume1 /mnt/gluster glusterfs defaults,_netdev 0 0
#hack for automount
nano /etc/init/mounting-glusterfs.conf
exec start wait-for-state WAIT_FOR=networking WAITER=mounting-glusterfs-$MOUNTPOINT

//!sudo gluster volume start volume1

#tilestream
install node 0.8
curl https://raw.githubusercontent.com/creationix/nvm/v0.16.0/install.sh | bash
source ~/.profile
nvm install 0.8.28
git clone https://github.com/mapbox/tilestream.git
cd tilestream
npm install
nano config.json
{
  "host": ["*.tilestream.maproller.org"],
  "tilePort": 8888,
  "uiPort": 8888,
  "tiles": "/mnt/gluster/tiles",
  "subdomains": "a,b,c"
}
bemærk i haproxy skal der indsættes:
reqirep ^Host: Host:\ 127.0.0.1

upstart script:

# tilestream.conf
description "A high performance tile server and simple web viewer for MBTiles files."
env PATH=/root/.nvm/v0.8.28/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
start on stopped networking
stop on runlevel [016]
limit nofile 1000000 1000000
console log
script
  chdir /root/tilestream
  NODE_ENV=production exec node ./index.js start --config config.json
end script
respawn



########################
# HAPROXY
########################
apt-get install haproxy
nano /etc/default/haproxy
ENABLED=1
mv /etc/haproxy/haproxy.cfg{,.original}
nano /etc/haproxy/haproxy.cfg
# Licensed under the Apache License, Version 2.0 (the "License"); you may not
# use this file except in compliance with the License. You may obtain a copy of
# the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations under
# the License.

global
        maxconn 512
        spread-checks 5

defaults
        mode http
        log global
        monitor-uri /_haproxy_health_check
        option log-health-checks
        option httplog
        balance roundrobin
        option forwardfor
        option redispatch
        retries 4
        option http-server-close
        timeout client 150000
        timeout server 3600000
        timeout connect 500

        stats enable
        stats scope .
        stats uri /_stats

frontend http-in
         # This requires HAProxy 1.5.x
         # bind *:$HAPROXY_PORT
         bind *:80
         default_backend couchdbs

backend couchdbs
        server couch1 10.129.173.106:5984 check inter 5s
        server couch2 10.129.174.73:5984 check inter 5s

service haproxy start

sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F
sudo iptables -A INPUT -p tcp -s 188.178.222.138 --dport 22 -i eth0 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -d 188.178.222.138 --sport 22 -o eth0 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT
sudo iptables -I OUTPUT -o eth0 -d 0.0.0.0/0 -j ACCEPT
sudo iptables -I INPUT -i eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth1 -j ACCEPT
sudo iptables -A OUTPUT -o eth1 -j ACCEPT
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT DROP
sudo iptables -P FORWARD DROP

sudo apt-get install iptables-persistent
iptables-restore < /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6


2014-04-12 17:03:46+02:00 1250552_GML_UTM32-EUREF89.zip
^CTraceback (most recent call last):
  File "/usr/local/bin/matrikel.py", line 80, in <module>
    outLayer.CreateFeature(outFeature)
  File "/usr/lib/python2.7/dist-packages/osgeo/ogr.py", line 1248, in CreateFeature
    return _ogr.Layer_CreateFeature(self, *args)

Frederikssund metadatabase_ac4a0c05bba0e09887eebf86d0000914
Helsingør metadatabase_ac4a0c05bba0e09887eebf86d0001528
Jammerbugt metadatabase_ac4a0c05bba0e09887eebf86d00020fa
