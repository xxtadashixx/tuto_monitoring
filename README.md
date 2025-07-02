# 🚀 Tutoriel de Monitoring avec Prometheus, Grafana, Node Exporter et Blackbox Exporter

> Un template simple et complet pour surveiller un serveur EC2 AWS et des sites web avec Prometheus & Grafana.

---

## 📋 Objectifs

- 📡 Surveiller un serveur **EC2 Linux** via **Node Exporter**
- 🌐 Surveiller des **sites web** via **Blackbox Exporter**
- 📊 Visualiser les métriques dans **Grafana**
- 🐳 Utiliser **Docker Compose** pour tout déployer rapidement

---

## 🧰 Prérequis

- Docker et Docker Compose installés
- Un serveur EC2 Linux (Ubuntu recommandé)
- Git installé localement

---

## 📂 Arborescence du projet

\`\`\`
monitoring-project/
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── blackbox.yml
└── grafana/
\`\`\`

---

## ⚙️ Étape 1 : Configuration de Prometheus

### 📁 prometheus/prometheus.yml

\`\`\`yaml
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
\`\`\`

---

### 📁 prometheus/blackbox.yml

\`\`\`yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      method: GET
\`\`\`

---

## ⚙️ Étape 2 : Lancer les services avec Docker Compose

\`\`\`bash
docker-compose up -d
\`\`\`

📌 Accès aux services :

- Prometheus → http://localhost:9090
- Grafana → http://localhost:3000 (login: `admin` / `admin`)
- Node Exporter → http://localhost:9100/metrics
- Blackbox Exporter → http://localhost:9115/probe

---

## ⚙️ Étape 3 : Installer Node Exporter sur l’EC2

Connectez-vous à votre serveur EC2 :

\`\`\`bash
ssh ubuntu@<IP_EC2>
\`\`\`

Puis installez et lancez Node Exporter :

\`\`\`bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.0/node_exporter-1.8.0.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*
./node_exporter &
\`\`\`

💡 *Astuce : pour un lancement automatique, vous pouvez créer un service systemd.*

---

## 📈 Étape 4 : Configuration de Grafana

1. Accédez à Grafana : http://localhost:3000  
2. Identifiants par défaut : `admin / admin`  
3. Ajoutez une nouvelle source de données :
   - Type : **Prometheus**
   - URL : `http://prometheus:9090`  
4. Importez des dashboards :
   - **Node Exporter Full** : ID `1860`
   - **Blackbox Exporter** : ID `7587`

---

## ✅ Étape 5 : Vérification

✔️ Si tout fonctionne, vous devriez voir :
- Les **métriques système** (CPU, RAM, disque) de votre EC2
- Le **statut UP/DOWN** des sites web surveillés

🧪 Pour tester une panne :

\`\`\`bash
pkill node_exporter
\`\`\`

→ Observez l’impact dans Prometheus ou Grafana.

---

## 📦 Ports utilisés

| Service             | Port |
|---------------------|------|
| Prometheus          | 9090 |
| Node Exporter (EC2) | 9100 |
| Blackbox Exporter   | 9115 |
| Grafana             | 3000 |

---

## 🧠 Bonus

- Ajouter des alertes (Prometheus alertmanager ou Grafana alerting)
- Intégration Slack/Email
- Déploiement multi-serveurs
- Supervision de base de données, Nginx, Docker, etc.

---

## 📝 Licence

Projet open-source sous licence **MIT** – utilisez-le librement à des fins éducatives ou professionnelles.
