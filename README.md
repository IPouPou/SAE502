# SAE502 - Supervision Réseaux

## Description  
Ce projet a pour objectif de déployer une infrastructure de supervision réseau entièrement automatisée.  
Grâce à cet environnement, il est possible de :  
- Monitorer des services.  
- Collecter des métriques.  
- Gérer des alertes avec **Grafana**, **Prometheus** et **Alertmanager**.  

Le projet repose sur des technologies modernes telles que **Docker** et **Ansible**, garantissant une installation rapide et une configuration reproductible.  

---

## Technologies utilisées  
- **Ansible** : Automatisation des déploiements.  
- **Docker** : Gestion des conteneurs.  
- **Prometheus** : Collecte et surveillance des métriques.  
- **Grafana** : Visualisation des données.  
- **Alertmanager** : Gestion des alertes.  

---

## Installation  
### Prérequis  
Avant de cloner le dépôt, assurez-vous d'avoir **Git** installé sur votre machine.  
Vous pouvez installer Git en suivant les instructions pour votre système d'exploitation depuis le [site officiel de Git](https://git-scm.com/).
### Cloner le dépôt GitHub  
```bash
git clone https://github.com/IPouPou/SAE502.git
cd SAE502
```
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


### Exécuter le script d'installation  

```bash
./install_and_run.sh
```
---
## Utilisation  

Une fois l'installation terminée, accédez aux services via votre navigateur :  

- **Grafana** : [http://<votre-ip>:3000](http://<votre-ip>:3000)  
- **Prometheus** : [http://<votre-ip>:9090](http://<votre-ip>:9090)  
- **Alertmanager** : [http://<votre-ip>:9093](http://<votre-ip>:9093)  

Utilisez les identifiants par défaut configurés dans le projet ou référez-vous à la documentation pour les personnaliser.  

