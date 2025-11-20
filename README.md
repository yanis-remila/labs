#  Architecture Big Data pour Gestion de Freelances


---

## RÉCAPITULATIF DES 5 LIVRABLES

### ✅ 1. Diagramme de Composants

**Architecture logique en 5 couches** :

1. **Couche Ingestion**
    - File Loader JSON/CSV
    - Kafka Producer

2. **Couche Streaming**
    - **Topic: Profils** (2 partitions)
    - **Topic: Factures** (2 partitions)
    - Kafka Consumer

3. **Couche Traitement**
    - Spark Master (orchestration)
    - Worker 1 (ID 1-500)
    - Worker 2 (ID 501-1000)
    - **T1: Nettoyage** (doublons + validation)
    - **T2: Calculs Financiers** (CA + revenus)
    - **T3: Agrégations** (compétences + taux)

4. **Couche Stockage**
    - PostgreSQL (PGsql)
    - 3 tables : Freelances, Factures, Indicateurs

5. **Couche Visualisation**
    - Dashboard Financier
    - Dashboard Ressources
    - Suivi Temps Réel

```mermaid
graph TB
    subgraph "📦 COUCHE INGESTION"
        C1["📄 File Loader JSON/CSV<br/>──────────<br/>Composant: DataIngestion<br/>Rôle: Lecture fichiers sources<br/>Input: JSON (Profils), CSV (Factures)"]

        C2["🔌 Kafka Producer<br/>──────────<br/>Composant: MessageBroker<br/>Rôle: Publication vers topics<br/>Output: 2 topics distincts"]
    end

    subgraph "🚀 COUCHE STREAMING"
        C3A["📨 Topic: Profils<br/>──────────<br/>Composant: EventStreaming<br/>Rôle: Flux nouveaux freelances<br/>Partitions: 2"]

        C3B["📨 Topic: Factures<br/>──────────<br/>Composant: EventStreaming<br/>Rôle: Flux paiements<br/>Partitions: 2"]

        C4["👂 Kafka Consumer<br/>──────────<br/>Composant: StreamConsumer<br/>Rôle: Consommation messages<br/>Tech: Spark Streaming"]
    end

    subgraph "⚡ COUCHE TRAITEMENT"
        C5["🎯 Spark Master<br/>──────────<br/>Composant: Orchestrator<br/>Rôle: Coordination + Distribution<br/>API: Cluster Manager"]

        C6A["⚙️ Worker 1<br/>──────────<br/>Composant: DataProcessor<br/>Rôle: Traitement ID 1-500<br/>Partition: 0"]

        C6B["⚙️ Worker 2<br/>──────────<br/>Composant: DataProcessor<br/>Rôle: Traitement ID 501-1000<br/>Partition: 1"]

        subgraph Transform["💾 Transformations Spark SQL"]
            C7A["🧹 T1: Nettoyage<br/>──────────<br/>Suppression doublons<br/>Validation données"]

            C7B["💰 T2: Calculs Financiers<br/>──────────<br/>CA par freelance<br/>Revenus mensuels"]

            C7C["📊 T3: Agrégations<br/>──────────<br/>Top compétences<br/>Taux d'occupation"]
        end
    end

    subgraph "🗄️ COUCHE STOCKAGE"
        C8["📊 PostgreSQL (PGsql)<br/>──────────<br/>Composant: DataWarehouse<br/>Tables:<br/>• Freelances (profils + compétences)<br/>• Factures (historique paiements)<br/>• Indicateurs (KPIs + stats)"]
    end

    subgraph "📊 COUCHE VISUALISATION"
        C10A["💰 Dashboard Financier<br/>──────────<br/>Composant: PowerBI-Finance<br/>• Top 10 freelances par CA<br/>• Évolution revenus mensuels"]

        C10B["👥 Dashboard Ressources<br/>──────────<br/>Composant: PowerBI-RH<br/>• Compétences disponibles<br/>• Taux occupation: 78%"]

        C10C["⚡ Suivi Temps Réel<br/>──────────<br/>Composant: PowerBI-Live<br/>• Missions en cours<br/>• Nouvelles factures"]
    end

    subgraph "🔧 COUCHE SUPPORT"
        C12["📝 Logger<br/>──────────<br/>Composant: Monitoring<br/>Rôle: Logs & Alertes"]
    end

    C1 -->|"JSON + CSV"| C2
    C2 -->|"Produit profils"| C3A
    C2 -->|"Produit factures"| C3B
    C3A -->|"Consomme"| C4
    C3B -->|"Consomme"| C4
    C4 -->|"Envoie flux"| C5
    C5 -->|"Distribue 1-500"| C6A
    C5 -->|"Distribue 501-1000"| C6B
    C6A -->|"Process"| C7A
    C6B -->|"Process"| C7A
    C7A -->|"Données nettoyées"| C7B
    C7B -->|"Calculs terminés"| C7C
    C7C -->|"Écrit"| C8
    C8 -->|"Connexion"| C10A
    C8 -->|"Connexion"| C10B
    C8 -->|"Connexion"| C10C
    C5 -.->|"Interroge localisation"| C6A
    C5 -.->|"Interroge localisation"| C6B
    C7A -.->|"Log"| C12
    C7B -.->|"Log"| C12
    C7C -.->|"Log"| C12

    style C1 fill:#e3f2fd
    style C2 fill:#e3f2fd
    style C3A fill:#fff9c4
    style C3B fill:#fff9c4
    style C4 fill:#fff9c4
    style C5 fill:#ef5350,color:#fff
    style C6A fill:#ffcdd2
    style C6B fill:#ffcdd2
    style C7A fill:#fff176
    style C7B fill:#fff176
    style C7C fill:#fff176
    style C8 fill:#c8e6c9
    style C10A fill:#b39ddb
    style C10B fill:#b39ddb
    style C10C fill:#b39ddb
    style C12 fill:#e0e0e0
```



---

### ✅ 2. Diagramme de Déploiement

```mermaid
graph TB
    subgraph Cloud["☁️ INFRASTRUCTURE CLOUD (AWS / Azure)"]
        subgraph VM1["🖥️ VM-01 : Ingestion & Streaming<br/>──────────<br/>Ubuntu 22.04<br/>RAM: 16 GB | CPU: 8 cores<br/>Disque: 500 GB SSD<br/>IP: 10.0.1.10"]
            D1["🐳 Container: File Loader<br/>──────────<br/>Image: python:3.11<br/>RAM: 2 GB | CPU: 1 core<br/>──────────<br/>Rôle: Lecture JSON/CSV<br/>Volume: /data/input<br/>Port: 8000"]

            D2["🐳 Container: Kafka Producer<br/>──────────<br/>Image: python:3.11<br/>RAM: 2 GB | CPU: 1 core<br/>──────────<br/>Rôle: Publication vers topics<br/>Port: 9093"]

            D3["🐳 Container: Zookeeper<br/>──────────<br/>Image: zookeeper:3.8<br/>RAM: 2 GB | CPU: 1 core<br/>──────────<br/>Port: 2181<br/>Volume: /var/lib/zookeeper"]

            D4["🐳 Container: Kafka Broker 01<br/>──────────<br/>Image: confluentinc/kafka:7.5<br/>RAM: 4 GB | CPU: 2 cores<br/>──────────<br/>Topics:<br/>📨 Profils (2 partitions)<br/>📨 Factures (2 partitions)<br/>──────────<br/>Port: 9092<br/>Volume: /var/lib/kafka"]

            D5["🐳 Container: Kafka Broker 02<br/>──────────<br/>Image: confluentinc/kafka:7.5<br/>RAM: 4 GB | CPU: 2 cores<br/>──────────<br/>Rôle: Réplication topics<br/>Port: 9094"]
        end

        subgraph VM2["🖥️ VM-02 : Traitement Spark<br/>──────────<br/>Ubuntu 22.04<br/>RAM: 32 GB | CPU: 16 cores<br/>Disque: 1 TB SSD<br/>IP: 10.0.2.10"]
            D6["🐳 Container: Spark Master<br/>──────────<br/>Image: bitnami/spark:3.5<br/>RAM: 8 GB | CPU: 4 cores<br/>──────────<br/>Rôles:<br/>• Coordination workers<br/>• Consumer Kafka<br/>• Distribution tâches<br/>──────────<br/>Port: 7077, 8080<br/>Env: SPARK_MODE=master"]

            D7["🐳 Container: Spark Worker 01<br/>──────────<br/>Image: bitnami/spark:3.5<br/>RAM: 10 GB | CPU: 5 cores<br/>──────────<br/>Traitement:<br/>• Freelances ID 1-500<br/>• Partition 0<br/>──────────<br/>Port: 8081<br/>Env: SPARK_WORKER_MEMORY=10G"]

            D8["🐳 Container: Spark Worker 02<br/>──────────<br/>Image: bitnami/spark:3.5<br/>RAM: 10 GB | CPU: 5 cores<br/>──────────<br/>Traitement:<br/>• Freelances ID 501-1000<br/>• Partition 1<br/>──────────<br/>Port: 8082<br/>Env: SPARK_WORKER_MEMORY=10G"]
        end

        subgraph VM3["🖥️ VM-03 : Stockage PostgreSQL<br/>──────────<br/>Ubuntu 22.04<br/>RAM: 16 GB | CPU: 8 cores<br/>Disque: 2 TB SSD<br/>IP: 10.0.3.10"]
            D9["🐳 Container: PostgreSQL 15<br/>──────────<br/>Image: postgres:15<br/>RAM: 12 GB | CPU: 6 cores<br/>──────────<br/>Database: freelances_db<br/>──────────<br/>Tables:<br/>• Freelances (profils)<br/>• Factures (paiements)<br/>• Indicateurs (KPIs)<br/>──────────<br/>Port: 5432<br/>Volume: /var/lib/postgresql<br/>Backup automatique"]

            D10["🐳 Container: PgAdmin<br/>──────────<br/>Image: dpage/pgadmin4<br/>RAM: 1 GB | CPU: 1 core<br/>──────────<br/>Rôle: Interface admin BDD<br/>Port: 5050"]
        end

        subgraph VM4["🖥️ VM-04 : Monitoring<br/>──────────<br/>Ubuntu 22.04<br/>RAM: 8 GB | CPU: 4 cores<br/>Disque: 500 GB SSD<br/>IP: 10.0.4.10"]
            D11["🐳 Container: Grafana<br/>──────────<br/>Image: grafana/grafana<br/>RAM: 2 GB | CPU: 1 core<br/>──────────<br/>Rôle: Dashboards monitoring<br/>Port: 3000"]

            D12["🐳 Container: Prometheus<br/>──────────<br/>Image: prom/prometheus<br/>RAM: 3 GB | CPU: 1 core<br/>──────────<br/>Rôle: Métriques système<br/>Port: 9090"]

            D13["🐳 Container: Kafka UI<br/>──────────<br/>Image: provectuslabs/kafka-ui<br/>RAM: 1 GB | CPU: 1 core<br/>──────────<br/>Rôle: Monitoring Kafka<br/>Port: 8080"]
        end

        subgraph Storage["☁️ OBJECT STORAGE"]
            N8["📦 S3 / Azure Blob Storage<br/>──────────<br/>Capacité: 5 TB<br/>──────────<br/>Contenu:<br/>• Fichiers sources (JSON/CSV)<br/>• Backups Docker volumes<br/>• Images Docker registry<br/>• Logs applicatifs<br/>• Snapshots VMs"]
        end
    end

    subgraph OnPrem["🏢 POSTE CLIENT"]
        N9["💻 Poste Utilisateur<br/>──────────<br/>OS: Windows 11<br/>──────────<br/>Applications:<br/>• Power BI Desktop<br/>• Docker Desktop (dev)<br/>• Git + VS Code<br/>• Web Browser<br/>──────────<br/>Connexion: HTTPS + VPN"]
    end

    subgraph External["🌐 SOURCES EXTERNES"]
        N10["📁 Dépôt Fichiers<br/>──────────<br/>📄 Profils freelances (JSON)<br/>📊 Factures (CSV)<br/>──────────<br/>Protocole: SFTP / API<br/>Fréquence: Quotidien<br/>Path: /data/input"]
    end

    subgraph Registry["🐳 DOCKER REGISTRY"]
        DR["Docker Hub / Private Registry<br/>──────────<br/>Images personnalisées:<br/>• freelanceflow/loader:v1.0<br/>• freelanceflow/producer:v1.0<br/>• freelanceflow/spark-jobs:v1.0<br/>──────────<br/>Images officielles:<br/>• postgres:15<br/>• kafka:7.5<br/>• spark:3.5<br/>• grafana, prometheus"]
    end

    N10 -->|"SFTP Upload<br/>Port 22"| D1
    D1 -->|"Fichiers traités"| D2
    D2 -->|"Kafka Protocol<br/>Port 9092"| D4
    D3 <-->|"Coordination<br/>Zookeeper"| D4
    D4 <-->|"Réplication<br/>Topics"| D5
    D4 -->|"Consumer<br/>Topics P+F"| D6

    D6 -->|"Spark RPC<br/>Distribue 1-500"| D7
    D6 -->|"Spark RPC<br/>Distribue 501-1000"| D8
    D6 -.->|"Query: Données<br/>freelance #245?"| D7
    D7 -.->|"Response: Oui"| D6

    D7 -->|"JDBC Write<br/>Port 5432"| D9
    D8 -->|"JDBC Write<br/>Port 5432"| D9

    D9 -->|"HTTPS<br/>DirectQuery"| N9
    D10 -.->|"Admin BDD"| D9

    D11 -->|"Scrape metrics"| D6
    D11 -->|"Scrape metrics"| D9
    D12 -->|"Push metrics"| D11
    D13 -->|"Monitor"| D4

    N9 -.->|"Monitoring UI"| D11
    N9 -.->|"Spark UI"| D6
    N9 -.->|"Kafka UI"| D13

    D1 -.->|"Backup"| N8
    D4 -.->|"Backup"| N8
    D9 -.->|"Backup"| N8

    DR -.->|"Pull images"| VM1
    DR -.->|"Pull images"| VM2
    DR -.->|"Pull images"| VM3
    DR -.->|"Pull images"| VM4

    style Cloud fill:#e3f2fd,stroke:#1976d2,stroke-width:3px
    style VM1 fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style VM2 fill:#ffebee,stroke:#c62828,stroke-width:4px
    style VM3 fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
    style VM4 fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px
    style Storage fill:#e0f2f1,stroke:#00695c,stroke-width:2px
    style OnPrem fill:#b39ddb,stroke:#5e35b1,stroke-width:2px
    style External fill:#e0e0e0,stroke:#616161,stroke-width:2px
    style Registry fill:#bbdefb,stroke:#1976d2,stroke-width:2px

    style D1 fill:#42a5f5,color:#fff
    style D2 fill:#42a5f5,color:#fff
    style D3 fill:#81c784
    style D4 fill:#ffd54f,stroke:#f57f17,stroke-width:2px
    style D5 fill:#ffd54f
    style D6 fill:#ef5350,color:#fff,stroke:#c62828,stroke-width:3px
    style D7 fill:#ff8a80
    style D8 fill:#ff8a80
    style D9 fill:#66bb6a,stroke:#2e7d32,stroke-width:3px
    style D10 fill:#aed581
    style D11 fill:#ba68c8,color:#fff
    style D12 fill:#ce93d8
    style D13 fill:#f48fb1
```

---

### ✅ 3. Indicateurs BI et Machine Learning

#### 📊 3 Dashboards Power BI

**1. Dashboard Financier**
- Top 10 freelances par CA
- Évolution revenus mensuels (line chart)
- KPIs : CA total, TJM moyen
- Requêtes SQL : CA par freelance, Revenus mensuels

**2. Dashboard Ressources**
- Compétences disponibles (pie chart)
- Taux d'occupation : 78% (gauge)
- Freelances disponibles vs en mission
- Top compétences demandées

**3. Suivi Temps Réel**
- Missions en cours (compteur live)
- Nouvelles factures du jour
- Refresh : 30 secondes
- Alertes factures en retard

#### 🤖 3 Modèles ML

**1. Prédiction CA Mensuel**
- Type : Time Series (ARIMA/Prophet)
- MAPE cible : < 15%
- Prédiction : 3 mois

**2. Matching Freelance-Compétence**
- Type : Content-Based Filtering
- Precision@5 : > 70%

**3. Détection Anomalies Factures**
- Type : Isolation Forest
- Taux faux positifs : < 5%

#### 💾 3 Transformations Spark SQL

**T1: Nettoyage**
```sql
-- Suppression doublons
-- Validation formats (email, dates, montants)
-- Normalisation données
```

**T2: Calculs Financiers**
```sql
-- CA par freelance
-- Revenus mensuels
-- TJM moyen par compétence
```

**T3: Agrégations**
```sql
-- Top compétences demandées
-- Taux d'occupation global
-- KPIs consolidés
```

---

### ✅ 4. Backlog Teams


#### 📋 FreelanceFlow - Backlog Projet (Version Finale)

## 🎯 EPIC 1 : Infrastructure Kafka & Ingestion

### 📦 User Story 1.1 : Configuration Kafka avec 2 Topics
**En tant que** Data Engineer  
**Je veux** configurer un cluster Kafka avec 2 topics distincts  
**Afin de** séparer les flux Profils et Factures

**Critères d'acceptation** :
- [ ] Cluster Kafka déployé avec 2 brokers
- [ ] **Topic "Profils"** créé avec 2 partitions
- [ ] **Topic "Factures"** créé avec 2 partitions
- [ ] Réplication factor = 2 pour chaque topic
- [ ] Tests production/consommation sur les 2 topics
- [ ] Monitoring topics dans Kafka UI

**Priorité** : 🔴 Haute  
**Points de complexité** : 8  
**Sprint** : Sprint 1  
**Assigné à** : @DataEngineering  
**Tags** : `infrastructure` `kafka` `topics` `streaming`

---

### 📦 User Story 1.2 : Script ingestion JSON/CSV vers Kafka
**En tant que** Data Engineer  
**Je veux** créer un producer Kafka qui lit JSON et CSV  
**Afin de** publier vers les bons topics

**Critères d'acceptation** :
- [ ] Lecture fichiers JSON (profils freelances)
- [ ] Lecture fichiers CSV (factures)
- [ ] Publication JSON → Topic "Profils"
- [ ] Publication CSV → Topic "Factures"
- [ ] Chargement quotidien automatisé
- [ ] Logs des publications
- [ ] Gestion erreurs et retry

**Priorité** : 🔴 Haute  
**Points de complexité** : 5  
**Sprint** : Sprint 1  
**Assigné à** : @Developer1  
**Tags** : `python` `kafka-producer` `json` `csv`

---

## ⚡ EPIC 2 : Cluster Spark & Distribution

### 📦 User Story 2.1 : Déploiement Cluster Spark
**En tant que** Data Engineer  
**Je veux** déployer 1 master + 2 workers  
**Afin de** traiter les données en parallèle

**Critères d'acceptation** :
- [ ] Spark Master configuré (32 GB RAM, 16 cores)
- [ ] Worker 1 : traitement ID 1-500 (16 GB RAM)
- [ ] Worker 2 : traitement ID 501-1000 (16 GB RAM)
- [ ] Interface Web Spark accessible
- [ ] Tests de distribution fonctionnels
- [ ] Configuration mémoire optimisée

**Priorité** : 🔴 Haute  
**Points de complexité** : 8  
**Sprint** : Sprint 1  
**Assigné à** : @DevOps  
**Tags** : `spark` `infrastructure` `distributed`

---

### 📦 User Story 2.2 : Consumer Kafka vers Spark
**En tant que** Data Engineer  
**Je veux** consommer les 2 topics Kafka avec Spark Streaming  
**Afin de** alimenter le traitement distribué

**Critères d'acceptation** :
- [ ] Consumer Spark Streaming fonctionnel
- [ ] Lecture Topic "Profils"
- [ ] Lecture Topic "Factures"
- [ ] Transformation en DataFrame Spark
- [ ] Distribution vers workers appropriés
- [ ] Gestion offsets Kafka
- [ ] Tests d'intégration

**Priorité** : 🔴 Haute  
**Points de complexité** : 13  
**Sprint** : Sprint 2  
**Assigné à** : @Developer2  
**Tags** : `spark-streaming` `kafka-consumer` `dataframe`

---

## 💾 EPIC 3 : Transformations Spark SQL

### 📦 User Story 3.1 : T1 - Nettoyage des données
**En tant que** Data Analyst  
**Je veux** implémenter le module de nettoyage  
**Afin de** supprimer doublons et valider les données

**Critères d'acceptation** :
- [ ] Suppression doublons (profils + factures)
- [ ] Validation emails (format correct)
- [ ] Validation montants (> 0)
- [ ] Validation dates (format ISO)
- [ ] Gestion valeurs nulles
- [ ] Logs des rejets
- [ ] Métriques qualité données

**Priorité** : 🔴 Haute  
**Points de complexité** : 5  
**Sprint** : Sprint 2  
**Assigné à** : @DataAnalyst  
**Tags** : `spark-sql` `data-quality` `cleaning`

---

### 📦 User Story 3.2 : T2 - Calculs Financiers
**En tant que** Data Analyst  
**Je veux** calculer les métriques financières  
**Afin de** obtenir CA par freelance et revenus mensuels

**Critères d'acceptation** :
- [ ] Calcul CA par freelance
- [ ] Calcul revenus mensuels
- [ ] TJM moyen par compétence
- [ ] Agrégation par période
- [ ] Requêtes SQL optimisées (< 5s)
- [ ] Tests unitaires calculs

**Priorité** : 🔴 Haute  
**Points de complexité** : 8  
**Sprint** : Sprint 2  
**Assigné à** : @DataAnalyst  
**Tags** : `spark-sql` `financial` `kpi`

---

### 📦 User Story 3.3 : T3 - Agrégations
**En tant que** Data Analyst  
**Je veux** calculer les agrégations métier  
**Afin d'** obtenir top compétences et taux d'occupation

**Critères d'acceptation** :
- [ ] Top compétences demandées
- [ ] Taux d'occupation : 78% (target)
- [ ] Statistiques par catégorie
- [ ] Freelances disponibles par compétence
- [ ] Requêtes avec window functions
- [ ] Performance < 3s

**Priorité** : 🟡 Moyenne  
**Points de complexité** : 8  
**Sprint** : Sprint 3  
**Assigné à** : @DataAnalyst  
**Tags** : `spark-sql` `aggregation` `analytics`

---

## 🗄️ EPIC 4 : Data Warehouse PostgreSQL

### 📦 User Story 4.1 : Création schéma PostgreSQL (PGsql)
**En tant que** Data Architect  
**Je veux** créer le schéma de base PostgreSQL  
**Afin de** stocker les 3 tables principales

**Critères d'acceptation** :
- [ ] Base `freelances_db` créée
- [ ] Table `Freelances` (profils + compétences)
- [ ] Table `Factures` (historique paiements)
- [ ] Table `Indicateurs` (KPIs + statistiques)
- [ ] Index optimisés
- [ ] Contraintes d'intégrité
- [ ] Documentation schéma

**Priorité** : 🔴 Haute  
**Points de complexité** : 5  
**Sprint** : Sprint 2  
**Assigné à** : @DataArchitect  
**Tags** : `postgresql` `pgsql` `schema` `database`

---

### 📦 User Story 4.2 : Pipeline ETL Spark → PostgreSQL
**En tant que** Data Engineer  
**Je veux** écrire les données transformées dans PostgreSQL  
**Afin de** persister les résultats des transformations T1, T2, T3

**Critères d'acceptation** :
- [ ] Connexion JDBC Spark → PostgreSQL
- [ ] Écriture table Freelances
- [ ] Écriture table Factures
- [ ] Écriture table Indicateurs
- [ ] Mode upsert (insert/update)
- [ ] Transactions ACID
- [ ] Tests de charge

**Priorité** : 🔴 Haute  
**Points de complexité** : 8  
**Sprint** : Sprint 3  
**Assigné à** : @Developer1  
**Tags** : `etl` `jdbc` `postgresql` `pgsql`

---

## 📊 EPIC 5 : Dashboards Power BI

### 📦 User Story 5.1 : Dashboard Financier
**En tant que** Directeur Financier  
**Je veux** visualiser le Top 10 et l'évolution des revenus  
**Afin de** suivre la performance financière

**Critères d'acceptation** :
- [ ] **Top 10 freelances par CA** : graphique bar chart
- [ ] **Évolution revenus mensuels** : line chart 12 mois
- [ ] Filtres : période, compétence
- [ ] KPI : CA total, TJM moyen
- [ ] Refresh quotidien automatique
- [ ] Export Excel

**Priorité** : 🔴 Haute  
**Points de complexité** : 5  
**Sprint** : Sprint 3  
**Assigné à** : @BIAnalyst  
**Tags** : `powerbi` `dashboard` `finance`

---

### 📦 User Story 5.2 : Dashboard Ressources
**En tant que** Responsable RH  
**Je veux** voir les compétences disponibles et le taux d'occupation  
**Afin d'** optimiser l'allocation des freelances

**Critères d'acceptation** :
- [ ] **Compétences disponibles** : pie chart par techno
- [ ] **Taux d'occupation : 78%** : gauge visual
- [ ] Répartition freelances disponibles/en mission
- [ ] Filtres par compétence
- [ ] Alerte si taux < 60% ou > 90%
- [ ] Détail freelances disponibles (table)

**Priorité** : 🔴 Haute  
**Points de complexité** : 5  
**Sprint** : Sprint 4  
**Assigné à** : @BIAnalyst  
**Tags** : `powerbi` `dashboard` `hr` `resources`

---

### 📦 User Story 5.3 : Suivi Temps Réel
**En tant que** Manager Opérationnel  
**Je veux** voir les missions en cours et nouvelles factures  
**Afin de** piloter l'activité en temps réel

**Critères d'acceptation** :
- [ ] **Missions en cours** : compteur + liste
- [ ] **Nouvelles factures** : compteur du jour
- [ ] Refresh toutes les 30 secondes
- [ ] Dernières 10 factures créées (table)
- [ ] Alertes : factures en retard
- [ ] Indicateur statut système (vert/orange/rouge)

**Priorité** : 🟡 Moyenne  
**Points de complexité** : 8  
**Sprint** : Sprint 5  
**Assigné à** : @BIAnalyst  
**Tags** : `powerbi` `realtime` `streaming` `monitoring`

---

## 🤖 EPIC 6 : Machine Learning

### 📦 User Story 6.1 : Modèle prédiction CA
**En tant que** Data Scientist  
**Je veux** créer un modèle Time Series pour prédire le CA  
**Afin d'** anticiper les revenus des 3 prochains mois

**Critères d'acceptation** :
- [ ] Collecte historique CA mensuel (12+ mois)
- [ ] Feature engineering (saisonnalité, tendances)
- [ ] Entraînement modèle (ARIMA/Prophet)
- [ ] MAPE < 15%
- [ ] Prédictions à 3 mois
- [ ] Intégration dans Dashboard Financier
- [ ] Documentation modèle

**Priorité** : 🟡 Moyenne  
**Points de complexité** : 13  
**Sprint** : Sprint 6  
**Assigné à** : @DataScientist  
**Tags** : `ml` `timeseries` `forecasting` `arima`

---

### 📦 User Story 6.2 : Détection anomalies factures
**En tant que** Contrôleur Financier  
**Je veux** détecter automatiquement les factures suspectes  
**Afin de** prévenir les fraudes

**Critères d'acceptation** :
- [ ] Algorithme Isolation Forest
- [ ] Features : montant, fréquence, TJM
- [ ] Taux faux positifs < 5%
- [ ] Intégration Suivi Temps Réel
- [ ] Alertes automatiques
- [ ] Dashboard anomalies détectées

**Priorité** : 🟢 Basse  
**Points de complexité** : 13  
**Sprint** : Sprint 7  
**Assigné à** : @DataScientist  
**Tags** : `ml` `anomaly-detection` `fraud`

---

## 🔧 EPIC 7 : DevOps & Monitoring

### 📦 User Story 7.1 : CI/CD Pipeline
**En tant que** DevOps Engineer  
**Je veux** automatiser le déploiement  
**Afin d'** accélérer les mises en production

**Critères d'acceptation** :
- [ ] Pipeline GitHub Actions / GitLab CI
- [ ] Tests automatisés (unit + integration)
- [ ] Build Docker images (Kafka, Spark)
- [ ] Déploiement auto sur dev
- [ ] Validation manuelle pour prod
- [ ] Rollback automatique si erreur

**Priorité** : 🟡 Moyenne  
**Points de complexité** : 8  
**Sprint** : Sprint 3  
**Assigné à** : @DevOps  
**Tags** : `cicd` `automation` `deployment` `docker`

---

### 📦 User Story 7.2 : Monitoring système
**En tant que** Ops Engineer  
**Je veux** monitorer la santé de l'infrastructure  
**Afin de** détecter les problèmes rapidement

**Critères d'acceptation** :
- [ ] Monitoring Kafka (lag, throughput)
- [ ] Monitoring Spark (jobs, stages)
- [ ] Monitoring PostgreSQL (connexions, queries)
- [ ] Métriques système (CPU, RAM, Disk)
- [ ] Alertes emails/Slack
- [ ] Dashboard Grafana
- [ ] SLA : uptime > 99%

**Priorité** : 🟡 Moyenne  
**Points de complexité** : 8  
**Sprint** : Sprint 4  
**Assigné à** : @DevOps  
**Tags** : `monitoring` `grafana` `alerting` `sla`

---

## 🔍 ARCHITECTURE DÉTAILLÉE

### 📥 Sources de Données
- **Fichiers JSON** : Profils freelances
  ```json
  {"nom": "Dupont", "compétences": ["Python"], "tarif_jour": "450€"}
  ```
- **Fichiers CSV** : Factures
  ```
  freelance, montant, date
  Dupont, 4500€, Oct-2025
  ```

### 🚀 Kafka - 2 Topics
1. **Topic: Profils**
    - Reçoit les nouveaux freelances
    - 2 partitions
    - Réplication factor = 2

2. **Topic: Factures**
    - Reçoit les paiements
    - 2 partitions
    - Réplication factor = 2

### ⚡ Spark - Architecture Distribuée

**Nœud Maître** :
- Coordonne le travail
- Distribue les tâches
- Interroge les workers : *"Tu as les données du freelance #245 ?"*

**Worker 1** :
- Traite freelances ID 1-500
- Partition 0

**Worker 2** :
- Traite freelances ID 501-1000
- Partition 1

### 💾 3 Transformations Spark SQL

**T1 : Nettoyage**
- Supprime doublons
- Valide les données

**T2 : Calculs Financiers**
- CA par freelance
- Revenus mensuels

**T3 : Agrégations**
- Top compétences demandées
- Taux d'occupation

### 🗄️ PostgreSQL (PGsql)

**Base : freelances_db**

**3 Tables** :
1. **Freelances** : Profils et compétences
2. **Factures** : Historique paiements
3. **Indicateurs** : KPIs et statistiques

### 📊 Power BI - 3 Dashboards

**1. Dashboard Financier**
- Top 10 freelances par CA
- Évolution revenus mensuels

**2. Dashboard Ressources**
- Compétences disponibles
- Taux d'occupation : 78%

**3. Suivi Temps Réel**
- Missions en cours
- Nouvelles factures

---
