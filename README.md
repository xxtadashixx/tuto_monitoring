corrige ce readme pour une vision meilleur 
# 🚀 Projet Tutoriel de Monitoring avec Prometheus, Grafana, Node Exporter et Blackbox Exporter

> Un template simple et complet pour surveiller un serveur EC2 AWS et des sites web avec Prometheus & Grafana.

---

## 📋 Objectifs

- 📡 Surveiller un serveur **EC2 Linux** via **Node Exporter**
- 🌐 Surveiller des **sites web** avec **Blackbox Exporter**
- 📊 Visualiser le tout dans **Grafana**
- 🐳 Utiliser **Docker Compose** pour un déploiement rapide

---

## 🧰 Prérequis

- Docker & Docker Compose installés
- Un serveur EC2 Linux (Ubuntu recommandé)
- Git

---

## 📂 Arborescence du projet
  monitoring-project/
  ├── docker-compose.yml
  ├── prometheus/
  │ ├── prometheus.yml
  │ └── blackbox.yml
  └── grafana/

## ⚙️ Étape 2 : Configuration de Prometheus
## 📁 prometheus/prometheus.yml

    global:
      scrape_interval: 15s
    
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
  
    - job_name: 'node-exporter'
      static_configs:
        - targets: ['<IP_EC2>:9100']  # Remplacez par l'IP publique de votre EC2
  
    - job_name: 'blackbox'
      metrics_path: /probe
      params:
        module: [http_2xx]
      static_configs:
        - targets:
            - https://www.google.com
            - http://mondomaine.com
      relabel_configs:
        - source_labels: [__address__]
          target_label: __param_target
        - source_labels: [__param_target]
          target_label: instance
        - target_label: __address__
          replacement: blackbox:9115

## 📁 prometheus/blackbox.yml

    modules:
      http_2xx:
        prober: http
        timeout: 5s
        http:
          method: GET

## ⚙️ Étape 3 : Lancer les services avec Docker Compose
### docker-compose up -d

Prometheus : http://localhost:9090

Grafana : http://localhost:3000 (login: admin / admin)

Node Exporter : http://localhost:9100/metrics

Blackbox Exporter : http://localhost:9115/probe

## ⚙️ Étape 4 : Installer Node Exporter sur votre EC2
Connectez-vous à votre instance EC2 :

### ssh ubuntu@<IP_EC2>
Puis installez Node Exporter :

### wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz

### tar xvf node_exporter-*.tar.gz

### cd node_exporter-*

### ./node_exporter &


### 🎯 Astuce : pour rendre ce service permanent, créez un service systemd (non obligatoire pour le test rapide)
## 📈 Étape 5 : Configuration de Grafana

1-Accédez à http://localhost:3000
2-Identifiants par défaut : admin / admin
3-Ajoutez une source de données Prometheus :
4-URL : http://prometheus:9090
5-Importez des dashboards :
6-Node Exporter Full : ID 1860
7- Blackbox Exporter : ID 7587


## ✅ Vérification

Si tout est OK, vous verrez :
Des métriques système de votre EC2 dans Grafana
des statuts UP/DOWN des sites web surveillés
Vous pouvez tester une panne en arrêtant Node Exporter sur EC2 : 


### pkill node_exporter


## 📦 Ports utilisés
| Service             | Port |
| ------------------- | ---- |
| Prometheus          | 9090 |
| Node Exporter (EC2) | 9100 |
| Blackbox Exporter   | 9115 |
| Grafana             | 3000 |
