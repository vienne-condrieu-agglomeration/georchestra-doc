.. geOrchestra documentation master file, created by
   sphinx-quickstart on Fri Mar 28 10:58:25 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Mise à jour 14.06
*****************

Le processus de mise à jour est decrit dans les release_notes du depot GiHub de geOrchestra. Nous decrivons ici, les changements effectués à ViennAgglo

Dossier de configuration
========================

Quelques modifications dans le dossier de configuration sont à noter :

* Remplacement du dossier CAS par le nouveau 
* Suppression du dossier security-proxy (le proxy est définit dans le script GenerateConfig.groovy)
* Suppression des éléments du shared.maven.filter devenu obsolètes
* Ajout du script GenerateConfig.groovy, et modification pour que le proxy fonctionne ::

   ...
   /**
    * updateSecProxyMavenFilters
    */
    def updateSecProxyMavenFilters() {

        def proxyDefaultTarget = "http://localhost:8180"
        def proxyGeoserver= "http://192.168.20.114"

        new PropertyUpdate(
            path: 'maven.filter',
            from: 'defaults/security-proxy',
            to: 'security-proxy'
        ).update { properties ->
            properties['cas.private.host'] = "localhost"
            properties['public.ssl'] = "443"
            properties['private.ssl'] = "8543"
            properties['proxy.defaultTarget'] = proxyDefaultTarget
            properties['proxy.geoserver'] = proxyGeoserver
            properties['proxy.mapping'] = """
   <entry key="analytics"     value="proxyDefaultTarget/analytics/" />
   <entry key="catalogapp"    value="proxyDefaultTarget/catalogapp/" />
   <entry key="downloadform"  value="proxyDefaultTarget/downloadform/" />
   <entry key="extractorapp"  value="proxyDefaultTarget/extractorapp/" />
   <entry key="geonetwork"    value="proxyDefaultTarget/geonetwork/" />
   <entry key="geoserver"     value="proxyGeoserver/geoserver/" />
   <entry key="geowebcache"   value="proxyDefaultTarget/geowebcache/" />
   <entry key="geofence"      value="proxyDefaultTarget/geofence/" />
   <entry key="header"        value="proxyDefaultTarget/header/" />
   <entry key="ldapadmin"     value="proxyDefaultTarget/ldapadmin/" />
   <entry key="mapfishapp"    value="proxyDefaultTarget/mapfishapp/" />
   <entry key="static"        value="proxyDefaultTarget/header/" />""".replaceAll("\n|\t","").replaceAll("proxyDefaultTarget",proxyDefaultTarget).replaceAll("p$
            properties['header.mapping'] = """

  
Migration base geonetwork
=========================

Dans la version 14.06, la base PostgreSQL de Geonetwork devient un schéma de la base georchestra.

* Recuperation de la base geonetwork --> georchestra.sql
* Injection dans la base georchestra ::

   pg_dump.exe --host 192.168.20.101 --port 5432 --username "dyndb" --role "dyndb" --no-password  --format plain --encoding UTF8 --verbose --file "C:\Users\jfrancois\Documents\georchestra.sql" --schema "public" --inserts -t categories -t categoriesdes -t cswservercapabilitiesinfo -t customelementset -t groups -t groupsdes -t harvesthistory -t isolanguages -t isolanguagesdes -t languages -t metadata -t metadatacateg -t metadatanotifications -t metadatanotifiers -t metadatarating -t metadatastatus -t operationallowed -t operations -t operationsdes -t params -t regions -t regionsdes -t relations -t requests -t serviceparameters -t services -t settings -t sources -t statusvalues -t statusvaluesdes -t thesaurus -t usergroups -t users -t validation "geonetwork"
   
* Changement des droits d'accès ::

   postgres=# ALTER ROLE geonetwork WITH ENCRYPTED PASSWORD 'www-data';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE georchestra to geonetwork;
   postgres=# GRANT ALL PRIVILEGES ON TABLE geonetwork.categories to geonetwork;
   
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.categories to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.categoriesdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.cswservercapabilitiesinfo to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.customelementset to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.groups to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.groupsdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.harvesthistory to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.isolanguages to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.isolanguagesdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.languages to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadata to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadatacateg to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadatanotifications to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadatanotifiers to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadatarating to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.metadatastatus to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.operationallowed to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.operations to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.operationsdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.params to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.regions to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.regionsdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.relations to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.requests to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.serviceparameters to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.services to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.settings to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.sources to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.statusvalues to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.statusvaluesdes to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.thesaurus to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.usergroups to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.users to geonetwork;
   georchestra=# GRANT ALL PRIVILEGES ON TABLE geonetwork.validation to geonetwork;


Supressions des "-privates"
===========================

Les applications ne comportent plus le suffixe "-private" dorénavant. Plusieurs fichiers sont donc à modifier :

* Les JAVA_OPTS de geonetwork dans le catalina.sh
* Le script de deploiement
   
LDAP
====
Notre base LDAP n'étant pas encore en production au moment du changement de version, nous l'avons supprimé puis recréé avec les nouvelles préconisations de la version 14.06   



Retour au :doc:`Sommaire </index>`