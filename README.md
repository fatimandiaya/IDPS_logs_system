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

![Snort](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/Snort.jpg)


## III. Architecture d’implémentation

Pour parfaire à ce projet, nous avions mis en place une architecture virtualisée sous vmware avec les différente machines suivantes : 

 - Une machine/SE linux sur laquelle a été  déployée snort (pour la détection d’intrusion) et  ELK( Elasticsearch, Logstash et Kibana pour la collecte, gestion et visualisation des Logs)
 - Une machine de pénétration ou attaquante : Kali OS
 - Une machine simple ( windows / linux) (qui jouera le rôle de la cible en fonction du scénario)
 - Une machine serveur

![Archi](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/archi.png)
 
## IV. Installation & configuration

  ### 1. ELK
  
**Installer la suite Elastic (ELK) 8.x sur Ubuntu 22.04 LTS**

Autrefois appelée ELK Stack, la suite Elastic est un ensemble d’outils puissants pour la gestion et l’analyse des logs : ElasticSearch (moteur d’analyse), Logstash (pipeline de traitement des données) et Kibana (outil de visualisation). Cette suite te permettra de visualiser des logs en quelques minutes.

Avant de commencer, quelques remarques :
 - Il s’agit d’une configuration de base, qui ne sera pas accessible publiquement, et qui ne sera pas sécurisée — elle sert uniquement à te familiariser avec les outils.
 - Nous allons installer la suite Elastic (ELK) et Filebeat sur des machines virtuelles séparées — la VM « Elastic Stack » est dédiée à la gestion/analyse des logs ; Filebeat sera installé sur le serveur depuis lequel tu veux envoyer les logs.

Allez, on y va !


### 1. Préparatifs & mise à jour

#sudo apt update && sudo apt upgrade -y

![5](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/5.png)

### 2. Installer Elasticsearch
   
 1- Télécharger et installer la clé publique d’Elasticsearch :
#wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

 2- Installer le paquet apt-transport-https pour permettre les téléchargements via HTTPS :
#sudo apt-get install apt-transport-https

![2](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/2.png)

 3- Ajouter le dépôt Elastic à ta liste de sources :
#echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

 4- Mettre à jour les paquets et installer Elasticsearch :
#sudo apt-get update && sudo apt-get install elasticsearch

![6](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/6.png)


***Attention*** : la sécurité d’Elasticsearch est activée par défaut. Le mot de passe, les certificats et les clés te seront affichés dans le terminal — il faut les sauvegarder quelque part. 
Comme ici on garde l’installation locale (non accessible depuis l’extérieur), on va désactiver les fonctions de sécurité.

 5- Modifier le fichier de configuration d’Elasticsearch :
   
 #sudo nano /etc/elasticsearch/elasticsearch.yml
   
  - Pour rendre le service accessible depuis n’importe quelle adresse réseau, changer network.host de localhost à 0.0.0.0

    ![8](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/8.png)

  - Désactiver la sécurité (changer de true à false) :

   ![7](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/7.png)

 6- Activer et démarrer Elasticsearch :
#sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch
#sudo systemctl status elasticsearch


![9](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/9.png)


 7- Tester l’API d’Elasticsearch :
#curl -XGET "localhost:9200"

![10](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/10.png)

Tu devrais obtenir une réponse JSON contenant des informations sur la version, le cluster, etc.
Si tu n’as pas cette réponse, vérifie la configuration et les règles du pare-feu (le port 9200 doit être autorisé).


### 3. Installer Logstash
#sudo apt install logstash

![11](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/11.png)


Pour que Logstash traite correctement les logs que tu enverras via Filebeat, il faut configurer des filtres.
Sinon, tu risques de voir dans Kibana que tu as des données, mais vides.

 1. Créer un fichier de filtre :
#sudo nano /etc/logstash/conf.d/beats.conf

 2. Activer et démarrer Logstash :
#sudo systemctl enable logstash && sudo systemctl start logstash
#sudo systemctl status logstash

![12](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/12.png)

Si le pare-feu est actif, autorise le port 5044 pour que Logstash reçoive les logs.

### 4. Installer Kibana
#sudo apt install kibana

![13](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/13.png)

 1- Modifier le fichier de configuration :
 #sudo nano /etc/kibana/kibana.yml
 -Décommente et ajuste :
 -server.port: 5601
 -server.host: "0.0.0.0" 
 -elasticsearch.hosts: ["http://localhost:9200"]

 2- Activer et démarrer Kibana :
#sudo systemctl enable kibana && sudo systemctl start kibana
#sudo systemctl status kibana

![14](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/14.png)

Si tout s’est bien passé, tu peux naviguer vers l’adresse IP de ton serveur ELK dans un navigateur : tu verras la page d’accueil de Kibana.

![15](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/15.png)

### 5. Installer Filebeat sur le serveur source de logs

Sur la machine dont tu veux envoyer les logs :
 
  1- Télécharger et installer Filebeat :
     #curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.9.2-amd64.deb

    ![16](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/16.png)

   #sudo dpkg -i filebeat-8.9.2-amd64.deb
     
  2- Ouvrir le fichier de configuration :
    #sudo nano /etc/filebeat/filebeat.yml

 ![FLB](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/FLB.png)

 Comme tu vas envoyer les logs à Logstash (et non directement à Elasticsearch), commente la section output.elasticsearch :

 ![FLB2](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/FLB2.png)

 décommente / active la section output.logstash en indiquant l'adresse IP de ton serveur ELK (au lieu de localhost) :
 
**output.logstash:**
  **hosts: ["192.168.50.130:5044"]**
  
 3- Activer et démarrer Filebeat :
    #sudo systemctl enable filebeat && sudo systemctl start filebeat
    #sudo systemctl status filebeat


 ![17](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/17.png)

 4- Filebeat fournit des modules prédéfinis pour divers services (Apache, système, etc). Pour les visualiser :
 
#sudo filebeat modules list

Pour nous, on va activer les modules system et apache :

#sudo filebeat modules enable system
#sudo filebeat modules enable apache

Chaque module correspond à un fichier de configuration dans /etc/filebeat/modules.d/. Ouvre-les et mets enabled: true si ce n’est pas déjà fait. (pense aux questions de format, crochets, guillemets, etc.)

 ![FLB3](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/FLB3.png)

 5- Charger les modèles d’index dans Elasticsearch (en utilistant l’IP de ton serveur ELK) :
    
  #sudo filebeat setup --index-management -E output.logstash.enabled=false -E 'output.elasticsearch.hosts=["192.168.40.130:9200"]'


 ![19](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/19.png)

 6- Charger les tableaux de bord (dashboards) dans Kibana 
   
   #sudo filebeat setup -E output.logstash.enabled=false -E output.elasticsearch.hosts=['192.168.40.130:9200'] -E setup.kibana.host=192.168.40.130:5601

 ![20](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/20.png)

 7- Résultat final
 
Si tout s’est bien déroulé, connecte-toi à l’interface Kibana via le navigateur à l’adresse du serveur ELK, dans la section **Discover**, tu devrais voir les données arriver. Tu peux aussi visualiser les flux en direct via le panneau **Observability**.

 ### 3. Snort
 
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
 
 4- Installation de snort 2.9.15.1

Pour installer Snort, nous allons effectuer les mêmes actions que nous avons effectuées lors de l’installation précédente. À partir du terminal, nous allons exécuter les commandes suivantes :

**# cd ~/snort_src/
#wget https://snort.org/downloads/snort/snort-2.9.15.1.tar.gz
#tar -xvzf snort-2.9.15.1.tar.gz
#cd snort-2.9.15.1/
#./configure --enable-sourcefire
#make && make install**
 
Ensuite, nous allons créer des fichiers et dossiers dont Snort aura besoin lorsqu'il sera exécuté. Nous attribuons la propriété de ces fichiers à l’utilisateur snort.

**-/etc/snort**, pour les fichiers de configurations de Snort

**-/etc/ snort/rules et /usr/local/lib/snort_dynamicrules**, pour les règles de Snort **-/var/log/snort** : dossier de journalisation de Snort.

Pour cela on exécute les commandes suivantes depuis un terminal :
 
**#mkdir /etc/snort
#mkdir /etc/snort/rules
#mkdir /usr/local/lib/snort_dynamicrules
#mkdir /var/log/snort**

On crée les fichiers qui vont nous permettre d’éditer nos propres règles pour Snort avec les commandes ci-contre :
 
**#touch /etc/snort/rules/white_list.rules
#touch /etc/snort/rules/black_list.rules /etc/snort/rules/local.rules**

Nous allons attribuer les permissions d’écriture, de lecture et d’exécution sur les répertoires et fichiers créés :
 
**#chmod -R 777 /etc/snort
#chmod -R 777 /var/log/snort
#chmod -R 777 /usr/local/lib/snort_dynamicrules**

On copie tous les fichiers ayant pour extension .conf et .map du répertoire **‶~/snort_src/snort- 2.9.15.1/etc/″** vers **‶/etc/snort/″** (le répertoire de configuration de snort que nous venons de créer) avec les commande ci-dessous :
 
**#cp ~/snort_src/snort-2.9.15.1/etc/*.conf* /etc/snort
#cp ~/snort_src/snort-2.9.15.1/etc/*.map /etc/snort**

Maintenant, nous allons vérifier si Snort est bien installé avec la commande **‶snort -V″** depuis le terminal :

 ![21](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/21.png)

4.1- Installation des règles

Les règles de Snort sont des instructions ou codes qui décrivent les signatures des virus, des sites web malveillants, des logiciels dangereux, des intrusions, des attaques et des paquets suspects. Pour installer les règles Snort, nous devons nous inscrire sur le site officiel de Snort https://www.snort.org ou plus précisément à l’URL https://www.snort.org/users/sign_up. Ensuite, nous serons en mesure de télécharger les règles pour la configuration de Snort. Une fois inscrit sur le site, nous pouvons télécharger les règles et continuer l’installation. Nous allons ensuite le décompresser dans /etc/snort pour que les répertoires et fichiers nécessaires soient directement disponibles pour Snort. Avec la commande suivante, nous allons décompresser (commande tar) le fichier téléchargé et le mettre dans /etc/snort (l’utilisation de l’option -C puis en indiquant le répertoire de destination) :

**# tar -xvzf snortrules-snapshot-29110.tar -C /etc/snort**
 
4.2-  Installation de Oinkmaster

Oinkmaster est un script écrit avec le langage de programmation PERL par l’éditeur du logiciel qui va nous servir à mettre jour les fichiers de règles qui sont présents dans /etc/snort/rules. Pour permettre à Oinkmaster de télécharger les règles de Snort, nous avons besoin de le configurer en synchronisation avec le site officiel de Snort. Pour cela nous avons besoin d’une clé appelé « Oink code » sur le site de Snort. Après enregistrement notre « Oink code » est .Nous allons à présent mettre en place Oinkmaster.

 ![23B](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/23B.png)


Nous allons vérifier si Oinkmaster est bien installé avec la commande **oinkmaster.pl -V**

**#oinkmaster.pl -V**

 ![24](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/24%20OINKMAKER.png)

Nous allons éditer le fichier :

**#vi /etc/oinkmaster.conf**

 ![25](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/25.png)


4.3 - Configuration de Snort 

4.3.1- Configuration de snort en mode IDS :

Pour configurer snort en mode Network Intrusion Detection System, nous allons effectuer quelques modifications dans le principal fichier de configuration de snort qui est snort.conf qui se trouve dans /etc/snort/snort.conf.

**#vi /etc/snort/snort.conf**
 
Nous allons modifier les lignes suivantes pour informer snort des différents réseaux :

**ü ipvar HOME_NET any en ipvar HOME_NET 192.168.50.0/24**: pour définir le réseau
que nous allons protéger. Dans cet exemple c'est le réseau 192.168.50.0/24, le /24 pour
préciser le masque de sous réseau.

**ü ipvar EXTERNAL_NET any en ipvar EXTERNAL_NET !$HOME_NET** : pour
indiquer à snort que tout hôte dont l'adresse IP est différente de l'adresse du réseau
protégé est un hôte du réseau externe.
Notons que nous avons la possibilité de définir les adresses des différents serveurs s’ils sont différents du **‘’$HOME_NET’’**.

 ![27](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/27.png)


Notons que nous avons la possibilité de définir les adresses des différents serveurs s’ils sont différents du **‘’$HOME_NET’’**.

**Test de la configuration du fichier de configuration de Snort**

Pour vérifier si nos configurations sont bonnes, nous allons tester Snort avec la commande suivante :

**#snort -T -c /etc/snort/snort.conf**

Si tout va bien, on aura un résultat semblable à celui -ci où nous avons le message ‶snort sccessfully valided the configuration! ″ pour nous informer que nous avons bien configurer Snort.

  ![Image5](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/Image5.png)


 4.3.2- Test du mode IDS:
 
Pour la réalisation de ce test, nous allons déclarer les protocoles ICMP, Telnet et FTP, comme protocoles suspects afin de tester le système de détection mis en place.

Nous allons ajouter ces règles dans /etc/snort/rules/local.rules :


 ![28](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/28.png)


On enregistre puis on quitte, ensuite on démarre Snort et les outils complémentaires.
Sur la machine du pirate (192.168.40.129) nous allons dans le terminal pour générer les paquets ICMP, Telnet et FTP pour tester le fonctionnement de Snort.
Pour détecter les alertes dans la console par Snort, on tape dans le terminal de VM1 (machine sur laquelle nous avons installé Snort) la commande suivante :

#Snort -A console -q -c  /etc/snort/snort.conf -I ens33

 ![29](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/29.png)


Sur la figure, nous remarquons que Snort a détecté les paquets Telnet avec le message ‶Telnet connexion″. Nous pouvons conclure que le mode IDS de Snort fonctionne correctement.

 4.3.3- Configuration de  Snort en mode IPS
Pour configurer Snort en tant que IPS, nous avons besoin d'apporter quelques modifications dans snort.conf. Mais avant, il faut vérifier les versions de DAQ installées avec la commande cicontre :

**#snort --daq-list**
Nous allons ajouter les lignes ciaprès au fichier de configuration de snort (snort.conf)

 ![31](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/31.png)

Nous allons vérifier la configuration du mode inline (IPS) :

**#snort -T -c /etc/snort/snort.conf -Q -i ens32:ens33**
 4.3.4- Test du mode IPS:
Pour le test du mode IPS on pourra bien simuler une attaque Denial of Service (DoS), mais l’inconvénient est que nous ne pourrons pas l’illustrer de façon claire et concis dans le document. Pour cela nous allons réaliser le test du mode IPS en reprenant le test précèdent en stoppant les paquets ICMP, Telnet et FTP.
Toujours dans /etc/snort/rules/local.rules, on ajoute les lignes suivantes :

 ![32](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/32.png)

On enregistre puis on quitte, ensuite on redémarre Snort et les outils complémentaires. Sur la machine du pirate (192.168.40.129) nous allons dans le terminal pour générer les paquets ICMP pour tester le fonctionnement de Snort

 ![33](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/33.png)


**#snort -A console -q -c /etc/snort/snort.conf -I ens32:ens33**

 ![34](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/34.png)

### V.  Scénarios
Ces cinq scénarios d’attaque ont été sélectionnés pour leur pertinence opérationnelle, leur diversité technique, et leur capacité à illustrer les capacités de détection et de corrélation d’un pipeline SOC local. Chaque scénario cible une classe de menace différente, permettant de tester :
-La détection en temps réel par l’IDS/IPS
-La collecte et centralisation des logs via syslog-ng/Logstash
-La visualisation et l’analyse dans Kibana

 ![34](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/5_scenarios.png)

 **Scénario 1 — Scan de réseau avec Nmap**
 
Simuler une tentative de reconnaissance réseau en utilisant l’outil Nmap pour identifier les ports ouverts et les services actifs sur la machine cible (DVWA).
Ce type de scan est souvent utilisé en phase préliminaire d’une attaque pour cartographier les services exposés.
Le scan a été lancé depuis la machine Kali vers DVWA avec la commande suivante : “ nmap -sS -p- 192.168.50.20
-sS : scan SYN furtif
-192.168.50.20 : IP de la cible DVWA

Sur la machine Kali, nous avons lancé un scan réseau avec Nmap afin d’identifier les hôtes actifs et les ports ouverts :
![S11](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/nmap1.png)

Snort a bien intercepté les paquets SYN répétés et a généré des alertes dans le fichier de logs "alert_json.txt"
![S12](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/nmap2.png)

Dans Kibana, on retrouve cette alerte avec le message “Scan Nmap SYN détecté”, l’IP source de Kali, et l’IP cible DVWA.
![S13](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/nmap3.png)
Cela confirme que le scan a été détecté comme une tentative de reconnaissance réseau, grâce à la règle sid:1000101.
![S14](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/nmap4.png)


**Scénario 2 :  Attaque par brute-force SSH**

Un cybercriminel effectue une tentative d’accès non autorisé à un serveur via le protocole SSH, en utilisant une attaque par brute-force. Ce type d’attaque consiste à tester un grand nombre de combinaisons de login/mot de passe dans un temps court, dans le but de compromettre un compte valide. Outil utilise : Hydra

D’abord nous avions procédé à l’ installation de  ssh sur notre IDS/SIEM
**#sudo apt update
#sudo apt install openssh-server
#Sudo systemctl start ssh
#Sudo systemctl enable ssh**

![35](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/ssh1.png)


Ensuite depuis kali nous avons exécuté : 
**“ hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.50.20”**

![36](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/ssh2.png)
Snort a détecté un nombre élevé de connexions SSH en peu de temps et a généré des alertes.
![S21](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/ssh3.png)
Sur Kibana, on voit clairement l’alerte “Brute-force SSH détecté”, avec l’IP source de Kali et le port 22 ciblé.
![S22](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/ssh4.png)
La règle sid:1000201 s’est déclenchée après 10 connexions en moins de 60 secondes, ce qui valide la détection.
![S22](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/ssh5.png)




**Scénario 3 : DOS**
L’attaquant effectue une attaque par déni de service (DoS) en saturant la table de connexions TCP du serveur cible avec des paquets SYN non complétés. Cette technique vise à épuiser les ressources du serveur en initiant un grand nombre de connexions TCP sans jamais les finaliser (absence de ACK), ce qui empêche les connexions légitimes d’être établies.

**Méthode d’attaque** : L’outil utilisé est hping3, configuré pour envoyer un flot rapide de paquets SYN vers le port 80 du serveur cible :

**Commande utilisée**
**“ sudo hping3 -S -p 80 -i u100 -c 10000 192.168.50.20 “**

**Paramètres** :
-S : envoie des paquets TCP avec le flag SYN
-p 80 : cible le port HTTP
-i u100 : intervalle de 100 microsecondes entre les paquets (très rapide)
-c 10000 : envoie 10 000 paquets
**192.168.50.20 : IP de la machine cible**

Sur cette première capture, on voit qu'on a exécuté la commande hping3 depuis Kali. Le terminal affiche l’envoi massif de paquets SYN vers le port 80 de DVWA.
![S31](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/dos1.png)
Ensuite, dans le fichier alert_json.txt sur la machine IDS, on voit que Snort3 a bien généré une alerte avec le message “Déni de service - SYN flood détecté”
![S32](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/dos2.png)
Sur Kibana, l’alerte est bien visible. On retrouve l’IP source de Kali, l’IP cible DVWA, le port 80, et le SID de la règle.
![S33](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/dos3.png)
Cela confirme que le trafic SYN non complété a été détecté comme une tentative de saturation TCP, typique d’un DoS.
![S34](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/dos4.png)
Le serveur n’a pas répondu aux paquets SYN envoyés par l’attaquant, car ceux-ci n’étaient pas suivis d’un ACK, empêchant l’établissement complet des connexions TCP. Ce comportement a été intercepté par Snort, qui a reconnu le flot massif de paquets SYN non finalisés comme une attaque par déni de service. La visualisation dans Kibana permet ensuite de confirmer l’origine de l’attaque, le type de trafic observé, ainsi que la règle de détection déclenchée.


**Scénario 4 : Détection de malware (EICAR)**

L’attaque simule l’introduction d’un fichier malveillant dans le système cible afin d’évaluer la capacité de l’IDS à détecter une signature connue de malware. Pour ce test, nous utilisons le fichier EICAR, développé par l’European Institute for Computer Antivirus Research. Ce fichier est un standard international utilisé pour tester les antivirus et les systèmes de détection, sans présenter de danger réel pour les systèmes.
*Méthode utilisée :  depuis notre VM Kali envoie directement la signature ASCII du fichier EICAR vers le port HTTP de la cible DVWA en utilisant **netcat :
*“ echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' | nc 192.168.50.20 80 "**

Cette commande simule un transfert réseau contenant une charge malveillante, sans passer par un navigateur ou un fichier physique.

![S41](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/eicar1.png)

Lors de l’envoi de la signature EICAR via le réseau , le serveur Apache retourne une erreur **HTTP 400 Bad Request** car la requête n’est pas conforme au protocole HTTP (pas de méthode *GET, ni d’en-têtes). Ce comportement est typique d’un attaquant qui injecte directement une charge malveillante dans le flux réseau. 

Malgré cette erreur côté serveur, Snort3 détecte la signature EICAR dans le contenu brut du paquet TCP, prouvant que la détection IDS ne dépend pas de la validité HTTP, mais bien de l’analyse du trafic réel.

![S42](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/eicar2.png)

Dans Kibana, on retrouve l’alerte générée par Snort avec le message “Malware détecté — Fichier EICAR”. Les métadonnées que nous avons configurées précédemment permettent d’afficher clairement l’IP source (la machine Kali), l’IP cible (DVWA), le port 80 utilisé pour l’attaque, ainsi que le SID de la règle déclenchée. Cela confirme que la détection fonctionne correctement et que l’alerte est bien remontée dans la chaîne de logs.
![S43](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/eicar3.png)
![S44](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/eicar4.png)

**Scénario 5 : injection SQL**

Dans ce scénario, j’ai simulé une attaque par injection SQL dans un environnement volontairement vulnérable afin de tester la capacité de notre pipeline Snort + ELK à détecter ce type de menace. Comme pour les autres scénarios, la cible reste la machine Ubuntu-SRV, mais cette fois-ci, l’attaque vise spécifiquement DVWA (Damn Vulnerable Web Application), une application web conçue pour l’apprentissage et la simulation d’attaques. DVWA est déployée sur la VM Ubuntu-SRV au sein de notre topologie EVE-NG.
Pour ce scénario, on a utilisé deux types d’injection SQL : contournement d’authentification et reconnaissance de schéma
  - 1 Contournement d’authentification via OR 1=1#
l'objectif de l’attaque est de simuler une tentative de bypass d’authentification en injectant une condition SQL toujours vraie dans le champ id. Ainsi l'attaquant contourne la vérification des identifiants et d’accéder à l’application sans mot de passe valide.

Sur kali, on accede a l'interface de DVWA , puis sur la section "SQL injectio" et au niveau du champ iput "user ID" on va injecté le payload : `1' OR 1=1#`
![S51](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql1.png)

Sur DVWA, la requête renvoie tous les enregistrements de la table des utilisateurs, contournant ainsi l'authentification ou les contrôles d'accès.
![S52](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql2.png)

Snort a intercepté la requête encodée par le navigateur  et a généré une alerte dans le fichier alert_json.txt, confirmant que la tentative d’injection a bien été détectée.
![S53](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql3.png)

Dans Kibana, on retrouve l’alerte avec le message “Injection SQL — OR 1=1 détectée”, l’IP source de Kali, l’IP cible DVWA, le port HTTP, et le SID de la règle.
![S54](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql4.png)
![S54](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql5.png)
Cela montre que notre pipeline Snort + ELK est capable de détecter les injections même lorsqu’elles sont encodées côté client.

- 2. Reconnaissance de schéma via “ SHOW TABLES “
L'objectif de l’attaque est d’injecter la commande SQL SHOW TABLES dans un champ input pour afficher la structure de la base de données cible.

Toujours au niveau du champ input "user ID" on injecte la commande ` 1``SHOW TABLES--`
![S55](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql6.png)
Snort a détecté cette tentative d’exploration de la base de données et a généré une alerte spécifique dans alert_json.txt.
![S56](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql7.png)

Sur Kibana, l’alerte apparaît avec le message “Injection SQL — SHOW TABLES détectée”, les IPs source et destination, et le SID correspondant.

![S57](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql8.png)
![S57](https://github.com/fatimandiaya/IDPS_logs_system/blob/main/Images/sql9.png)
Cette détection confirme que notre IDS est capable a la fois capables d’intercepter des requêtes SQL orientées reconnaissance, et les bypass d’authentification.




### VI.  Conclusion
L'objectif principal de ce projet était de concevoir et de mettre en œuvre une solution complète de surveillance , en fusionnant les capacités de la Pile ELK pour la gestion centralisée des logs et de l'outil Snort pour la détection et la prévention d'intrusion.

Nous avons réussi à établir un environnement où Filebeat assure la collecte des journaux d'activité et leur alimentation pour les ingérer dans la Pile ELK, permettant ainsi leur analyse et leur exploitation visuelle via Kibana. L'efficacité de notre solution a été validée par la configuration de Snort, et par des tests concrets simulant différents scénarios d'attaques, tels que le brute force SSH, la détection de malware EICAR, le scan de réseau, l'injection SQL et la tentative de DoS. Ces démonstrations ont confirmé la capacité de l'architecture à identifier les comportements malveillants et à réagir en conséquence. 
Pour l'avenir, les perspectives d'amélioration majeures résident dans l'intégration de techniques d'apprentissage automatique pour affiner la détection d'anomalies comportementales et l'ajout d'outils de gestion des vulnérabilités afin d'enrichir le contexte des alertes. 

En définitive, ce projet fournit une fondation solide et fonctionnelle pour une stratégie de cyberdéfense proactive, essentielle pour la résilience de toute infrastructure face aux menaces numériques contemporaines.



