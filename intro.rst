.. geOrchestra documentation master file, created by
   sphinx-quickstart on Fri Mar 28 10:58:25 2014.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Introduction
************

Cette documentation a pour but de décrire la mise en place de la plateforme geOrchestra suivant l'architecture ci-dessous. Cette architecture est composée de deux serveurs* et les webapps de geOrchestra sont divisées sur quatres instances de Tomcat. Le serveur 2 met en place un load balancing entre deux instances de Tomcat contenant chacunes un Geoserver.

La rédaction de cette documentation est passée par la mise en place de plusieurs machine virtuelle et a pour base la documentation disponible sur le dépot geOrchestra de GitHub. Elle n'est pas exhaustive et a été rédigé par prise de notes.

Avant d'entrer à proprement dit dans l'installation de geOrchestra, il est necessaires de configurer les serveurs.

Serveur Linux sous Debian 7.4

   .. figure::  img/archi.PNG
      :align:   center

      Architecture de la solution.  
   
