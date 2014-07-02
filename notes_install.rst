.. geOrchestra documentation master file, created by
   sphinx-quickstart on Fri Mar 28 10:58:25 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Configuration des serveurs
**************************

Logiciels prérequis
===================
Avant de commencer avec les logiciels requis, quelques logiciels facultatifs pour le confort de travail:

* Openssh pour travailler avec putty
* Samba pour partager les dossiers avec le poste de travail
* Postfix comme serveur mail

.. code-block :: bash

   apt-get install openssh-server samba postfix
   
Pour pouvoir compiler depuis les sources

.. code-block :: bash

   apt-get install build-essential python-virtualenv python-dev

   
Configuration PostgreSQL/PostGIS
================================
PostgreSQL est déjà installé sur un serveur distant. Il ne reste plus qu'à créer la base qui accueillera nos données. On peut le faire à partir du client PGAdmin. PostGIS est déjà installé et configuré.

Attention la base geofence n'est pas deployée ici.

* Se connecter avec  un utilisateur ayant les droits d'ecriture
* Création d'une base "georchestra" avec comme template celui livré à l'installation de PostGIS
* Execution des requêtes SQL du dépot GitHub geOrchestra(Pour le code complet à executer, voir :doc:`Création de la base PostgreSQL </code>`)
* Création d'une base "geonetwork" avec comme template celui livré à l'installation de PostGIS

LDAP (Serveur 1)
================
Installation

.. code-block :: bash

   apt-get install slapd ldap-utils git-core
   cd /home/user/download
   git clone git://github.com/georchestra/LDAP.git
   cd LDAP

Configuration

.. code-block :: bash

   ldapadd -Y EXTERNAL -H ldapi:/// -f georchestra-bootstrap.ldif
   ldapadd -D"cn=admin,dc=georchestra,dc=org" -W -f georchestra-root.ldif
   ldapadd -D"cn=admin,dc=georchestra,dc=org" -W -f georchestra.ldif

APACHE 2 (Serveur 2)
====================

Pour le serveur 2 on installe simplement apache ::

   apt-get install apache2
   
APACHE 2 (Serveur 1)
====================  
   
Installation d'apache et configuration du site georchestra. 

.. code-block :: bash

   apt-get install apache2
   a2enmod proxy_ajp proxy_connect proxy_http proxy ssl rewrite headers
   service apache2 graceful
   
   #Création du fichier de configuration du nouveau site
   cd /etc/apache2/sites-available
   a2dissite default default-ssl #Desactivation des sites par defaut
   touch georchestra  #Creation du Virtual host de notre site
 
Modification du fichier créé ::
 
   <VirtualHost *:80>
     ServerName vm-georchestra
     DocumentRoot /var/www/georchestra/htdocs
     LogLevel warn
     ErrorLog /var/www/georchestra/logs/error.log
     CustomLog /var/www/georchestra/logs/access.log "combined"
     Include /var/www/georchestra/conf/*.conf
     ServerSignature Off
   </VirtualHost>
   <VirtualHost *:443>
     ServerName vm-georchestra
     DocumentRoot /var/www/georchestra/htdocs
     LogLevel warn
     ErrorLog /var/www/georchestra/logs/error.log
     CustomLog /var/www/georchestra/logs/access.log "combined"
     Include /var/www/georchestra/conf/*.conf
     SSLEngine On
     SSLCertificateFile /var/www/georchestra/ssl/georchestra.crt
     SSLCertificateKeyFile /var/www/georchestra/ssl/georchestra-unprotected.key
     SSLCACertificateFile /etc/ssl/certs/ca-certificates.crt
     ServerSignature Off
   </VirtualHost>

Configuration du dossier

.. code-block :: bash
   
   a2ensite georchestra
   cd /var/www
   mkdir georchestra
   cd georchestra
   mkdir conf htdocs logs ssl
   chgrp www-data logs/
   chmod g+w logs/

Page d'erreur

.. code-block :: bash

   mkdir -p /var/www/georchestra/htdocs/errors
   wget http://sdi.georchestra.org/errors/50x.html -O /var/www/georchestra/htdocs/errors/50x.html
   
ProxyPass (/var/www/georchestra/conf/proxypass.conf) ::   
   
   <IfModule !mod_proxy.c>
       LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so
   </IfModule>
   <IfModule !mod_proxy_http.c>
       LoadModule proxy_http_module /usr/lib/apache2/modules/mod_proxy_http.so
   </IfModule>
   
   RewriteLog /tmp/rewrite.log
   RewriteLogLevel 3
   
   SetEnv no-gzip on
   ProxyTimeout 999999999
   
   AddType application/vnd.ogc.context+xml .wmc
   
   RewriteEngine On
   RewriteRule ^/analytics$ /analytics/ [R]
   RewriteRule ^/cas$ /cas/ [R]
   RewriteRule ^/catalogapp$ /catalogapp/ [R]
   RewriteRule ^/downloadform$ /downloadform/ [R]
   RewriteRule ^/extractorapp$ /extractorapp/ [R]
   RewriteRule ^/geoserver$ /geoserver/ [R]
   RewriteRule ^/geofence$ /geofence/ [R]
   RewriteRule ^/geowebcache$ /geowebcache/ [R]
   RewriteRule ^/header$ /header/ [R]
   RewriteRule ^/ldapadmin$ /ldapadmin/ [R]
   RewriteRule ^/ldapadmin/privateui$ /ldapadmin/privateui/ [R]
   RewriteRule ^/mapfishapp$ /mapfishapp/ [R]
   RewriteRule ^/proxy$ /proxy/ [R]
   RewriteRule ^/geonetwork-private/?(.*)$ /geonetwork/$1 [R]
   
   ErrorDocument 502 /errors/50x.html
   ErrorDocument 503 /errors/50x.html
   
   ProxyPass /casfailed.jsp ajp://localhost:8009/casfailed.jsp 
   ProxyPassReverse /casfailed.jsp ajp://localhost:8009/casfailed.jsp
   
   ProxyPass /j_spring_cas_security_check ajp://localhost:8009/j_spring_cas_security_check 
   ProxyPassReverse /j_spring_cas_security_check ajp://localhost:8009/j_spring_cas_security_check
   
   ProxyPass /j_spring_security_logout ajp://localhost:8009/j_spring_security_logout 
   ProxyPassReverse /j_spring_security_logout ajp://localhost:8009/j_spring_security_logout
   
   <Proxy ajp://localhost:8009/analytics/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /analytics/ ajp://localhost:8009/analytics/ 
   ProxyPassReverse /analytics/ ajp://localhost:8009/analytics/
   
   <Proxy ajp://localhost:8009/cas/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /cas/ ajp://localhost:8009/cas/ 
   ProxyPassReverse /cas/ ajp://localhost:8009/cas/
   
   <Proxy ajp://localhost:8009/catalogapp/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /catalogapp/ ajp://localhost:8009/catalogapp/ 
   ProxyPassReverse /catalogapp/ ajp://localhost:8009/catalogapp/
   
   <Proxy ajp://localhost:8009/downloadform/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /downloadform/ ajp://localhost:8009/downloadform/ 
   ProxyPassReverse /downloadform/ ajp://localhost:8009/downloadform/
   
   <Proxy ajp://localhost:8009/extractorapp/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /extractorapp/ ajp://localhost:8009/extractorapp/ 
   ProxyPassReverse /extractorapp/ ajp://localhost:8009/extractorapp/
   
   <Proxy ajp://localhost:8009/geonetwork/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /geonetwork/ ajp://localhost:8009/geonetwork/ 
   ProxyPassReverse /geonetwork/ ajp://localhost:8009/geonetwork/
   
   <Proxy ajp://localhost:8009/geonetwork-private/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /geonetwork-private/ ajp://localhost:8009/geonetwork-private/ 
   ProxyPassReverse /geonetwork-private/ ajp://localhost:8009/geonetwork-private/
   
   <Proxy ajp://localhost:8009/geoserver/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /geoserver/ ajp://localhost:8009/geoserver/ 
   ProxyPassReverse /geoserver/ ajp://localhost:8009/geoserver/
   
   <Proxy ajp://localhost:8009/geofence/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /geofence/ ajp://localhost:8009/geofence/ 
   ProxyPassReverse /geofence/ ajp://localhost:8009/geofence/
   
   ProxyPass /geowebcache/ ajp://localhost:8009/geowebcache/ 
   ProxyPassReverse /geowebcache/ ajp://localhost:8009/geowebcache/
   
   <Proxy ajp://localhost:8009/ldapadmin/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /ldapadmin/ ajp://localhost:8009/ldapadmin/
   ProxyPassReverse /ldapadmin/ ajp://localhost:8009/ldapadmin/
   
   <Proxy ajp://localhost:8009/mapfishapp/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /mapfishapp/ ajp://localhost:8009/mapfishapp/ 
   ProxyPassReverse /mapfishapp/ ajp://localhost:8009/mapfishapp/
   
   <Proxy ajp://localhost:8009/proxy/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /proxy/ ajp://localhost:8009/proxy/ 
   ProxyPassReverse /proxy/ ajp://localhost:8009/proxy/
   
   <Proxy ajp://localhost:8009/header/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /header/ ajp://localhost:8009/header/
   ProxyPassReverse /header/ ajp://localhost:8009/header/
   
   <Proxy ajp://localhost:8009/_static/*>
       Order deny,allow
       Allow from all
   </Proxy>
   ProxyPass /_static/ ajp://localhost:8009/_static/
   ProxyPassReverse /_static/ ajp://localhost:8009/_static/

Certification SSL

.. code-block :: bash
   
   cd /var/www/georchestra/ssl
   openssl genrsa -des3 -out georchestra.key 1024
   openssl req -new -key georchestra.key -out georchestra.csr
   
Common name : vm-georchestra ::
   
   openssl rsa -in georchestra.key -out georchestra-unprotected.key
   openssl x509 -req -days 365 -in georchestra.csr -signkey georchestra.key -out georchestra.crt
   service apache2 graceful
   sudo nano /etc/hosts
   
127.0.0.01	vm-georchestra

Test :
http://vm-georchestra
https://vm-georchestra

JAVA (serveur 1&2)
==================
::

   apt-get install openjdk-7-jdk
   
TOMCAT (serveur 1&2)
====================
Le schéma de l'architecture montre que trois instances de Tomcat sont présentes. Elles seront nommées comme suit :

Serveur 1 :
   * Tomcat60 : Instance contenant le CAS et le security proxy.
   * Tomcat61 : Instance contenant toutes les webapps sauf geoserver, le CAS et le proxy.
   
Serveur 2 :
   * Tomcat62 : Instance contenant geoserver.
   * Tomcat63 : Instance contenant geoserver.
   
Utilisateur
-----------

Un utilisateur spécifique est créé pour gérer Tomcat. ::

   groupadd tomcat
   useradd -g tomcat -s /usr/sbin/nologin -m -d /home/tomcat tomcat

Cet utilisateur ne peut pas se logguer il sert uniquement à lancer Tomcat et stopper Tomcat de cette manière :

.. code-block :: bash

   cd /opt/tomcat6/tomcat6X/bin #Avec X, une instance de Tomcat
   su -p -s /bin/sh tomcat startup.sh
   su -p -s /bin/sh tomcat shutdown.sh

Il est possible de créer des services pour gérer plus facilement les instances (Voir :doc:`Créer un service tomcat </memo>`)

Installation
------------

Pour créer plusieurs instances de Tomcat, la solution la plus souple est de télécharger le tar.gz plutôt que d'utiliser un apt-get install (liens symboliques...).

.. code-block :: bash

   #Serveur 1&2
   mkdir /opt/tomcat6
   cd /opt/tomcat6
   wget http://apache.crihan.fr/dist/tomcat/tomcat-6/v6.0.39/bin/apache-tomcat-6.0.39.tar.gz
   tar -xf apache-tomcat-6.0.39.tar.gz
   
Cloner le dossier une fois pour chaque instance.

.. code-block :: bash 
 
   #serveur 1
   cp -R apache-tomcat-6.0.39 tomcat60
   mv apache-tomcat-6.0.39 tomcat61
   #serveur 2
   cp -R apache-tomcat-6.0.39 tomcat62
   mv apache-tomcat-6.0.39 tomcat63
   
   
L'utilisateur tomcat est propriétaire des trois instances.

.. code-block :: bash

   #serveur 1
   chown -R tomcat:tomcat tomcat60
   chown -R tomcat:tomcat tomcat61
   #serveur 2
   chown -R tomcat:tomcat tomcat62
   chown -R tomcat:tomcat tomcat63
   

Configuration
-------------
 
* **Variable d'environnement**
 
Les scripts startup.sh et shutdown.sh des dossiers bin doivent être modifiés. (/opt/tomcat6/tomcat6X/bin)

.. code-block :: bash

   export JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-amd64
   export PATH=$JAVA_HOME/bin:$PATH
   export BASEDIR=/opt/tomcat6/tomcat6X
   export CATALINA_BASE=/opt/tomcat6/tomcat6X
   export CATALINA_HOME=/opt/tomcat6/tomcat6X

Dans les scripts catalina.sh (/opt/tomcat6/tomcat6X/bin), rajouter : 

Tomcat60 ::
   
   JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true -Xms128m -Xmx256m -XX:MaxPermSize=256m"
   JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.trustStore=/opt/share/keystore -Djavax.net.ssl.trustStorePassword=mdp"

Tomcat61 ::

   JAVA_OPTS="$JAVA_OPTS -Djava.awt.headless=true -Xms512m -Xmx1024m -XX:MaxPermSize=256m"
   JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.trustStore=/opt/share/keystore -Djavax.net.ssl.trustStorePassword=mdp"
   #GEONETWORK
   JAVA_OPTS="$JAVA_OPTS -Dgeonetwork.dir=/home/user/geonetwork_datadir -Dgeonetwork-private.schema.dir=/opt/tomcat6/tomcat61/webapps/geonetwork-private/WEB-INF/data/config/schema_plugins -Dgeonetwork.jeeves.configuration.overrides.file=/opt/tomcat6/tomcat61/webapps/geonetwork-private/WEB-INF/config-overrides-georchestra.xml"
   #EXTRACTORAPP
   JAVA_OPTS="$JAVA_OPTS -Dorg.geotools.referencing.forceXY=true -Dextractor.storage.dir=/home/user/tmp_extracts/"
   #GDAL
   JAVA_OPTS="$JAVA_OPTS -Djava.library.path=/var/sig/gdal/NativeLibs" 
   
Tomcat62 & Tomcat63 ::

   JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.trustStore=/opt/share/keystore -Djavax.net.ssl.trustStorePassword=mdp"
   #GEOSERVER
   JAVA_OPTS="$JAVA_OPTS -Xms2G -Xmx2G -XX:PermSize=256m -XX:MaxPermSize=256m -DGEOSERVER_DATA_DIR=/var/geoserver_datadir -DGEOWEBCACHE_CACHE_DIR=/var/geowebcache_datadir -Djava.awt.headless=true -Dfile.encoding=UTF8 -Djavax.servlet.request.encoding=UTF-8 -Djavax.servlet.response.encoding=UTF-8  -server -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:ParallelGCThreads=2  -XX:SoftRefLRUPolicyMSPerMB=36000 -XX:NewRatio=2 -XX:+AggressiveOpts"

L'utilisateur Tomcat doit être propriétaire des dossiers qu'il utilise : geoserver_datadir, geowebcache_datadir... ::

   chown -R tomcat:tomcat geoserver_datadir 
   
* **server.xml**

Ensuite, les numéros de port doivent être modifiés pour que chaque instance écoute sur un port différent. Les instances 1, 2 et 3 n'ont pas besoin du port SSL.

+--------------------+--------------------+-----------------+--------------+-----------+
| Nom de l'instance  | Connecteur HTTP    | Connecteur JK   | Port d'arrêt | Port SSL  |
+====================+====================+=================+==============+===========+
| Tomcat60           | 8080               | 8009            | 8005         | 8443      | 
+--------------------+--------------------+-----------------+--------------+-----------+
| Tomcat61           | 8180               | 8109            | 8105         | x         |
+--------------------+--------------------+-----------------+--------------+-----------+
| Tomcat62           | 8280               | 8209            | 8205         | x         |
+--------------------+--------------------+-----------------+--------------+-----------+
| Tomcat63           | 8380               | 8309            | 8305         | x         |
+--------------------+--------------------+-----------------+--------------+-----------+

Modifier les ports dans le fichier server.xml en suivant le tableau ci-dessus.

.. code-block :: bash

   cd /opt/tomcat6/tomcat6X/conf
   nano server.xml

Modifier la configuration des connecteurs ::

   #Exemple pour l'instance 1
   <Connector port="8080" protocol="HTTP/1.1" 
      connectionTimeout="20000" 
      URIEncoding="UTF-8"
      redirectPort="8443" />
	  
   <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
      URIEncoding="UTF-8"
      maxThreads="150" scheme="https" secure="true"
      clientAuth="false"
      keystoreFile="/opt/share/keystore"
      keystorePass="mdp"
      compression="on"
      compressionMinSize="2048"
      noCompressionUserAgents="gozilla, traviata"
      compressableMimeType="text/html,text/xml,text/javascript,application/x-javascript,application/javascript,text/css" />
	  
   <Connector URIEncoding="UTF-8"
      port="8009"
      protocol="AJP/1.3"
      connectionTimeout="20000"
      redirectPort="8443" />	

* **Manager**   

Par défaut, le manager n'est pas accessible puisque aucun utilisateur n'existe. Pour créer l'utilisateur, modifier le fichier /opt/tomcat6/tomcat6X/conf/tomcat-users.xml dans chaque instance de Tomcat ::
   
   <tomcat-users>
    <role rolename="manager-gui"/>
    <user username="admin" password="mdp" roles="manager-gui"/>
   </tomcat-users>
   
* **WebApp ROOT**

Supprimer la webapp ROOT des trois instances

* **Keystore**

Création du keystore (dans le dossier share)

.. code-block :: bash

   cd /opt/share/keystore
   keytool -genkey -alias georchestra_localhost -keystore keystore -storepass mdp -keypass mdp -keyalg RSA -keysize 2048
  
Prénom et nom : localhost

Load Balancing (serveur 2)
--------------------------

Mise en place
^^^^^^^^^^^^^

D'après http://java4it.blogspot.fr/2008/09/tutoriel-load-balancing-avec-apache-et.html

* Copier depuis /etc/apache2/mods-available vers /etc/apache2/mods-enabled : proxy_ajp.load, proxy_balancer.load, proxy_balancer.conf
* Ajouter l'attribut jvmroute à l'élément engine (dans  /opt/tomcat6/tomcat6X/conf/server.xml) ::

   #Tomcat62
   <engine name="Catalina" defaulthost="localhost" jvmroute="tomcat62">
   #Tomcat63
   <engine name="Catalina" defaulthost="localhost" jvmroute="tomcat63">
   
* Ajouter au fichier proxy_balancer.conf (dans /etc/apache2/mods-enabled/)::

   <Proxy balancer://mycluster>
     BalancerMember ajp://ip_server2:8209 min=10 max=100 route=tomcat62 loadfactor=1
     BalancerMember ajp://ip_server2:8309 min=10 max=100 route=tomcat63 loadfactor=1
     Order deny,allow
     Allow from all
   </Proxy>

   ProxyPass / balancer://mycluster/ stickysession=JSESSIONID

* Redémarrer les deux instances de Tomcat puis apache2

Gestion du geoserver_datadir
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

La configuration contenu dans le geoserver_data_dir est chargée uniquement au lancement de Geoserver (donc au lancement de Tomcat).
Il faut donc qu'après chaque modification sur une instance, les autres instances soient synchronisées.
L'api REST de Geoserver permet de recharger la conf via la commande reload en mode POST
Camptocamp a écrit un script python qui permet d'exécuter les commandes REST sur chaque instance. Nous l'avons légèrement modifié pour l'intégrer à notre plateforme.
Ce script est executé en mode CGI. Pour ce faire il faut :

* Activer le module cgi d'Apache
* Ajouter dans le virtualhost du site (/etc/apache2/sites-available/georchestra) ::

   ScriptAlias /scripts/ /var/www/georchestra/scripts/
   <Directory /var/www/georchestra/scripts/>        
      AddHandler cgi-script .cgi .py
      AllowOverride None
      Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
      Order allow,deny
      Allow from all      
   </Directory>

* Attention : Les sauts de lignes du fichier python doivent être au format Unix (LF), le propriétaire du dossier est www-data (l'utilisateur Apache).

* Pour le script voir :doc:`Code Python CGI </code>`


GDAL/OGR/JAI
============
L'installation de GDAL est nécessaire pour l'extracteur et permet l'ouverture de plus de format dans Mapfishapp. Elle permet aussi à Geoserver de supporter plus de formats.

* Activer le support JAI (Serveur 2)

D'après le blog geomatips :

.. code-block :: bash
   
   cd /usr/lib/jvm/java-1.7.0-openjdk-amd64/
   #Librairies JAI
   wget  http://download.java.net/media/jai/builds/release/1_1_3/jai-1_1_3-lib-linux-amd64-jdk.bin
   sh jai-1_1_3-lib-linux-amd64-jdk.bin
   
   #Librairies JAI imageIO
   wget http://download.java.net/media/jai-imageio/builds/release/1.1/jai_imageio-1_1-lib-linux-amd64-jdk.bin
   export _POSIX2_VERSION=199209
   sh jai_imageio-1_1-lib-linux-amd64-jdk.bin

A noter que les paquets Debian sont disponibles
   
Relancer TOMCAT (le support JAI doit être passé à TRUE dans l'etat du serveur, si geOrchestra a déjà été déployé)

* Installer les Natives Libs (Serveur 1&2)

Au début, nous avions pris les Natives de geo-solutions, maintenant C2C met à disposition les siennes

.. code-block :: bash
   
   cd /home/user/download
   wget http://sdi.georchestra.org/%7Epmauduit/gdalogr-java-bindings/gdal-georchestra-debian6-amd64-mifmid-patched.tar.gz
   tar xvzf gdal-georchestra-debian6-amd64-mifmid-patched.tar.gz
   mkdir -p /var/sig/gdal/NativeLibs/
   cd /var/sig/gdal/NativeLibs/
   #Copier le contenu du dossier java   
   
   cd ..
   mkdir gdal-data
   #Copier le contenu du dossier share
   

Il faut ensuite définir les variables d'environnement correspondantes.
Dans /etc/environnement : ::
  
   GDAL_DATA="/var/sig/gdal/gdal-data"
   LD_LIBRARY_PATH="/var/sig/gdal/NativeLibs:/lib:/usr/lib"

* Installer la libECW

Récupérer l'archive http://mirror.ovh.net/gentoo-distfiles/distfiles/libecwj2-3.3-2006-09-06.zip

.. code-block :: bash

   unzip libecwj2-3.3-2006-09-06.zip
   cd libecwj2-3.3-2006-09-06
   ./configure
   make
   make install
   
* Installer GDAL

Avant de lancer la compilation de GDAL, le paquet libxerces peut-être necessaire :

.. code-block :: bash

   apt-get install libxerces-c-dev
   cd /home/igeo/download
   wget http://download.osgeo.org/gdal/1.10.1/gdal-1.10.1.tar.gz
   tar xzf gdal-1.10.1.tar.gz
   cd gdal-1.10.1
   ./configure
   make
   make install
   
* Installer l'extension GDAL Geoserver

.. code-block :: bash

   cd /opt/tomcat6/tomcat62/webapps/geoserver/WEB-INF/lib/
   wget http://downloads.sourceforge.net/geoserver/geoserver-2.3.2-gdal-plugin.zip
   unzip geoserver-2.3.2-gdal-plugin.zip
   rm -f geoserver-2.3.3-gdal-plugin.zip


Retour au :doc:`Sommaire </index>`