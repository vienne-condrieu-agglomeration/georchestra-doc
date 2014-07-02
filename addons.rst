Gestion des addons
******************

L'activation d'un addon se fait dans le fichier de configuration GEOR_custom de Mapfishapp (/configuration/profil/app/js/).

Cadastre
========
Ce plugin permet de localiser une parcelle en renseignant la commune, la section cadastrale et le numero de section. La recherche par propriétaire est aussi disponible mais elle ne permet pas la localisation de la parcelle. Il faut être connecté pour avoir accès à cette fonctionnalité.

Activation du plugin
--------------------
Rajouter dans la variable ADDONS ::

   {
        "id": "cadastre", // Nom tel qu'il est présent dans l'arborescence
        "name": "Cadastre", // Nom d'affichage
        "title": {
            "en": "Cadastre",
            "es": "Cadastro",
            "fr": "Cadatsre"
        },
        "description": {
            "en": "Allow to locate a cadastral parcel",
            "es": "Puede localizar una parcela catastral",
            "fr": "Permet de localiser une parcelle cadastrale"
        }
   }

Configuration du plugin
-----------------------
Dans les sources de GeOrchestra/mapfishapp/src/main/webapp/app/addons/cadastre

Le fichier cities.json doit contenir toutes les villes pour lesquelles la recherche est possible. Mise en forme : ::

   {"type":"FeatureCollection","features":[
      {"type":"Feature","id":"19","properties":{"nom":"VIENNE","insee":"38544"}},
      {"type":"Feature","id":"20","properties":{"nom":"SEYSSUEL","insee":"38487"}},
	  ...
   ]}
   
Dans le fichier manifest.json modifier les serveurs wfs pour qu'ils correspondent à nos sections et parcelles.

.. code-block :: json

   "tab1": {
      "field1": {
         "file": "citiesVA.json",
         "geometry": "the_geom",
         "wfs": "http://vm-georchestra/geoserver/wfs",
         "typename": "julien:capv_dgi_commune",
         "valuefield": "insee",
         "displayfield": "nom",
         "template": "<b>{nom}</b> ({insee})"
      },
      "field2": {
         "wfs": "http://vm-georchestra/geoserver/wfs",
         "typename": "julien:capv_dgi_section",
         "geometry": "the_geom",
         "matchingproperties": {
            "field1": "insee"
         },
         "valuefield": "designation",
         "displayfield": "designation",
         "template": "<b>{designation}</b>"
      },
      "field3": {
         "wfs": "http://vm-georchestra/geoserver/wfs",
         "typename": "julien:capv_dgi_parcelle",
         "geometry": "the_geom",
         "matchingproperties": {
            "field1": "insee",
            "field2": "section"
         },
         "valuefield": "numparc",
         "displayfield": "numparc",
         "template": "<b>{numparc}</b> {section}"
      }
   },
   "tab2": {
      "field1": {
         "this field": "is currently the same as field1 from tab1",
         "no config option here": "is taken into account"
      }, 
      "field2": {
         "wfs": "http://vm-georchestra/geoserver/wfs",
         "typename": "julien:capv_dgi_parcelle_unite_fiscale_avancee",
         "matchingproperties": {
            "field1": "insee"
         },
         "valuefield": "ddenom",
         "displayfield": "ddenom"
      }
   }
   
   
Loupe Orthophoto
================
La loupe orthophoto permet d'afficher dans une fenêtre flottante un service wms autre que le fond de plan. On peut zoomer dans cette fenêtre.
Elle est déjà activée par défaut dans le GEOR_custom. Par défaut c'est une image satellite proposée par un service de GeoBretagne. Pour modifier le service c'est dans le manifest.json que ça ce passe ::

   "default_options": {
      "mode": "static",
      "baseLayerConfig": {
         "wmsurl": "http://vm-georchestra/geoserver/wms",
         "layer": "julien:CAPV_ORTHO_2009",
         "format": "image/jpeg",
         "buffer": 8
      }
   },


Retour au :doc:`Sommaire </index>`