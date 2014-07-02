Memo, pense-bête
****************

Fonctionnement du proxy
=======================

Dans Georchestra, les urls sont redirigées plusieurs fois entre le client et la webapp concernée. Le schéma ci-dessous récapitule les étapes de redirection :

   .. figure::  img/proxy.JPG
      :align:   center
   

Accès privé aux couches
=======================
Problématique : On veut limiter l'accès à certaines couches à certains utilisateurs.
On prend l'exemple des partenaires urbanistes qui accèdent à toutes les données publiques + les données d'urbanismes

**Dans Ldap_admin** : Créer un groupe nommé URBA

**Dans Geoserver** :

* Créer un nouvel espace de travail
* Créer un nouveau rôle 'ROLE_URBA'
* Editer les règle d'accès à l'espace de travail.
* Catalog Mode : mélangé

Créer un service tomcat
=======================
Problématique : On veut démarrer/stopper/redemarrer chaque instance de tomcat facilement via une simple commande. Tomcat doit aussi être lancé au démarrage du serveur.

* Dans /etc/init.d, créer un fichier tomcat60d ::

   #!/bin/bash

   ### BEGIN INIT INFO
   # Provides:          /etc/init.d/tomcat60d
   # Required-Start:    $remote_fs $syslog
   # Required-Stop:     $remote_fs $syslog
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: Start daemon at boot time
   # Description:       Enable service provided by daemon.
   ### END INIT INFO
   
   START_TOMCAT=/opt/tomcat6/tomcat60/bin/startup.sh
   STOP_TOMCAT=/opt/tomcat6/tomcat60/bin/shutdown.sh
   PROG="tomcat60"
   
   start(){
   	echo -n "Starting $PROG: "
    #Demarrer avec l'utilisateur tomcat
   	su -p -s /bin/sh tomcat ${START_TOMCAT}
   	echo "done."
   }
    
   stop(){
   	echo -n "Shutting down $PROG: "
   	su -p -s /bin/sh tomcat ${STOP_TOMCAT}
   	echo "done."
   }
    
   restart(){
      stop
      sleep 10
      start
   }
    
   reload(){
      restart
   }
    
   case "$1" in
     start)
         start
         ;;
     stop)
         stop
         ;;
     restart)
         restart
         ;;
     reload)
         reload
         ;;
     *)
         echo "Usage : $0 {start|stop|restart|reload}"
   esac
    
   exit 0
 

* Ajouter le script au services Debian ::

   chkconfig --add tomcat60d

* Rendre executable le script ::

   chmod +x tomcat60d
   
* Gérer le service via les commandes ::

   service tomcat60d start/stop/restart/reload
   
* Même chose pour les autres instances de Tomcat


Retour au :doc:`Sommaire </index>`
