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

![ELK](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/082918_1504_ELKStackTut1.webp)



   ### 2. Filebeat
Filebeat est un agent léger de collecte de logs. Son rôle est de surveiller en temps réel les fichiers de journaux générés par les systèmes, applications ou outils de sécurité, puis de les transmettre vers Logstash ou directement vers Elasticsearch pour traitement et stockage.

   ### 3. Snort
Snort est un système de détection et de prévention d’intrusions (IDS/IPS) open source.
Il analyse le trafic réseau en temps réel à la recherche de comportements suspects, d’attaques ou d’anomalies à l’aide de signatures et de règles définies.
Lorsqu’une menace est détectée, Snort génère une alerte qui est ensuite envoyée vers Filebeat, puis intégrée dans Elasticsearch pour être visualisée dans Kibana.

IMAGE

## III. Architecture d’implémentation

Pour parfaire à ce projet, nous avions mis en place une architecture virtualisée sous vmware avec les différente machines suivantes : 

 - Une machine/SE linux sur laquelle a été  déployée snort (pour la détection d’intrusion) et  ELK( Elasticsearch, Logstash et Kibana pour la collecte, gestion et visualisation des Logs)
 - Une machine de pénétration ou attaquante : Kali OS
 - Une machine simple ( windows / linux) (qui jouera le rôle de la cible en fonction du scénario)
 - Une machine serveur

IMAGE
 
## IV. Installation & configuration

  ### 1. ELK
  
Installer la suite Elastic (ELK) 8.x sur Ubuntu 22.04 LTS
(Y compris l’installation et la configuration de Filebeat pour l’envoi de logs)

Autrefois appelée ELK Stack, la suite Elastic est un ensemble d’outils puissants pour la gestion et l’analyse des logs : ElasticSearch (moteur d’analyse), Logstash (pipeline de traitement des données) et Kibana (outil de visualisation). Cette suite te permettra de visualiser des logs en quelques minutes.

Avant de commencer, quelques remarques :
 - Il s’agit d’une configuration de base, qui ne sera pas accessible publiquement, et qui ne sera pas sécurisée — elle sert uniquement à te familiariser avec les outils.
 - Nous allons installer la suite Elastic (ELK) et Filebeat sur des machines virtuelles séparées — la VM « Elastic Stack » est dédiée à la gestion/analyse des logs ; Filebeat sera installé sur le serveur depuis lequel tu veux envoyer les logs.

Allez, on y va !


### 1. Préparatifs & mise à jour

#sudo apt update && sudo apt upgrade -y

IMAGE

### 2. Installer Elasticsearch
   
 1- Télécharger et installer la clé publique d’Elasticsearch :
#wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

 2- Installer le paquet apt-transport-https pour permettre les téléchargements via HTTPS :
#sudo apt-get install apt-transport-https

IMAGE

 3- Ajouter le dépôt Elastic à ta liste de sources :
#echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

 4- Mettre à jour les paquets et installer Elasticsearch :
#sudo apt-get update && sudo apt-get install elasticsearch


IMAGE

***Attention*** : la sécurité d’Elasticsearch est activée par défaut. Le mot de passe, les certificats et les clés te seront affichés dans le terminal — il faut les sauvegarder quelque part. 
Comme ici on garde l’installation locale (non accessible depuis l’extérieur), on va désactiver les fonctions de sécurité.

 5- Modifier le fichier de configuration d’Elasticsearch :

   #sudo nano /etc/elasticsearch/elasticsearch.yml
   
     - Pour rendre le service accessible depuis n’importe quelle adresse réseau, changer network.host de localhost à 0.0.0.0

         IMAGE

