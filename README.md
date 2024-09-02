### **Centralisation des Logs pour une Entreprise en Croissance**

#### **Contexte d'Entreprise**

Vous faites partie d'une entreprise en pleine croissance spécialisée dans le développement de solutions logicielles pour des clients internationaux. Avec l'augmentation du nombre de clients et de services, la gestion des logs est devenue un enjeu crucial pour assurer la stabilité, la sécurité et l'optimisation des performances de l'infrastructure. Les logs sont essentiels pour la détection des incidents, le suivi des performances, et la conformité réglementaire.

L'entreprise a mandaté une équipe composée de 4 à 6 ingénieurs pour mettre en place une solution de centralisation des logs. L'objectif est de collecter et d'analyser les logs provenant de différentes sources, d'assurer leur stockage sécurisé et leur accessibilité en temps réel. Le tout doit être réalisé en utilisant les services d'AWS tout en restant dans les limites du Free Tier.

#### **Objectif du Projet**

Mettre en place un système de centralisation des logs en utilisant Rsyslog sur des instances EC2, avec une extension pour envoyer les logs vers Amazon CloudWatch pour une gestion et une surveillance améliorées. À la fin du projet, vous présenterez vos résultats et vos recommandations aux décideurs techniques de l'entreprise, qui décideront de la mise en production ou non de la solution proposée.

### **Le projet Eyes : Configuration d’un Système de Centralisation des Logs avec Rsyslog et CloudWatch sur AWS EC2**

**Durée :** 4 heures  
**Objectif :** Apprendre à configurer un serveur Rsyslog pour la collecte centralisée de logs à partir de différentes sources sur des instances EC2 et à intégrer CloudWatch pour la visualisation et l'analyse des logs.  
**Modalité de travail :** Par groupe  
**Notation :** 7 points (4 points pour la démonstration, 3 points pour la présentation)

----------

### **Prérequis :**

-   Compte AWS avec accès au Free Tier.

----------

### **Étape 1 : Configuration des Instances EC2**

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

### **Étape 4 : Centralisation des Logs avec AWS CloudWatch**

1.  **Configurer l'intégration de Rsyslog avec CloudWatch :**
    -   Sur l'instance `Rsyslog-Server`, installez l'agent CloudWatch Logs :
        `sudo apt-get install awslogs -y`
    -   Configurez l'agent pour envoyer les logs Rsyslog à CloudWatch :
        `sudo nano /etc/awslogs/awslogs.conf`
    -   Modifiez le fichier pour inclure le chemin du fichier de log `/var/log/remote_logs.log` :
    ```bash
    [/var/log/remote_logs.log]
    log_group_name = /aws/ec2/rsyslog
    log_stream_name = {instance_id}
    file = /var/log/remote_logs.log
    ```
    -   Redémarrez l'agent AWS Logs :
        `sudo systemctl restart awslogs`
    -   Assurez-vous que l'agent AWS Logs est configuré pour démarrer automatiquement au boot :
        `sudo systemctl enable awslogs`

2.  **Vérifiez que les logs sont envoyés à CloudWatch :**
    -   Allez dans la console AWS et accédez à CloudWatch.
    -   Sous **Logs**, vérifiez que le groupe de logs `/aws/ec2/rsyslog` contient les logs du fichier `/var/log/remote_logs.log`.

----------

### **Étape 5 : Validation, Conclusion et Recommandations**

1.  **Envoyer un message de log depuis le client :**
    -   Utilisez la commande suivante sur le client pour envoyer un message de test :
        `logger -p local0.notice "Test log message from $(hostname) client pour la team -nom ou numéro de votre team-"`
2.  **Vérifier la réception du log sur le serveur :**
    -   Consultez le fichier `/var/log/remote_logs.log` sur le serveur pour vérifier si le message du client a été enregistré :
        `cat /var/log/remote_logs.log`
3.  **Vérifier la réception du log dans CloudWatch :**
    -   Dans CloudWatch, assurez-vous que le log est bien apparu sous le groupe de logs configuré.
4.  **Rédiger une présentation pour les décideurs :**
    -   Préparez une présentation PPT qui présente :
        -   Une démonstration de la centralisation des logs via Rsyslog.
        -   L'intégration avec AWS CloudWatch pour une surveillance en temps réel.
        -   Les avantages de cette solution pour l'entreprise (facilité de détection des incidents, conformité, gestion centralisée).
        -   Les coûts engagés pour cette architecture.
        -   Les recommandations pour la mise en production.
        -   Si l'entreprise décide de passer à GCP ou à Cloud Azure, quelles sont les ressources nécessaires pour assurer le même fonctionnement que ce que vous proposez pour AWS ?
        -   Proposez les schéma d'architecture de la centralisation et du stockage des logs pour GCP et Azure en vous basant sur les ressources identifiées.

----------

### **Résultats Attendus**

-   Une infrastructure fonctionnelle de centralisation des logs avec Rsyslog sur AWS.
-   Une intégration réussie avec AWS CloudWatch pour une gestion et une visualisation en temps réel des logs.
-   Une présentation claire des résultats et des recommandations, avec un focus sur les bénéfices et les prochaines étapes pour l'entreprise.
