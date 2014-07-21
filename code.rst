Codes
*****

Création de la base PostgreSQL
==============================

.. code-block :: sql

   -- MAPFISHAPP
   create schema mapfishapp;
   
   create table mapfishapp.geodocs (
     id bigserial primary key, -- 1 to 9223372036854775807 (~ 1E19)
     username varchar(200), -- can be NULL (eg: anonymous user)
     standard varchar(3) not null, -- eg: CSV, KML, SLD, WMC
     raw_file_content text not null, -- file content
     file_hash varchar(32) unique not null, -- md5sum
     created_at timestamp without time zone default NOW(), -- creation date
     last_access timestamp without time zone, -- last access date
     access_count integer default 0 -- access count, defaults to 0
   );
   
   create index geodocs_file_hash on mapfishapp.geodocs using btree (file_hash);
   create index geodocs_username on mapfishapp.geodocs using btree (username);
   create index geodocs_standard on mapfishapp.geodocs using btree (standard);
   create index geodocs_created_at on mapfishapp.geodocs using btree (created_at);
   create index geodocs_last_access on mapfishapp.geodocs using btree (last_access);
   create index geodocs_access_count on mapfishapp.geodocs using btree (access_count);
   
   -- LDAP
   CREATE SCHEMA ldapadmin;
   
   SET search_path TO ldapadmin,public,pg_catalog;
   
   
   CREATE TABLE user_token (
       uid character varying NOT NULL,
       token character varying,
       creation_date timestamp with time zone
   );
   
   ALTER TABLE ONLY user_token
       ADD CONSTRAINT uid PRIMARY KEY (uid);
   
   CREATE UNIQUE INDEX token_idx ON user_token USING btree (token);
   
   -- DOWNLOAD FORM
   create schema downloadform;
   
   set search_path to downloadform,public,pg_catalog;
   
   create table log_table (
     id serial primary key,
     username varchar(200), -- can be NULL (eg: anonymous user)
     sessionid varchar(32) not null, -- this is the security-proxy JSESSIONID
     first_name varchar(200) not null,
     second_name varchar(200) not null,
     company varchar(200) not null,
     email varchar(200) not null,
     phone varchar(100),
     requested_at timestamp without time zone default NOW(),
     comment text
   );
   create index log_table_username on log_table using btree (username);
   create index log_table_sessionid on log_table using btree (sessionid);
   
   
   -- GN: log MD id and filename (resource.get parameters)
   create table geonetwork_log (
     metadata_id integer not null, -- this is not the UUID, but the local ID
     filename varchar(200) not null
   ) inherits (log_table);
   create index geonetwork_log_id_fname on geonetwork_log using btree (metadata_id, filename);
   
   -- extractorapp log table, which contains just the JSON spec for now (could be exploited later client side to display extracted stuff)
   -- json_spec example : {"emails":["toto@titi.com"],"globalProperties":{"projection":"EPSG:4326","resolution":0.5,"rasterFormat":"geotiff","vectorFormat":"shp","bbox":{"srs":"EPSG:4326","value":[-2.2,42.6,1.9,46]}},"layers":[{"projection":null,"resolution":null,"format":null,"bbox":null,"owsUrl":"http://s.com/geoserver/wfs/WfsDispatcher?","owsType":"WFS","layerName":"pigma:cantons"},{"projection":null,"resolution":null,"format":null,"bbox":null,"owsUrl":"http://s.com/geoserver/pigma/wcs?","owsType":"WCS","layerName":"pigma:protected_layer_for_integration_testing"}]}
   create table extractorapp_log (
     json_spec text not null
   ) inherits (log_table);
   create index extractorapp_log_json_spec on extractorapp_log using btree (json_spec);
   
   
   create table data_use (
     id serial primary key,
     name varchar(100)
   );
   
   -- sample data:
   insert into data_use (name) values ('Administratif et budgétaire');
   insert into data_use (name) values ('Aménagement du Territoire et Gestion de l''Espace');
   insert into data_use (name) values ('Communication');
   insert into data_use (name) values ('Environnement');
   insert into data_use (name) values ('Fond de Plan');
   insert into data_use (name) values ('Foncier et Urbanisme');
   insert into data_use (name) values ('Formation');
   insert into data_use (name) values ('Gestion du Domaine Public');
   insert into data_use (name) values ('Mise en valeur du Territoire (Tourisme)');
   insert into data_use (name) values ('Risques Naturels et Technologiques');
   
   
   create table logtable_datause (
     logtable_id integer not null,
     datause_id integer not null,
     primary key (logtable_id, datause_id)
   );
   -- commented out because it generates an error:
   --alter table logtable_datause add constraint fk_logtable_id foreign key (logtable_id) REFERENCES log_table (id) ;
   --org.postgresql.util.PSQLException: ERROR: insert or update on table "logtable_datause" violates foreign key constraint "fk_logtable_id"
   --Detail: Key (logtable_id)=(2) is not present in table "log_table".
   --  at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(QueryExecutorImpl.java:2102)
   alter table logtable_datause add constraint fk_datause_id foreign key (datause_id) REFERENCES data_use (id) ;
   
   create table extractorapp_layers (
     id serial primary key,
     extractorapp_log_id integer NOT NULL,
     projection character varying(12),
     resolution double precision,
     format character varying(10),
     bbox_srs character varying(12),
     "left" double precision,
     bottom double precision,
     "right" double precision,
     top double precision,
     ows_url character varying(1024),
     ows_type character varying(3),
     layer_name text
   );
   
   create index extractorapp_layers_layer_name on extractorapp_layers using btree (layer_name);
   
   -- STATISTIQUES
   CREATE SCHEMA ogcstatistics;

   SET search_path TO ogcstatistics,public,pg_catalog;
   
   
   CREATE TABLE ogc_services_log (
     user_name character varying(255),
     date date,
     service character varying(5),
     layer character varying(255),
     id bigserial NOT NULL,
     request character varying(20),
     org character varying(255),
     CONSTRAINT primary_key PRIMARY KEY (id )
   );
   
   CREATE INDEX user_name_index ON ogc_services_log USING btree (user_name);
   CREATE INDEX date_index ON ogc_services_log USING btree (date);
   CREATE INDEX service_index ON ogc_services_log USING btree (service);
   CREATE INDEX layer_index ON ogc_services_log USING btree (layer);

Code Python CGI
===============

Le script d'origine est disponible ici https://gist.github.com/fvanderbiest/f5d5e467c7ca004ce73b 

.. code-block :: guess

   #!/usr/bin/env python
   #-*- coding: utf-8 -*-
   
   print 'Content-Type: text/html; charset=utf-8\n'
   
   """This is a script that we use to reload geoserver catalogs when load balancing them"""
   
   """
   Copyright 2014 Camptocamp. All rights reserved.
   
   Redistribution and use in source and binary forms, with or without modification, are
   permitted provided that the following conditions are met:
   
      1. Redistributions of source code must retain the above copyright notice, this list of
         conditions and the following disclaimer.
   
      2. Redistributions in binary form must reproduce the above copyright notice, this list
         of conditions and the following disclaimer in the documentation and/or other materials
         provided with the distribution.
   
   THIS SOFTWARE IS PROVIDED BY CAMPTOCAMP ``AS IS'' AND ANY EXPRESS OR IMPLIED
   WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND
   FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL CAMPTOCAMP OR
   CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
   CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
   SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
   ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
   NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
   ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
   
   The views and conclusions contained in the software and documentation are those of the
   authors and should not be interpreted as representing official policies, either expressed
   or implied, of Camptocamp.
   """
   
   config = {
       'geoserver': [
           'ipserver2:8280',
           'ipserver2:8380',
           #'', # next one
       ]
   }
   
   # DO NOT MODIFY ANYTHING BELOW THIS #
   # (unless you know what you're doing)
   
   import cgi
   import sys, os
   import urllib2
   
   headers = {
       "sec-roles": "ROLE_ADMINISTRATOR",
       "sec-username": "fake_user",
   }
   
   failures = []
   
   for host in config['geoserver']:
       req = urllib2.Request("http://"+host+"/geoserver/rest/reload?recurse=true", "nothing", headers)
       try:
           r = urllib2.urlopen(req)
       except urllib2.URLError as e:
           failures.append((host, e.reason))
   
   #Le retour est json qui sera exploité avec jquery
   if len(failures) == 0:
       print '{"message": "Reload OK", "style": "btn-success"}'
   else:
       #print "Status: 500 Internal Server Error"
       #print ""
       #for f in failures:
           #print "Reload failed for host %s, reason is '%s'" %  f
       print '{"message": "Reload failed for '+ failures[0][0] +', reason is '+ failures[0][1] +'", "style": "btn-danger"}
   
Script de compilation & deploiement
===================================

Script serveur1  

.. code-block :: guess
   
   #!/bin/bash

   date
   
   #VARIABLE DE PROFIL, VERSION et repertoire GitHub
   PROFILE=viennagglo
   VERSION=14.06
   GITDIR=/home/user/download/georchestra_1406/
   TOMCAT60=/opt/tomcat6/tomcat60
   TOMCAT61=/opt/tomcat6/tomcat61
   
   #REPERTOIRE GEORCHESTRA
   cd ${GITDIR}
   
   #COMPILATION
   ./mvn -Dmaven.test.skip=true -Dserver=${PROFILE} install
   
   #CREATION D'UN DOSSIER TEMPORAIRE
   mkdir /tmp/georchestra_deploy_tmp
   cd /tmp/georchestra_deploy_tmp
   
   #COPIE DE TOUT LES WAR COMPILE DANS LE DOSSIER TEMPORAIRE
   find /root/.m2/repository/ -name "*${VERSION}-${PROFILE}.war" -exec cp {} /tmp/georchestra_deploy_tmp \;
   echo COPY TMP OK
   
   #RENOMMER TOUT LES WAR
   mv security-proxy-${VERSION}-${PROFILE}.war ROOT.war
   mv analytics-${VERSION}-${PROFILE}.war analytics.war
   mv cas-server-webapp-${VERSION}-${PROFILE}.war cas.war
   mv catalogapp-${VERSION}-${PROFILE}.war catalogapp.war
   mv downloadform-${VERSION}-${PROFILE}.war downloadform.war
   mv extractorapp-${VERSION}-${PROFILE}.war extractorapp.war
   mv geonetwork-main-${VERSION}-${PROFILE}.war geonetwork.war
   mv geoserver-webapp-${VERSION}-${PROFILE}.war geoserver.war
   mv ldapadmin-${VERSION}-${PROFILE}.war ldapadmin.war
   mv mapfishapp-${VERSION}-${PROFILE}.war mapfishapp.war
   mv header-${VERSION}-${PROFILE}.war header.war
   mv geowebcache-webapp-${VERSION}-${PROFILE}.war geowebcache.war
   echo RENAME OK
   
   #ARRET DE TOMCAT60
   service tomcat60d stop
   echo TOMCAT60 SHUTDOWN
   
   #ARRET DE TOMCAT61
   service tomcat61d stop
   echo TOMCAT61 SHUTDOWN
   
   #SUPRESSION DES  FICHIERS WAR (TOMCAT60)
   rm -Rf ${TOMCAT60}/webapps/ROOT*
   rm -Rf ${TOMCAT60}/webapps/cas*
   echo WAR60 DELETED
   
   #SUPRESSION DES  FICHIERS WAR (TOMCAT61)
   rm -Rf ${TOMCAT61}/webapps/analytics*
   rm -Rf ${TOMCAT61}/webapps/catalogapp*
   rm -Rf ${TOMCAT61}/webapps/downloadform*
   rm -Rf ${TOMCAT61}/webapps/extractorapp*
   rm -Rf ${TOMCAT61}/webapps/geonetwork*
   rm -Rf ${TOMCAT61}/webapps/ldapadmin*
   rm -Rf ${TOMCAT61}/webapps/mapfishapp*
   rm -Rf ${TOMCAT61}/webapps/header*
   rm -Rf ${TOMCAT61}/webapps/geowebcache*
   echo WAR61 DELETED
   
   #COPIER le cas et root vers les webapps de TOMCAT60
   cp -f /tmp/georchestra_deploy_tmp/cas.war ${TOMCAT60}/webapps
   cp -f /tmp/georchestra_deploy_tmp/ROOT.war ${TOMCAT60}/webapps
   echo COPY WAR60 OK
   
   #COPIER TOUT LES WAR VERS LES WEBAPPS DE TOMCAT61 SAUF GEOSERVER
   cp -f /tmp/georchestra_deploy_tmp/analytics.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/catalogapp.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/downloadform.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/extractorapp.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/geonetwork.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/ldapadmin.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/mapfishapp.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/header.war ${TOMCAT61}/webapps
   cp -f /tmp/georchestra_deploy_tmp/geowebcache.war ${TOMCAT61}/webapps
   echo COPY WAR61 OK
   
   #DEMARRAGE DE TOMCAT 60
   service tomcat60d start
   echo TOMCAT60 STARTED
   
   #DEMARRAGE DE TOMCAT 61
   service tomcat60d start
   echo TOMCAT61 STARTED
   
   #Copie du war geoserver sur le serveur 2
   scp /tmp/georchestra_deploy_tmp/geoserver.war root@ipserveur2:/home/user/compil_data
   #Lancement du script de deploiement de geoserver sur le serveur 2
   ssh root@ipserveur2 /home/user/scripts/DEPLOY.sh
   
   echo ------------------------------------
   echo --     COMPILATION TERMINEE       --
   date
   echo ------------------------------------

Script serveur2

.. code-block :: guess

   #!/bin/bash

   #ARRET DE TOMCAT62
   cd  /opt/tomcat6/tomcat62/bin/
   su -p -s /bin/sh tomcat shutdown.sh
   echo TOMCAT62 SHUTDOWN
   
   #ARRET DE TOMCAT63
   cd  /opt/tomcat6/tomcat63/bin/
   su -p -s /bin/sh tomcat shutdown.sh
   echo TOMCAT63 SHUTDOWN
   
   #SUPRESSION DES FICHIERS WAR (TOMCAT62)
   rm -Rf /opt/tomcat6/tomcat62/webapps/*geoserver*
   echo WAR62 DELETED
   
   #SUPRESSION DES FICHIERS WAR (TOMCAT63)
   rm -Rf /opt/tomcat6/tomcat63/webapps/*geoserver*
   echo WAR63 DELETED
   
   #COPIER GEOSERVER VERS LES WEBAPPS DE TOMCAT62
   cp /home/user/compil_data/geoserver.war /opt/tomcat6/tomcat62/webapps/
   mkdir /opt/tomcat6/tomcat62/webapps/geoserver
   
   #DEPLOYER GEOSERVER
   cd /opt/tomcat6/tomcat62/webapps
   unzip geoserver.war -d geoserver
   
   #AJOUT DU LOG POUR GEOSERVER62
   sed -i "60a <context-param>\n<param-name>GEOSERVER_LOG_LOCATION</param-name>\n<param-value>/var/geoserver_datadir/logs/geoserver_62.log</param-value>\n</context-param>" geoserver/WEB-INF/web.xml
   
   #AJOUT DE L'EXTENSION ECW
   cd /opt/tomcat6/tomcat62/webapps/geoserver/WEB-INF/lib/
   wget http://downloads.sourceforge.net/geoserver/geoserver-2.3.2-gdal-plugin.zip
   unzip geoserver-2.3.2-gdal-plugin.zip
   
   echo WAR62 OK
   
   #COPIER LE GEOSERVER VERS LES WEBAPPS DE TOMCAT63
   cp /home/user/compil_data/geoserver.war /opt/tomcat6/tomcat63/webapps/
   mkdir /opt/tomcat6/tomcat63/webapps/geoserver
   
   #DEPLOYER GEOSERVER
   cd /opt/tomcat6/tomcat63/webapps
   unzip geoserver.war -d geoserver
   
   #AJOUT DU LOG POUR GEOSERVER63
   sed -i "60a <context-param>\n<param-name>GEOSERVER_LOG_LOCATION</param-name>\n<param-value>/var/geoserver_datadir/logs/geoserver_63.log</param-value>\n</context-param>" geoserver/WEB-INF/web.xml
   
   #AJOUT DE L'EXTENSION ECW
   cd /opt/tomcat6/tomcat63/webapps/geoserver/WEB-INF/lib/
   wget http://downloads.sourceforge.net/geoserver/geoserver-2.3.2-gdal-plugin.zip
   unzip geoserver-2.3.2-gdal-plugin.zip
   
   echo WAR63 OK
   
   #DEMARRAGE DE TOMCAT 62
   cd  /opt/tomcat6/tomcat62/bin/
   su -p -s /bin/sh tomcat startup.sh
   echo TOMCAT62 STARTED
   
   #DEMARRAGE DE TOMCAT 63
   cd  /opt/tomcat6/tomcat63/bin/
   su -p -s /bin/sh tomcat startup.sh
   echo TOMCAT63 STARTED
   
Retour au :doc:`Sommaire </index>`
