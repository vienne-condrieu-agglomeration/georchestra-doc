.. geOrchestra documentation master file, created by
   sphinx-quickstart on Fri Mar 28 10:58:25 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Georchestra
***********

Point de départ
===============
* Cloner le dépôt GitHub de georchestra :

.. code-block :: bash
   
   cd /home/user/download
   git clone -b 14.01 --recursive https://github.com/georchestra/georchestra.git

* Compiler

.. code-block :: bash

   cd georchestra
   ./mvn -Dmaven.test.skip=true -Ptemplate install
   
Configuration
=============

La configuration de georchestra se fait en clonant le dossier template du dossier config/configuration. On peut ensuite modifier les élément de ce dossier et lancer la compilation comme suit. 

.. code-block :: bash
    
   cd georchestra
   ./mvn -Dmaven.test.skip=true -Dserver=viennagglo install   

On peut aussi compiler une seule webapp à la fois ::

   cd /home/user/download/georchestra/mapfishapp
   ../mvn -Dmaven.test.skip=true -Dserver=viennagglo install
   
Deploiement
===========

Pour compiler/deployer les différentes webapps dans les trois instances de Tomcat, nous avons réalisé le script bash disponible ici :doc:`Script de compilation </code>`

Retour au :doc:`Sommaire </index>`