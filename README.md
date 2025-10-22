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

  - Désactiver la sécurité (changer de true à false) :

     IMAGE

 6- Activer et démarrer Elasticsearch :
#sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch
#sudo systemctl status elasticsearch


IMAGE


 7- Tester l’API d’Elasticsearch :
#curl -XGET "localhost:9200"

IMAGE

Tu devrais obtenir une réponse JSON contenant des informations sur la version, le cluster, etc.
Si tu n’as pas cette réponse, vérifie la configuration et les règles du pare-feu (le port 9200 doit être autorisé).


### 3. Installer Logstash
#sudo apt install logstash

IMAGE


Pour que Logstash traite correctement les logs que tu enverras via Filebeat, il faut configurer des filtres.
Sinon, tu risques de voir dans Kibana que tu as des données, mais vides.

 1. Créer un fichier de filtre :
#sudo nano /etc/logstash/conf.d/beats.conf

 3. Activer et démarrer Logstash :
#sudo systemctl enable logstash && sudo systemctl start logstash
#sudo systemctl status logstash

  IMAGE

Si le pare-feu est actif, autorise le port 5044 pour que Logstash reçoive les logs.

### 4. Installer Kibana
#sudo apt install kibana

IMAGE

 1- Modifier le fichier de configuration :
 #sudo nano /etc/kibana/kibana.yml
 -Décommente et ajuste :
 -server.port: 5601
 -server.host: "0.0.0.0" 
 -elasticsearch.hosts: ["http://localhost:9200"]

 2- Activer et démarrer Kibana :
#sudo systemctl enable kibana && sudo systemctl start kibana
#sudo systemctl status kibana

IMAGE

Si tout s’est bien passé, tu peux naviguer vers l’adresse IP de ton serveur ELK dans un navigateur : tu verras la page d’accueil de Kibana.

IMAGE

### 5. Installer Filebeat sur le serveur source de logs

Sur la machine dont tu veux envoyer les logs :
 
  1- Télécharger et installer Filebeat :
     #curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.9.2-amd64.deb

     IMAGE


   #sudo dpkg -i filebeat-8.9.2-amd64.deb
     
  2- Ouvrir le fichier de configuration :
    #sudo nano /etc/filebeat/filebeat.yml

 IMAGE

 Comme tu vas envoyer les logs à Logstash (et non directement à Elasticsearch), commente la section output.elasticsearch :

 IMAGE

 décommente / active la section output.logstash en indiquant l'adresse IP de ton serveur ELK (au lieu de localhost) :
 
**output.logstash:**
  **hosts: ["192.168.50.130:5044"]**
  
 3- Activer et démarrer Filebeat :
    #sudo systemctl enable filebeat && sudo systemctl start filebeat
    #sudo systemctl status filebeat


IMAGE

 4- Filebeat fournit des modules prédéfinis pour divers services (Apache, système, etc). Pour les visualiser :
 
#sudo filebeat modules list

Pour nous, on va activer les modules system et apache :

#sudo filebeat modules enable system
#sudo filebeat modules enable apache

Chaque module correspond à un fichier de configuration dans /etc/filebeat/modules.d/. Ouvre-les et mets enabled: true si ce n’est pas déjà fait. (pense aux questions de format, crochets, guillemets, etc.)

IMAGE

 5- Charger les modèles d’index dans Elasticsearch (en utilistant l’IP de ton serveur ELK) :
    
    #sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["192.168.40.130:9200"]'


IMAGE 

 6- Charger les tableaux de bord (dashboards) dans Kibana 
   
   #sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['192.168.40.130:9200'] -E setup.kibana.host=192.168.40.130:5601

IMAGE

 7- Résultat final
 
Si tout s’est bien déroulé, connecte-toi à l’interface Kibana via le navigateur à l’adresse du serveur ELK, dans la section **Discover**, tu devrais voir les données arriver. Tu peux aussi visualiser les flux en direct via le panneau **Observability**.

 3- Snort
 
 Snort a besoin d’un certain nombre de paquets pour fonctionner. Certains se trouvent dans les dépôts et d’autres devront être installés manuellement en téléchargeant les sources et en les compilant.
 
 1- Mise à jour du système
#sudo apt update && sudo apt upgrade -y

 2- Installer les dépendances nécessaires
Snort a besoin de plusieurs outils et bibliothèques :
Pour installer l’ensemble de ces packages, nous allons, depuis le terminal, exécuter la commande suivante :

 #apt-get install mysql mysql-bench mysql-server mysql-devel mysqlclient10 php-mysql httpd pcre-devel php-gd gd mod_ssl glib2-devel gcc-c++ libpcap-devel php php-pear apt-get-utils gcc flex bison zlib* libpcap* pcre* libdnet libdnet-devel tcpdump
  
Une fois l’installation des prérequis de Snort terminée, nous allons effectuer l’installation de la bibliothèque d’acquisition de données.

 3- Installation de la bibliothèque d’acquisition de données (DAQ)
 
Snort utilise la bibliothèque d'acquisition de données (DAQ) pour extraire les appels vers les bibliothèques de capture de paquets. Pour commencer l'installation, nous allons créer un dossier d'installation qui contiendra nos fichiers tarball (fichiers d’archives) téléchargés. À partir d’un terminal nous allons télécharger et installer le paquet de DAQ. Création du répertoire ‶snort_src/″

 #mkdir ~/snort_src

On entre dans le répertoire créé avec la commande suivante :

 #cd ~/snort_src/

Ensuite on télécharge le daq-2.0.6 avec la commande ci-contre :

 #wget https://snort.org/downloads/snort/daq-2.0.6.tar.gz

On constate que le fichier téléchargé est un fichier compressé (extension.tar.gz), pour cela on le décompresse avec la commande ci-dessous :
 
 #tar -xvzf daq-2.0.6.tar.gz
 
Et enfin on procède à l’installation du daq-2.0.6 avec les commandes suivantes :
 #cd daq-2.0.6/
 #./configure
 #make && make install
  
 Maintenant que nous avons fait le nécessaire en installant les prérequis de Snort, nous allons installer snort-2.9.15.1
 

