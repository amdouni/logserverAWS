### **Lab Pratique : Configuration d’un Serveur de Logs avec Rsyslog sur AWS EC2**

**Durée :** 3 heures  
**Plateforme :** AWS EC2 (Free Tier)  
**Objectif :** Apprendre à configurer un serveur Rsyslog pour la collecte centralisée de logs à partir de différentes sources sur des instances EC2.

----------

### **Prérequis :**

-   Compte AWS avec accès au Free Tier.

----------

### **Étape 1 : Configuration des instances EC2**

1.  **Lancer deux instances EC2 :**
    
    -   Accédez à l'AWS Management Console.
    -   Allez dans la section **EC2** et cliquez sur **Launch Instance**.
    -   Configurez deux instances EC2 Ubuntu 20.04 (Free Tier eligible).
    -   **Instance 1 :** Nom de l'instance : `Rsyslog-Server`
    -   **Instance 2 :** Nom de l'instance : `Rsyslog-Client`
    -   Sélectionnez le type d'instance **t2.micro**.
    -   Créez une nouvelle paire de clés ou utilisez une paire existante pour SSH.
    -   Configurez le **Security Group** :
        -   Autorisez le trafic SSH (port 22).
        -   Autorisez le trafic UDP (port 514) entre les deux instances.
    -   Lancez les instances.
2.  **Obtenir l'adresse IP des instances :**
    
    -   Après le lancement, obtenez les adresses IP publiques des deux instances.
3.  **Connectez-vous aux instances :**
    
    -   Utilisez SSH pour vous connecter aux instances depuis votre terminal :
        
        `ssh -i "votre-paire-de-clés.pem" ubuntu@IP_DU_SERVEUR`
        
        `ssh -i "votre-paire-de-clés.pem" ubuntu@IP_DU_CLIENT` 
        

----------

### **Étape 2 : Installation et Configuration de Rsyslog sur le Serveur**

1.  **Installer Rsyslog sur le serveur :**
    
    -   Connectez-vous à l'instance `Rsyslog-Server`.
    -   Exécutez les commandes suivantes pour installer Rsyslog :
        
        `sudo apt-get update`

        `sudo apt-get install rsyslog -y` 
        
2.  **Configurer Rsyslog pour recevoir des logs via UDP :**
    
    -   Ouvrez le fichier de configuration de Rsyslog :
        
        `sudo nano /etc/rsyslog.conf` 
        
    -   Décommentez les lignes suivantes pour permettre la réception des logs via UDP :
        
        `module(load="imudp")`

        `input(type="imudp" port="514")` 
        
    -   Configurez la sortie des logs reçus dans un fichier dédié en ajoutant à la fin du fichier :
        
        `*.* /var/log/remote_logs.log` 
        
    -   Enregistrez et fermez le fichier (`Ctrl + O`, `Enter`, `Ctrl + X`).
3.  **Redémarrez le service Rsyslog :**
    
    `sudo systemctl restart rsyslog` 
    
4.  **Configurer le firewall (UFW) pour autoriser le trafic UDP sur le port 514 :**
    
    -   Installez et configurez UFW :
        
        `sudo apt-get install ufw -y`

        `sudo ufw allow 514/udp`

        `sudo ufw enable` 
        

----------

### **Étape 3 : Configuration du Client pour Envoyer des Logs**

1.  **Installer Rsyslog sur le client :**
    
    -   Connectez-vous à l'instance `Rsyslog-Client`.
    -   Installez Rsyslog :
        
        `sudo apt-get update`

        `sudo apt-get install rsyslog -y` 
        
2.  **Configurer Rsyslog sur le client pour envoyer des logs au serveur :**
    
    -   Ouvrez le fichier de configuration de Rsyslog sur le client :
        
        `sudo nano /etc/rsyslog.conf` 
        
    -   Ajoutez la configuration suivante à la fin du fichier pour envoyer les logs au serveur Rsyslog (remplacez `SERVER_IP` par l'adresse IP de votre serveur) :
        
        `*.* @SERVER_IP:514` 
        
    -   Enregistrez et fermez le fichier (`Ctrl + O`, `Enter`, `Ctrl + X`).
3.  **Redémarrez le service Rsyslog :**
    
    `sudo systemctl restart rsyslog` 
    
----------

### **Étape 4 : Validation et Conclusion**

1.  **Envoyer un message de log depuis le client :**
    
    -   Utilisez la commande suivante sur le client pour envoyer un message de test :
        
        `logger -p local0.notice "Test log message from $(hostname) client"` 
        
2.  **Vérifier la réception du log sur le serveur :**
    
    -   Revenez sur le serveur et consultez le fichier `/var/log/remote_logs.log` pour vérifier si le message du client a été enregistré :
        
        `cat /var/log/remote_logs.log` 
        
    -   Vous devriez voir le message envoyé depuis le client.
