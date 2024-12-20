# SAE502 - Supervision Réseaux

## Description  

Ce projet a pour objectif de déployer une infrastructure de supervision réseau entièrement automatisée.  
Grâce à cet environnement, il est possible de :  
- Monitorer des services.  
- Collecter des métriques.  
- Gérer des alertes avec **Grafana**, **Prometheus** et **Alertmanager**.  

Le projet les technologies  telles que **Docker** et **Ansible**, garantissant une installation rapide et une configuration reproductible.  

---

## Technologies utilisées  
- **Ansible** : Automatisation des déploiements.  
- **Docker** : Gestion des conteneurs.  
- **Prometheus** : Collecte et surveillance des métriques.  
- **Grafana** : Visualisation des données.  
- **Alertmanager** : Gestion des alertes.  

---


## Résultat du Projet : Ce que vous obtiendrez

Avec ce projet, vous déploierez un environnement de supervision complet comprenant les services suivants :  

### 1. **Serveur Grafana**
Un serveur Grafana configuré avec plusieurs dashboards permettant de visualiser :  
- L'état de l'espace de stockage  
- L'utilisation du CPU  
- Les connexions TCP  
- L'utilisation de la mémoire  

Ces dashboards vous offrent une vue d'ensemble de vos systèmes et conteneurs en temps réel.  

### 2. **Serveur Prometheus**
Un serveur Prometheus collectant des métriques provenant :  
- Des conteneurs Docker  
- Du réseau (via Node Exporter ) 

Prometheus est configuré avec des alertes pour surveiller l'état de votre infrastructure. Vous serez averti en cas de :  
- Charge système (load) trop élevée  
- Espace disque insuffisant  
- Tentative de DDoS (nombre excessif de connexions TCP)  
- Utilisation excessive de la mémoire  

### 3. **Serveur Alertmanager**
Un serveur Alertmanager configuré pour gérer les alertes générées par Prometheus.  
En fonction de la gravité des alertes, vous recevrez des notifications par email. Cela permet de prioriser les actions nécessaires pour résoudre les problèmes détectés.  

---

## Récupérer le dépot  
### Prérequis  
Avant de cloner le dépôt, assurez-vous d'avoir **Git** installé sur votre machine.  
Vous pouvez installer Git en suivant les instructions pour votre système d'exploitation depuis le [site officiel de Git](https://git-scm.com/).
### Cloner le dépôt GitHub  
```bash
git clone https://github.com/IPouPou/SAE502.git
cd SAE502
```
## Installation

### Exécuter le script d'installation  

```bash
sudo su
chmod +x install_and_run.sh
./install_and_run.sh
```
---
## Utilisation  
Une fois l'installation terminée, accédez aux services via votre navigateur :  

- **Grafana** : [http://ip_grafana:3000](http://<votre-ip>:3000)  
- **Prometheus** : [http://ip_prometheus:9090](http://<votre-ip>:9090)  
- **Alertmanager** : [http://ip_alertmanager:9093](http://<votre-ip>:9093)  
Utilisez les identifiants par défaut configurés dans le projet ou référez-vous à la documentation pour les personnaliser.  
---


## Fichier a vérifier/modifier avant d'installer

### Configuration du réseau Docker

Dans le fichier `networkcreate`, vous pouvez modifier le **nom** et l'**IP** du réseau Docker sur lequel vos conteneurs seront déployés.  
Cela vous permet de personnaliser les paramètres du réseau Docker, notamment pour adapter l'IP du réseau à votre environnement.
```ini
 - name: Créer un réseau Docker
      community.docker.docker_network:
        name: projetsae
        driver: bridge
        ipam_config:
          - subnet: 192.168.10.0/24
        state: present
```
### Configuration des IP des services

Dans le fichier `inventaire.ini`, vous trouverez les adresses IP des services **Grafana**, **Prometheus** et **Alertmanager**.  
Si vous avez choisi des adresses IP différentes pour ces services lors de la création des conteneurs, vous devez modifier ce fichier pour qu'il corresponde aux adresses IP définies dans le fichier de déploiement des conteneurs.


Voici un exemple de la structure du fichier `inventaire.ini` :  

```ini
[grafana]
192.168.10.5

[prometheus]
192.168.10.4

[alertmanager]
192.168.10.6

[hote]
10.0.2.15
```
### Configuration des Targets de prometheus

Dans le fichier `roles/prometheus/files/prometheus.yml`, vous devez modifier les adresses IP pour qu'elles correspondent aux IP globales définies lors de la configuration des services. Ce fichier configure Prometheus pour collecter des métriques et gérer les alertes.  

Voici un extrait du fichier :  

```yaml
scrape_interval: 15s
evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'docker_services'
    static_configs:
      - targets:
          - "192.168.10.5:3000"  # Grafana
          - "192.168.10.6:9093"  # AlertManager

  - job_name: 'machines'
    static_configs:
      - targets:
          - "10.0.2.15:9100"

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - "192.168.10.6:9093"
```
**Modifications à effectuer**

**Grafana** : Remplacez l'IP 192.168.10.5:3000 par l'adresse IP globale que vous avez définie pour Grafana.

**AlertManager** : Remplacez l'IP 192.168.10.6:9093 par l'adresse IP globale que vous avez définie pour AlertManager.

**Machines** : Si vous collectez des métriques sur d'autres machines, ajoutez ou modifiez les cibles sous le job machines.

**Note** : Assurez-vous que les IP modifiées dans ce fichier sont cohérentes avec celles définies dans le fichier inventaire.ini et dans la configuration du réseau Docker.
---

### Configuration des Notifications par Email

Le fichier `roles/alertmanager/files/alertmanager.yml` contient les paramètres nécessaires pour configurer les notifications par email via Alertmanager. Vous devez modifier ce fichier pour y ajouter vos informations de connexion à un serveur SMTP.  

### Sections à Modifier
Voici un exemple de la structure à configurer :  
```yaml
receivers:
- name: 'email-alert'
  email_configs:
  - send_resolved: true
    to: ''            # Adresse email du destinataire
    from: ''          # Adresse email de l'expéditeur
    smarthost: 'smtp.gmail.com:587'  # Serveur SMTP avec port
    auth_username: '' # Nom d'utilisateur SMTP (ex : votre adresse Gmail)
    auth_password: '' # Mot de passe ou clé d'application SMTP
```
---

## Fonctionnement du script d'installation

Le script `install_and_run.sh` automatise l'installation et la configuration des outils nécessaires au projet. Voici les principales étapes :  

1. **Mise à jour des packages**  
   - Met à jour les packages existants sur le système.  

2. **Installation des dépendances**  
   - Installe Docker, Ansible, et Tree.  
   - Configure et vérifie leur installation.  

3. **Installation de Node Exporter**  
   - Télécharge, configure, et démarre le service Node Exporter pour la collecte de métriques.  

4. **Déploiement et configuration des services**  
   - Exécute une série de playbooks Ansible pour :  
     - Créer le réseau Docker.  
     - Déployer les conteneurs nécessaires (Grafana, Prometheus, AlertManager).  
     - Configurer Prometheus, Grafana, et AlertManager.  

### Résultat :  
Une fois le script exécuté, tous les outils et services nécessaires seront installés, configurés, et prêts à l'emploi.  


## Arborescence du Projet

Voici la structure des fichiers et répertoires du projet :  

```plaintext
 ── playbooks/                # Contient les playbooks Ansible pour l'installation et la configuration  
     ── alertmanager.yml            # Déploie et configure Alertmanager  
     ── configprometheus.yml        # Configure Prometheus  
     ── deploiementconteneur.yml    # Déploie les conteneurs Docker  
     ── dockernetworkcreate.yml     # Crée le réseau Docker  
     ── grafanaconfig.yml           # Configure Grafana  

 ── roles/                    # Contient les rôles Ansible pour chaque service  
     ── alertmanager/               # Rôle pour Alertmanager  
         ── files/  
             ── alertmanager.yml    # Fichier de configuration Alertmanager  
         ── tasks/  
             ── deployconfig.yml    # Tâches pour déployer Alertmanager  
             ── main.yml            # Point d'entrée des tâches Alertmanager  

     ── grafana/                   # Rôle pour Grafana  
         ── tasks/  
             ── createdashboard.yml         # Crée un dashboard dans Grafana  
             ── dashboard_disk_usage.yml    # Dashboard d'utilisation disque  
             ── dashboard_error_network.yml # Dashboard d'erreurs réseau  
             ── main.yml                    # Point d'entrée des tâches Grafana

     ── prometheus/                # Rôle pour Prometheus  
         ── files/  
             ── alert_rules.yml    # Règles d'alertes pour Prometheus  
             ── prometheus.yml     # Fichier de configuration Prometheus  
         ── tasks/  
             ── deployconfig.yml   # Tâches pour configurer Prometheus  
             ── main.yml           # Point d'entrée des tâches Prometheus  

 ── README.md                 # Documentation du projet  
 ── ansible.cfg               # Configuration globale d'Ansible  
 ── install_and_run.sh        # Script d'installation et d'exécution du projet  
 ── inventaire.ini            # Fichier d'inventaire contenant les IP des services
```
**Explication des répertoires** :

   **playbooks/** : Contient les différents playbooks Ansible utilisés pour déployer et configurer les services.
   
   **roles/** : Chaque sous-dossier représente un rôle Ansible (Alertmanager, Grafana, Prometheus) avec leurs fichiers et tâches respectifs.
   
   **files/**: Contient les fichiers de configuration (ex : prometheus.yml).
   
   **tasks/** : Décrit les tâches d'installation et de configuration exécutées par Ansible.
   
   **ansible.cfg** : Fichier de configuration pour Ansible.

   **install_and_run.sh** : Script Bash automatisant l'installation des dépendances et l'exécution des playbooks Ansible.
   
   **inventaire.ini**: Définit les adresses IP des services (à adapter selon votre configuration réseau).
   
