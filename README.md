# IDPS_logs_system

## I.Introduction
La sécurité des réseaux informatiques représente aujourd’hui un enjeu majeur pour les organisations, face à la multiplication des cyberattaques et l’exploitation des vulnérabilités systèmes. Afin d’assurer une protection efficace des infrastructures, il est essentiel de disposer de mécanismes capables de détecter les anomalies et de centraliser les journaux d’activité (logs) pour une meilleure analyse et réponse aux incidents.

Notre projet a pour objectif de concevoir et de mettre en œuvre une solution complète qui nous permettra:
la collecte centralisée des logs issus des équipements de notre réseau,
la détection d’intrusions à l’aide d’un système IDS/IPS comme Snort,
 la visualisation et l’analyse des événements de sécurité avec la pile ELK (Elasticsearch, Logstash, Kibana),
 la notification en cas d’un potentiel incident.


Le projet vise ainsi à renforcer la visibilité et la fiabilité de la supervision des réseaux informatiques. En simulant différents scénarios d’attaques, il permet de démontrer l’efficacité du système mis en place pour la détection d’intrusions et la gestion des incidents.   


## II.Outils utilisés
Pour la réalisation de notre projet nous avons utilisés différents outils tels que:
  ### 1. La Pile ELK (Elasticsearch, Logstash, Kibana)
 La Pile ELK est une suite d'outils utilisée pour collecter, stocker, analyser et visualiser de grands volumes de données. Dans notre architecture, elle forme le cœur du système de gestion des logs.
 
   - Elasticsearch: C’est la base de données de logs. Il stocke tous les logs de sécurité collectés et les rend accessibles pour la recherche ainsi que l'analyse en temps réel.
    
   - Logstash: Il agit comme un intermédiaire entre la source des logs et elasticsearch. Il collecte, filtre, transforme et envoie les données vers elasticsearch.
    
   - Kibana: Il représente l' Interface utilisateur de visualisation. Il fournit un tableau de bord et des outils d'exploration pour visualiser les logs, les tendances, et surtout, les anomalies et alertes détectées par Snort.

IMAGE


   ### 2. Filebeat
Filebeat est un agent léger de collecte de logs. Son rôle est de surveiller en temps réel les fichiers de journaux générés par les systèmes, applications ou outils de sécurité, puis de les transmettre vers Logstash ou directement vers Elasticsearch pour traitement et stockage.

   ### 3. Snort
Snort est un système de détection et de prévention d’intrusions (IDS/IPS) open source.
 
