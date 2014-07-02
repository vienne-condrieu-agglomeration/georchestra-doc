Configuration
*************

La configuration de Georchestra se fait en dupliquant le dossier template disponible sous config/configurations/template.
Ce dossier est divisé en sous-dossier représentant chacun une webapp. Un seul fichier est transversale à toutes les webapps : shared.maven.filter. Ce fichier contient des variables qui seront réutilisées dans toutes les webapps (nom du serveur, adresse mail, utilisateur postgres...).
Cette doc présente la configuration par dossier/webapp.
Certains fichiers/dossiers ne sont pas, par défaut, dans la config template, il faut les copier depuis config/default.

.. warning::

   Seuls les dossiers modifiés dans le cadre de la mise en place de notre plateforme sont présents. Cette page est donc vouée à évoluer au fur et à mesure de la personnalisation. 

shared.maven.filter
===================
C'est le fichier qui définit les variables globales. La plupart des variables disponibles sont largement commentées.
On y trouve notamment : la langue de l'interface, les paramètres de connexion aux BDD, le niveau de log, l'activation ou non du formulaire de téléchargement ou encore l'activation ou non des statistiques.

analytics
=========
Pas de conf particulière, les paramètres sont définis à partir du shared.maven.filter

cas-server-webapp
=================
Définition du css de la page de login.

extractorapp
============
* WEB-INF/templates : Modèles des mails envoyés lors des extractions
* app/js/GEOR_CUSTOM.js

Ce fichier permet de configurer la page d'extraction (couches présentes par défaut dans l'extracteur, emprise par défaut, les SRS ...)

.. warning::

   Il faut utiliser les services WMS pour les startups layers. Les services WFS ne fonctionnent pas
   
geoserver-webapp
================
Le fichier web.xml copié depuis georchestra/geoserver/geoserver-submodule/src/web/app/src/main/webapp/WEB-INF/web.xml permet de determiner le geoserver_datadir.

header
======
* Modification du logo du header via le dossier img
* Modification du header via le fichier header.jsp (copié depuis home/igeo/download/georchestra/header/src/main/webapp/WEB-INF/jsp)

LDAP Admin
==========
Dans WEB-INF/template (copié depuis default) --> modification des mails envoyés lors d'inscription.

mapfishapp
==========
* Dans WEB-INF/print : Modification des templates d'impression
* Dans app/js/GEOR_CUSTOM : C'est le fichier principal de configuration. On peut choisir les WMC présents, les echelles de visualisation, les addons (voir `ici <http://vm-georchestra/doc/addons.html>`_), les services proposés par défaut...
* [*].wmc : Web Map Context : Ce sont des fichiers qui chargent une configuration. Une fois dans mapfish on peut choisir un des contextes disponibles. Ils définissent principalement l'emprise et les couches disponibles.

.. Note::

   D'après le cours laval :
   Standardisé par l'OGC depuis 2005
   Document permettant le regroupement d'une ou plusieurs cartes provenant d'un ou plusieurs services cartographiques (WMS). 
   
   On y décrit les informations suivantes:
   
   * le(s) serveur(s) fournissant le(s) couche(s) de la carte générale,
   
   * l'étendue géographique et la projection cartographique de chaque couche partagée,
   
   * des métadonnées opérationnelles pour qu'un logiciel client puisse reproduire la carte générale,
   
   * des métadonnées auxiliaires pour annoter et décrire les cartes et leur provenance.
   
security-proxy
==============
Dans le maven.filter, il faut modifier les ports et IP de redirection ::

   proxy.defaultTarget=http://localhost:8080
   
   # DEVIENT   
   
   proxy.defaultTarget=http://localhost:8180
   proxy.geoserver=http://ipserver2:8280
   
   # Avec le load balancing, on ne précise plus le port de geoserver
   
   proxy.geoserver=http://ipserver2
   
Deuxième chose à modifier, geoserver-private devient geoserver ::
   
   <entry key="geoserver" value="${proxy.defaultTarget}/geoserver-private/" />\
   <entry key="geoserver" value="${proxy.defaultTarget}/geoserver/" />\

   
Retour au :doc:`Sommaire </index>`