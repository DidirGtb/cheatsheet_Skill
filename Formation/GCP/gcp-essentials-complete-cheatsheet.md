# Google Cloud Platform - Essentials Cheatsheet

> **Parcours** : Google Cloud Skills Boost - Essentials  
> **Auteur** : Didir  
> **Dernière mise à jour** : Janvier 2026

---

## 📋 Table des matières

1. [Architecture GCP](#1-architecture-gcp)
2. [Projets GCP](#2-projets-gcp)
3. [IAM (Identity & Access Management)](#3-iam-identity--access-management)
4. [APIs & Services](#4-apis--services)
5. [Compute Engine](#5-compute-engine)
6. [Réseau & Firewall](#6-réseau--firewall)
7. [Cloud Shell & gcloud CLI](#7-cloud-shell--gcloud-cli)
8. [Gestion des coûts](#8-gestion-des-coûts)
9. [Bonnes pratiques](#9-bonnes-pratiques)
10. [Commandes essentielles](#10-commandes-essentielles)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Architecture GCP

### Hiérarchie des ressources

```
Organization (niveau entreprise)
    └── Folder (groupement logique)
        └── Project (conteneur principal)
            └── Resources (VM, Storage, DB...)
```

### Concepts clés

| Concept | Description | Exemple |
|---------|-------------|---------|
| **Organization** | Nœud racine pour une entreprise | `bnpparibas.com` |
| **Folder** | Regroupement de projets | `prod/`, `dev/`, `team-data/` |
| **Project** | Conteneur de ressources + facturation | `myapp-prod-123456` |
| **Resource** | Services GCP individuels | VM, Bucket, Database |

### Régions et Zones

#### Qu'est-ce qu'une région ?
- **Region** = Zone géographique (ex: `europe-west9` = Paris)
- **Zone** = Datacenter dans une région (ex: `europe-west9-a`)

#### Régions Europe principales

```
europe-west1    → Belgique (St. Ghislain)
europe-west4    → Pays-Bas (Eemshaven)  
europe-west9    → France (Paris) 🇫🇷
europe-north1   → Finlande (Hamina)
europe-west3    → Allemagne (Francfort)
```

#### Critères de choix

| Critère | Impact |
|---------|--------|
| **Latence** | Plus proche des users = plus rapide |
| **Coûts** | Varient selon la région (+/- 20%) |
| **Compliance** | RGPD → données en Europe obligatoire |
| **Disponibilité** | Multi-zones = Haute disponibilité |

---

## 2. Projets GCP

### 🎯 Pourquoi utiliser des projets ?

Un **Project** est le conteneur de base dans GCP. Toutes les ressources appartiennent à un projet.

#### Avantages de la segmentation

```
✅ Séparation des environnements (dev/staging/prod)
✅ Isolation de la facturation (coûts par projet)
✅ Gestion IAM simplifiée (permissions par projet)
✅ Sécurité renforcée (isolation des ressources)
✅ Maintenance facilitée (vue claire des ressources)
```

#### Exemple d'architecture

```
myapp-dev          → Développement
myapp-staging      → Tests/pré-production
myapp-prod         → Production
myapp-analytics    → Data & Analytics
myapp-ml           → Machine Learning
```

### 📸 Screenshot 1 : Liste des projets
**Emplacement** : `./screenshots/01-projects-list.png`  
**Description** : Vue du sélecteur de projet avec la liste de tous vos projets

![Liste des projets](./screenshots/01-projects-list.png)

### Créer un projet

#### Via Console Web

1. Cliquer sur le **sélecteur de projet** (en haut à gauche)
2. Bouton **New Project**
3. Remplir les informations :
   - **Project name** : `MyApp Production` (nom lisible)
   - **Project ID** : `myapp-prod-123456` (unique globalement)
   - **Organization** : (si applicable)
4. Cliquer sur **Create**

#### Via gcloud CLI

```bash
# Créer un projet
gcloud projects create PROJECT_ID --name="PROJECT_NAME"

# Exemple concret
gcloud projects create myapp-prod-456789 \
  --name="MyApp Production"

# Définir comme projet actif
gcloud config set project myapp-prod-456789

# Vérifier le projet actif
gcloud config get-value project
```

### Identifier un projet

Chaque projet a **3 identifiants** :

| Type | Caractéristique | Exemple | Usage |
|------|-----------------|---------|-------|
| **Project ID** | Unique global, immuable | `myapp-prod-456789` | APIs, CLI |
| **Project Name** | Lisible, modifiable | `MyApp Production` | Console |
| **Project Number** | Numérique, généré | `123456789012` | Interne GCP |

```bash
# Voir tous les identifiants
gcloud projects describe myapp-prod-456789

# Sortie :
# createTime: '2026-01-10T10:30:00.000Z'
# lifecycleState: ACTIVE
# name: MyApp Production
# projectId: myapp-prod-456789
# projectNumber: '123456789012'
```

### Naming conventions

#### Format recommandé

```
{app}-{environment}-{team/region}

✅ Bons exemples :
crm-prod-sales
analytics-dev-data
website-staging-frontend
ml-model-prod-datascience

❌ Mauvais exemples :
test
project1
mon-truc
asdfghjkl
```

### Labels (tags)

Les **labels** sont des paires clé-valeur pour organiser les ressources.

#### Labels recommandés

```yaml
env: prod
team: data-engineering
owner: idir
cost-center: CC-1234
managed-by: terraform
application: crm
```

#### Appliquer des labels

```bash
# Ajouter des labels
gcloud projects update myapp-prod-456789 \
  --update-labels=env=prod,team=data,owner=idir

# Voir les labels
gcloud projects describe myapp-prod-456789 \
  --format="value(labels)"

# Filtrer les projets par label
gcloud projects list --filter="labels.env=prod"
gcloud projects list --filter="labels.team=data"
```

### Commandes essentielles projets

```bash
# Lister tous les projets
gcloud projects list

# Créer un projet
gcloud projects create PROJECT_ID --name="NAME"

# Voir les détails
gcloud projects describe PROJECT_ID

# Définir comme actif
gcloud config set project PROJECT_ID

# Supprimer (soft delete, 30 jours de grâce)
gcloud projects delete PROJECT_ID

# Restaurer
gcloud projects undelete PROJECT_ID
```

---

## 3. IAM (Identity & Access Management)

### 🔐 Concept clé

**IAM** contrôle **qui** peut faire **quoi** sur **quelles ressources**.

```
Principal (qui) + Role (quoi) + Resource (où) = Permission
```

#### Exemple concret

```yaml
Principal: alice@example.com
Role: Compute Instance Admin
Resource: Projet "myapp-prod"

→ Alice peut créer/modifier/supprimer des VMs dans myapp-prod
```

### 📸 Screenshot 2 : Page IAM
**Emplacement** : `./screenshots/02-iam-page.png`  
**Description** : Interface IAM montrant les membres et leurs rôles

![Page IAM](./screenshots/02-iam-page.png)

### Types de membres (Principals)

| Type | Format | Usage |
|------|--------|-------|
| **Google Account** | `user:alice@gmail.com` | Utilisateur individuel |
| **Service Account** | `sa@project.iam.gserviceaccount.com` | Application/VM |
| **Google Group** | `group:devs@example.com` | Groupe d'utilisateurs |
| **Domain** | `domain:example.com` | Tous les users d'un domaine |
| **All Users** | `allUsers` | ⚠️ Public (éviter !) |

### Rôles Basic (labs uniquement)

> ⚠️ **En production, éviter ces rôles** : trop larges, préférer des rôles spécifiques

#### Viewer (Lecture seule)

```yaml
ID: roles/viewer
Permissions:
  - Lecture de toutes les ressources
  - Aucune modification possible
  - Idéal pour : Audit, monitoring, reporting
```

**Cas d'usage** : Auditeur, manager qui veut voir les ressources sans risque

#### Editor (Modification)

```yaml
ID: roles/editor
Permissions:
  - Toutes les permissions de Viewer
  - Créer, modifier, supprimer des ressources
  - ❌ Ne peut PAS gérer les membres IAM
  - ❌ Ne peut PAS configurer la facturation
```

**Cas d'usage** : Développeur en environnement dev/staging

#### Owner (Propriétaire)

```yaml
ID: roles/owner
Permissions:
  - Toutes les permissions d'Editor
  - ✅ Gérer les rôles IAM (ajouter/supprimer membres)
  - ✅ Configurer la facturation
  - ✅ Supprimer le projet
```

**Cas d'usage** : Admin projet, lead technique

### Rôles prédéfinis (Production)

Au lieu des rôles Basic, utiliser des **rôles spécifiques** :

#### Compute Engine

```bash
roles/compute.instanceAdmin.v1     # Gérer les VMs
roles/compute.networkAdmin         # Gérer le réseau
roles/compute.securityAdmin        # Gérer la sécurité
roles/compute.viewer               # Lecture seule Compute
```

#### Storage

```bash
roles/storage.admin                # Admin buckets
roles/storage.objectAdmin          # Admin objets
roles/storage.objectViewer         # Lecture objets
roles/storage.objectCreator        # Créer objets
```

#### BigQuery

```bash
roles/bigquery.admin               # Admin BigQuery
roles/bigquery.dataEditor          # Modifier données
roles/bigquery.dataViewer          # Lecture données
roles/bigquery.jobUser             # Exécuter des jobs
```

### 📸 Screenshot 3 : Attribution de rôles
**Emplacement** : `./screenshots/03-iam-grant-access.png`  
**Description** : Interface d'attribution de rôles avec recherche de rôles

![Attribution de rôles](./screenshots/03-iam-grant-access.png)

### Principe du moindre privilège

```
❌ Mauvaise pratique :
   Tout le monde a Editor ou Owner

✅ Bonne pratique :
   Chacun a le rôle minimum nécessaire
```

#### Exemple d'architecture IAM

```yaml
Projet: myapp-prod

Membres:
  - alice@example.com:
      - roles/compute.instanceAdmin.v1
      - roles/storage.objectViewer
      
  - bob@example.com:
      - roles/bigquery.dataEditor
      - roles/storage.objectCreator
      
  - devops-sa@project.iam.gserviceaccount.com:
      - roles/compute.admin
      - roles/storage.admin
      
  - group:data-team@example.com:
      - roles/bigquery.dataViewer
```

### Gérer IAM via gcloud

```bash
# Ajouter un membre avec un rôle
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='user:alice@example.com' \
  --role='roles/compute.instanceAdmin.v1'

# Voir la policy IAM du projet
gcloud projects get-iam-policy PROJECT_ID

# Retirer un membre
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member='user:alice@example.com' \
  --role='roles/compute.instanceAdmin.v1'

# Lister les rôles disponibles
gcloud iam roles list

# Chercher un rôle spécifique
gcloud iam roles list --filter="name:compute"
```

### Service Accounts

Les **Service Accounts** sont des comptes pour les applications, pas les humains.

#### Créer un Service Account

```bash
# Créer
gcloud iam service-accounts create my-sa \
  --display-name="My Service Account"

# Donner des permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com' \
  --role='roles/compute.instanceAdmin.v1'

# Lister les SAs
gcloud iam service-accounts list

# Générer une clé (JSON)
gcloud iam service-accounts keys create key.json \
  --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

#### Cas d'usage Service Accounts

```
✅ VM qui accède à Cloud Storage
✅ Application qui écrit dans BigQuery
✅ Pipeline CI/CD qui déploie sur GCP
✅ Cloud Functions qui appelle des APIs
```

---

## 4. APIs & Services

### 🔌 Concept clé

Sur GCP, un projet **ne peut pas utiliser n'importe quel service par défaut**.  
La plupart des services nécessitent l'**activation d'une API**.

```
Service GCP = API à activer
```

### Pourquoi activer des APIs ?

```
✅ Sécurité : contrôle des services utilisés
✅ Coûts : évite l'usage accidentel de services payants
✅ Quotas : gestion des limites par API
✅ Monitoring : tracking de l'usage par API
```

### 📸 Screenshot 4 : API Library
**Emplacement** : `./screenshots/04-api-library.png`  
**Description** : Bibliothèque des APIs avec recherche

![API Library](./screenshots/04-api-library.png)

### Activer une API

#### Via Console Web

1. Navigation menu → **APIs & Services** → **Library**
2. Rechercher l'API (ex : "Dialogflow API")
3. Cliquer sur l'API
4. Bouton **Enable**
5. Attendre l'activation (quelques secondes)

### 📸 Screenshot 5 : Dialogflow API
**Emplacement** : `./screenshots/05-dialogflow-api.png`  
**Description** : Page de détails d'une API avec bouton Enable

![Dialogflow API](./screenshots/05-dialogflow-api.png)

#### Via gcloud CLI

```bash
# Activer une API
gcloud services enable compute.googleapis.com

# Activer plusieurs APIs
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  bigquery.googleapis.com

# Lister les APIs activées
gcloud services list --enabled

# Lister toutes les APIs disponibles
gcloud services list --available

# Désactiver une API (⚠️ peut casser des ressources !)
gcloud services disable dialogflow.googleapis.com
```

### APIs essentielles par service

#### Compute Engine

```bash
compute.googleapis.com          # Compute Engine API
```

#### Storage & Databases

```bash
storage.googleapis.com          # Cloud Storage
sqladmin.googleapis.com         # Cloud SQL
bigtable.googleapis.com         # Cloud Bigtable
firestore.googleapis.com        # Firestore
```

#### Big Data & ML

```bash
bigquery.googleapis.com         # BigQuery
dataflow.googleapis.com         # Dataflow
aiplatform.googleapis.com       # Vertex AI
ml.googleapis.com               # Cloud ML Engine
```

#### Networking

```bash
dns.googleapis.com              # Cloud DNS
cloudresourcemanager.googleapis.com  # Resource Manager
servicenetworking.googleapis.com     # Service Networking
```

### Permissions requises

Pour activer des APIs, il faut avoir :

```yaml
Rôles acceptés:
  - roles/editor
  - roles/owner
  - roles/serviceusage.serviceUsageAdmin
```

### Coûts liés aux APIs

```
⚠️ Activation d'une API = GRATUIT
💰 Utilisation de l'API = Peut être payant

Exemples:
  - Compute Engine API : gratuite
  - Usage des VMs : payant
  
  - BigQuery API : gratuite
  - Requêtes BigQuery : payantes (1 To/mois gratuit)
```

### Quotas et limites

Chaque API a des **quotas** (limites d'utilisation) :

```bash
# Voir les quotas d'une API
gcloud services quota list \
  --service=compute.googleapis.com

# Voir les quotas actifs du projet
gcloud compute project-info describe \
  --project=PROJECT_ID
```

#### Augmenter les quotas

1. Console → **IAM & Admin** → **Quotas**
2. Filtrer par service
3. Sélectionner le quota
4. **Edit Quotas**
5. Demander une augmentation (justification requise)

---

## 5. Compute Engine

### 💻 Concept clé

**Compute Engine** = Service de machines virtuelles (VM) sur GCP.

```
VM = Serveur virtuel avec CPU, RAM, disque, réseau
```

### 📸 Screenshot 6 : Create Instance
**Emplacement** : `./screenshots/06-compute-create-instance.png`  
**Description** : Formulaire de création d'une VM

![Create Instance](./screenshots/06-compute-create-instance.png)

### Créer une VM

#### Via Console Web

1. Navigation menu → **Compute Engine** → **VM instances**
2. Cliquer **Create instance**
3. Configurer :

##### Paramètres de base

| Paramètre | Description | Exemple |
|-----------|-------------|---------|
| **Name** | Nom de la VM | `vm-web-prod-01` |
| **Region** | Zone géographique | `europe-west9` (Paris) |
| **Zone** | Datacenter spécifique | `europe-west9-a` |

##### Machine type (CPU + RAM)

```
Séries populaires:

E2 (General Purpose - Économique)
├── e2-micro      → 0.25 vCPU, 1 GB RAM   (Free Tier éligible)
├── e2-small      → 0.5 vCPU,  2 GB RAM
├── e2-medium     → 1 vCPU,    4 GB RAM
└── e2-standard-4 → 4 vCPU,    16 GB RAM

N2 (Balanced - Production)
├── n2-standard-2 → 2 vCPU,    8 GB RAM
├── n2-standard-4 → 4 vCPU,    16 GB RAM
└── n2-standard-8 → 8 vCPU,    32 GB RAM

C2 (Compute Optimized - Haute perf)
└── c2-standard-4 → 4 vCPU,    16 GB RAM

M2 (Memory Optimized - Gros RAM)
└── m2-ultramem-208 → 208 vCPU, 5888 GB RAM
```

##### Boot Disk (Disque de démarrage)

```yaml
OS disponibles:
  - Debian 11, 12
  - Ubuntu 20.04 LTS, 22.04 LTS, 24.04 LTS
  - CentOS 7, Stream 8, 9
  - RHEL 7, 8, 9
  - Windows Server 2016, 2019, 2022
  - Container-Optimized OS (pour Docker)

Taille disque:
  - Minimum: 10 GB
  - Recommandé: 20-50 GB
  - Production: 100+ GB selon besoin

Type de disque:
  - Standard (HDD) : Pas cher, lent
  - Balanced (SSD) : Bon rapport qualité/prix
  - SSD : Rapide, cher
```

##### Firewall

```yaml
Options:
  ☐ Allow HTTP traffic (port 80)
  ☐ Allow HTTPS traffic (port 443)

Attention:
  - SSH (port 22) est ouvert par défaut
  - Ces options créent des firewall rules automatiques
  - Pour des règles custom, voir section Réseau
```

#### Via gcloud CLI

```bash
# Créer une VM basique
gcloud compute instances create vm-web-01 \
  --zone=europe-west9-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud

# Créer avec options avancées
gcloud compute instances create vm-web-prod \
  --zone=europe-west9-a \
  --machine-type=n2-standard-2 \
  --image-family=ubuntu-2204-lts \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-balanced \
  --tags=http-server,https-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx'
```

### Machine Types - Guide de choix

```
Workload léger (dev, test):
→ e2-micro, e2-small

Application web standard:
→ e2-medium, e2-standard-2

API backend (trafic moyen):
→ n2-standard-2, n2-standard-4

Base de données, cache:
→ m1-megamem (optimisé mémoire)

Calcul intensif (ML, rendering):
→ c2-standard-4, c2-standard-8

GPU (ML training, 3D):
→ n1-standard-4 + GPU Tesla T4/V100
```

### Gérer les VMs

```bash
# Lister toutes les VMs
gcloud compute instances list

# Voir les détails d'une VM
gcloud compute instances describe VM_NAME --zone=ZONE

# Démarrer une VM arrêtée
gcloud compute instances start VM_NAME --zone=ZONE

# Arrêter une VM
gcloud compute instances stop VM_NAME --zone=ZONE

# Redémarrer une VM
gcloud compute instances reset VM_NAME --zone=ZONE

# Supprimer une VM
gcloud compute instances delete VM_NAME --zone=ZONE

# Changer le machine type (VM doit être arrêtée)
gcloud compute instances set-machine-type VM_NAME \
  --machine-type=n2-standard-4 \
  --zone=ZONE
```

### Se connecter à une VM

#### Option 1 : SSH via navigateur (le plus simple)

1. Console → **Compute Engine** → **VM instances**
2. Cliquer le bouton **SSH** à côté de la VM
3. Une fenêtre de terminal s'ouvre dans le navigateur

### 📸 Screenshot 7 : SSH Browser
**Emplacement** : `./screenshots/07-ssh-browser.png`  
**Description** : Connexion SSH via le navigateur

![SSH Browser](./screenshots/07-ssh-browser.png)

#### Option 2 : SSH via Cloud Shell

```bash
# Se connecter
gcloud compute ssh VM_NAME --zone=ZONE

# Exemple
gcloud compute ssh vm-web-01 --zone=europe-west9-a

# Avec un utilisateur spécifique
gcloud compute ssh USER@VM_NAME --zone=ZONE
```

#### Option 3 : SSH local (depuis ton PC)

```bash
# Générer et ajouter ta clé SSH
gcloud compute config-ssh

# Se connecter (après config-ssh)
ssh VM_NAME.ZONE.PROJECT_ID

# Exemple
ssh vm-web-01.europe-west9-a.myapp-prod-456789
```

### Startup Scripts

Les **startup scripts** s'exécutent au démarrage de la VM.

#### Inline startup script

```bash
gcloud compute instances create vm-nginx \
  --zone=europe-west9-a \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx
    echo "Hello from $(hostname)" > /var/www/html/index.html'
```

#### Startup script depuis un fichier

```bash
# Créer startup.sh
cat > startup.sh << 'EOF'
#!/bin/bash
apt-get update
apt-get install -y nginx git
systemctl enable nginx
systemctl start nginx
EOF

# Créer la VM avec le script
gcloud compute instances create vm-nginx \
  --zone=europe-west9-a \
  --metadata-from-file=startup-script=startup.sh
```

#### Startup script depuis Cloud Storage

```bash
# Upload du script dans un bucket
gsutil cp startup.sh gs://my-bucket/scripts/

# Créer la VM
gcloud compute instances create vm-nginx \
  --zone=europe-west9-a \
  --metadata=startup-script-url=gs://my-bucket/scripts/startup.sh
```

### Metadata & Custom Metadata

```bash
# Ajouter des metadata à la création
gcloud compute instances create vm-app \
  --metadata=env=prod,team=data,version=1.0

# Ajouter metadata à une VM existante
gcloud compute instances add-metadata VM_NAME \
  --metadata=new-key=new-value

# Voir les metadata d'une VM
gcloud compute instances describe VM_NAME \
  --format="value(metadata.items)"

# Accéder aux metadata depuis la VM
curl -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/
```

### Disques persistants

```bash
# Créer un disque
gcloud compute disks create my-disk \
  --size=100GB \
  --zone=europe-west9-a \
  --type=pd-balanced

# Attacher un disque à une VM (VM arrêtée)
gcloud compute instances attach-disk VM_NAME \
  --disk=my-disk \
  --zone=ZONE

# Lister les disques
gcloud compute disks list

# Snapshots (backup)
gcloud compute disks snapshot my-disk \
  --snapshot-names=my-snapshot \
  --zone=ZONE

# Créer un disque depuis un snapshot
gcloud compute disks create restored-disk \
  --source-snapshot=my-snapshot \
  --zone=ZONE
```

---

## 6. Réseau & Firewall

### 🌐 Concepts clés

#### IP externe vs interne

```yaml
IP Externe (External IP):
  - Accessible depuis Internet
  - Éphémère (change au redémarrage) ou Statique (fixe)
  - Coûte de l'argent (surtout si statique et inutilisée)

IP Interne (Internal IP):
  - Accessible uniquement dans le VPC
  - Privée (non routable sur Internet)
  - Gratuite
```

#### Voir les IPs d'une VM

```bash
# Console
Compute Engine → VM instances → Colonne "External IP" et "Internal IP"

# gcloud
gcloud compute instances describe VM_NAME \
  --zone=ZONE \
  --format="get(networkInterfaces[0].accessConfigs[0].natIP,networkInterfaces[0].networkIP)"
```

### Firewall Rules

Les **firewall rules** contrôlent le trafic réseau entrant et sortant.

#### Règles par défaut

```yaml
default-allow-internal:
  - Autorise tout le trafic entre VMs du même réseau
  - Protocoles: TCP, UDP, ICMP

default-allow-ssh:
  - Autorise SSH (port 22) depuis n'importe où
  - Source: 0.0.0.0/0

default-allow-rdp (Windows):
  - Autorise RDP (port 3389) depuis n'importe où
  - Source: 0.0.0.0/0

default-allow-icmp:
  - Autorise ping
```

#### Créer une règle firewall

##### Autoriser HTTP (port 80)

```bash
gcloud compute firewall-rules create allow-http \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:80 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=http-server
```

##### Autoriser HTTPS (port 443)

```bash
gcloud compute firewall-rules create allow-https \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=https-server
```

##### Autoriser un port custom

```bash
# Exemple: port 8080 pour une API
gcloud compute firewall-rules create allow-api \
  --direction=INGRESS \
  --priority=1000 \
  --network=default \
  --action=ALLOW \
  --rules=tcp:8080 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=api-server
```

#### Appliquer des tags aux VMs

```bash
# Ajouter un tag à une VM
gcloud compute instances add-tags VM_NAME \
  --tags=http-server,https-server \
  --zone=ZONE

# La VM hérite maintenant des règles avec ces tags
```

#### Lister les règles firewall

```bash
# Toutes les règles
gcloud compute firewall-rules list

# Détails d'une règle
gcloud compute firewall-rules describe RULE_NAME

# Supprimer une règle
gcloud compute firewall-rules delete RULE_NAME
```

#### Bonnes pratiques firewall

```yaml
✅ Utiliser des source ranges spécifiques:
   --source-ranges=203.0.113.0/24  # Ton bureau
   --source-ranges=YOUR_IP/32       # Ton IP seulement

❌ Éviter 0.0.0.0/0 (tout Internet) sauf nécessaire

✅ Utiliser des tags pour grouper les VMs:
   --target-tags=web-servers
   --target-tags=database-servers

✅ Nommer clairement les règles:
   allow-http-from-office
   allow-ssh-from-bastion

✅ Documenter les règles:
   --description="Allow HTTPS from CloudFlare"
```

---

## 7. Cloud Shell & gcloud CLI

### ☁️ Cloud Shell

**Cloud Shell** = Terminal dans le navigateur avec gcloud préconfiguré.

#### Caractéristiques

```yaml
Accès: Icône terminal en haut à droite de la console
Environnement:
  - Debian Linux
  - gcloud SDK préinstallé
  - 5 GB de stockage persistant ($HOME)
  - Python, Node.js, Java, Go préinstallés
  - vim, nano, emacs disponibles
Limites:
  - Session expire après 20 min d'inactivité
  - 60 heures/semaine d'usage
```

#### Ouvrir Cloud Shell

```bash
1. Console GCP → Icône ">_" (en haut à droite)
2. Terminal s'ouvre dans le navigateur
3. Prêt à utiliser gcloud !
```

### gcloud CLI - Configuration

```bash
# Voir la configuration active
gcloud config list

# Définir le projet par défaut
gcloud config set project PROJECT_ID

# Définir la région/zone par défaut
gcloud config set compute/region europe-west9
gcloud config set compute/zone europe-west9-a

# Voir les propriétés disponibles
gcloud config list --all

# Créer une configuration nommée
gcloud config configurations create prod
gcloud config configurations create dev

# Activer une configuration
gcloud config configurations activate prod

# Lister les configurations
gcloud config configurations list
```

### gcloud CLI - Commandes essentielles

#### Authentification

```bash
# Se connecter
gcloud auth login

# Voir le compte actif
gcloud auth list

# Changer de compte
gcloud config set account ACCOUNT

# Service Account authentication
gcloud auth activate-service-account --key-file=key.json
```

#### Projets

```bash
# Lister les projets
gcloud projects list

# Définir le projet actif
gcloud config set project PROJECT_ID

# Voir le projet actif
gcloud config get-value project
```

#### Compute Engine

```bash
# Lister les VMs
gcloud compute instances list

# Créer une VM
gcloud compute instances create VM_NAME

# SSH dans une VM
gcloud compute ssh VM_NAME --zone=ZONE

# Arrêter/Démarrer
gcloud compute instances stop VM_NAME
gcloud compute instances start VM_NAME

# Supprimer
gcloud compute instances delete VM_NAME
```

#### Format de sortie

```bash
# JSON
gcloud compute instances list --format=json

# YAML
gcloud compute instances list --format=yaml

# Table (par défaut)
gcloud compute instances list --format=table

# Extraire une valeur spécifique
gcloud compute instances describe VM_NAME \
  --format="value(networkInterfaces[0].networkIP)"

# CSV
gcloud compute instances list --format="csv(name,zone,status)"
```

#### Filtres

```bash
# Filtrer les VMs en running
gcloud compute instances list --filter="status=RUNNING"

# Filtrer par zone
gcloud compute instances list --filter="zone:europe-west9"

# Filtrer par label
gcloud compute instances list --filter="labels.env=prod"

# Combinaisons
gcloud compute instances list \
  --filter="status=RUNNING AND zone:europe-west9"
```

### Commandes utiles quotidiennes

```bash
# Voir les coûts du projet (estimation)
gcloud billing accounts list
gcloud billing projects describe PROJECT_ID

# Voir les quotas
gcloud compute project-info describe

# Voir les APIs activées
gcloud services list --enabled

# Voir les logs
gcloud logging read "resource.type=gce_instance" --limit=50

# Info sur son environnement
gcloud info

# Aide sur une commande
gcloud compute instances create --help
gcloud help compute instances create  # Équivalent
```

---

## 8. Gestion des coûts

### 💰 Comprendre la facturation GCP

#### Ce qui coûte

```yaml
Compute:
  - VM running : Facturé à la seconde (minimum 1 minute)
  - VM stopped : Disque toujours facturé !
  - IP externe statique : ~7€/mois si inutilisée

Storage:
  - Disques persistants : ~0.04€/GB/mois (SSD)
  - Cloud Storage : ~0.02€/GB/mois (Standard)
  - Snapshots : ~0.026€/GB/mois

Network:
  - Trafic entrant (ingress) : GRATUIT
  - Trafic sortant (egress) : Payant
    - Vers Internet : ~0.12€/GB
    - Entre régions : ~0.02€/GB
    - Même région : GRATUIT
```

#### Tarifs indicatifs VM (europe-west9)

```
e2-micro      : ~0.008€/h = ~6€/mois   (free tier: 1 instance gratuite)
e2-small      : ~0.017€/h = ~12€/mois
e2-medium     : ~0.034€/h = ~25€/mois
n2-standard-2 : ~0.099€/h = ~72€/mois
n2-standard-4 : ~0.198€/h = ~144€/mois

Note: Ces prix sont indicatifs et peuvent varier
```

### Optimisation des coûts

#### 1. Arrêter les VMs inutilisées

```bash
# Arrêter une VM (disque toujours facturé)
gcloud compute instances stop VM_NAME --zone=ZONE

# Supprimer une VM (disque aussi supprimé par défaut)
gcloud compute instances delete VM_NAME --zone=ZONE

# Script pour arrêter toutes les VMs d'un projet
for vm in $(gcloud compute instances list --format="value(name,zone)"); do
  name=$(echo $vm | cut -d' ' -f1)
  zone=$(echo $vm | cut -d' ' -f2)
  gcloud compute instances stop $name --zone=$zone
done
```

#### 2. Supprimer les ressources orphelines

```bash
# Disques non attachés
gcloud compute disks list --filter="-users:*"

# Les supprimer
for disk in $(gcloud compute disks list --filter="-users:*" --format="value(name,zone)"); do
  name=$(echo $disk | cut -d' ' -f1)
  zone=$(echo $disk | cut -d' ' -f2)
  gcloud compute disks delete $name --zone=$zone --quiet
done

# IPs externes statiques non utilisées
gcloud compute addresses list --filter="status=RESERVED"

# Les supprimer
gcloud compute addresses delete ADDRESS_NAME --region=REGION
```

#### 3. Utiliser des labels pour le tracking

```bash
# Ajouter des labels aux ressources
gcloud compute instances add-labels VM_NAME \
  --labels=env=dev,team=data,cost-center=cc-123

# Filtrer par label dans la facturation
# Console → Billing → Reports → Group by: Label
```

#### 4. Committed Use Discounts (CUD)

```
Engagement 1 an : -25% de réduction
Engagement 3 ans : -52% de réduction

Idéal pour :
  - VMs en production (24/7)
  - Workloads prévisibles
```

#### 5. Preemptible VMs (VM préemptibles)

```yaml
Caractéristiques:
  - 80% moins cher
  - Durée max: 24h
  - Peut être arrêtée à tout moment par GCP

Usage:
  - Batch processing
  - CI/CD runners
  - Développement/test
  - Calcul distribué tolérant aux pannes
```

```bash
# Créer une VM préemptible
gcloud compute instances create vm-preempt \
  --preemptible \
  --zone=europe-west9-a
```

### Budgets et alertes

#### Créer un budget

1. Console → **Billing** → **Budgets & alerts**
2. **Create budget**
3. Configurer :
   - Montant (ex: 100€/mois)
   - Alertes à 50%, 90%, 100%
   - Destinataires email

#### Via gcloud

```bash
# Nécessite l'API Cloud Billing Budget
gcloud services enable billingbudgets.googleapis.com

# Créer un budget (exemple basique)
# Note: Plus complexe, voir la doc pour des exemples complets
```

### Voir les coûts

```bash
# Lister les comptes de facturation
gcloud billing accounts list

# Voir les coûts d'un projet (console uniquement)
# Billing → Reports → Filter by Project
```

### Checklist fin de lab

```yaml
Avant de quitter un lab:
  - [ ] Arrêter toutes les VMs
  - [ ] Supprimer les VMs de test
  - [ ] Supprimer les disques non utilisés
  - [ ] Libérer les IPs statiques
  - [ ] Supprimer les firewall rules custom
  - [ ] Vérifier qu'il ne reste aucune ressource

Commandes rapides:
  # Lister tout
  - gcloud compute instances list
  - gcloud compute disks list
  - gcloud compute addresses list
  - gcloud compute firewall-rules list
```

---

## 9. Bonnes pratiques

### 🎯 Organisation

#### 1. Naming conventions strictes

```yaml
Format recommandé:
  {service}-{env}-{purpose}-{number}

Exemples:
  vm-prod-web-01
  vm-dev-api-01
  disk-prod-db-data-01
  fw-prod-allow-https
  sa-prod-gke-nodes
```

#### 2. Labels systématiques

```yaml
Labels minimum:
  env: prod | staging | dev
  team: data | frontend | backend
  owner: alice | bob
  cost-center: cc-1234
  project: myapp
  managed-by: terraform | manual
```

#### 3. Documentation

```yaml
Documenter:
  - Architecture dans un README
  - Diagrammes réseau
  - Procédures de déploiement
  - Contacts des propriétaires
  - Dépendances entre services
```

### 🔒 Sécurité

#### 1. Principe du moindre privilège

```yaml
❌ Mauvais:
   - Tout le monde a Editor
   - Service Accounts avec Owner

✅ Bon:
   - Rôles spécifiques par besoin
   - SA avec permissions minimales
   - Review régulier des accès
```

#### 2. Service Accounts pour les applications

```yaml
❌ Ne JAMAIS:
   - Utiliser son compte perso dans le code
   - Partager des clés JSON par email

✅ Toujours:
   - Créer un SA dédié par application
   - Utiliser Workload Identity (GKE)
   - Rotation des clés JSON
```

#### 3. Firewall rules restrictives

```yaml
❌ Éviter:
   --source-ranges=0.0.0.0/0 (tout Internet)

✅ Préférer:
   --source-ranges=YOUR_OFFICE_IP/32
   --source-ranges=CLOUDFLARE_IPS
   --source-tags=frontend-tier
```

#### 4. Secrets Management

```bash
# ❌ Ne PAS mettre de secrets dans :
- Metadata
- Startup scripts
- Code source
- Variables d'environnement (visibles)

# ✅ Utiliser Secret Manager
gcloud secrets create db-password \
  --data-file=password.txt

# Accéder au secret depuis une VM (via SA)
gcloud secrets versions access latest \
  --secret=db-password
```

### 📊 Monitoring

#### 1. Activer Cloud Monitoring

```bash
# Installer l'agent monitoring (VM)
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
```

#### 2. Logs essentiels

```bash
# Voir les logs d'une VM
gcloud logging read "resource.type=gce_instance AND resource.labels.instance_id=INSTANCE_ID" --limit=50

# Logs d'une application
gcloud logging read "jsonPayload.message=~\"ERROR\"" --limit=50

# Export des logs vers BigQuery (pour analyse)
# Console → Logging → Log Router → Create Sink
```

### 🚀 Automatisation

#### 1. Infrastructure as Code

```yaml
Outils recommandés:
  - Terraform (multi-cloud)
  - Deployment Manager (GCP natif)
  - Pulumi (code TypeScript/Python)

Avantages:
  - Reproductibilité
  - Version control
  - Peer review
  - Rollback facile
```

#### 2. Scripts d'automation

```bash
# Script de backup quotidien
#!/bin/bash
for disk in $(gcloud compute disks list --format="value(name,zone)"); do
  name=$(echo $disk | cut -d' ' -f1)
  zone=$(echo $disk | cut -d' ' -f2)
  snapshot_name="$name-$(date +%Y%m%d)"
  gcloud compute disks snapshot $name \
    --snapshot-names=$snapshot_name \
    --zone=$zone
done
```

---

## 10. Commandes essentielles

### 🔧 Quick Reference

#### Configuration

```bash
# Projet
gcloud config set project PROJECT_ID
gcloud config get-value project

# Région/Zone par défaut
gcloud config set compute/region europe-west9
gcloud config set compute/zone europe-west9-a

# Compte actif
gcloud config set account user@example.com
gcloud auth list
```

#### Compute Engine

```bash
# VMs
gcloud compute instances list
gcloud compute instances create VM_NAME
gcloud compute instances describe VM_NAME
gcloud compute instances start VM_NAME
gcloud compute instances stop VM_NAME
gcloud compute instances delete VM_NAME

# SSH
gcloud compute ssh VM_NAME --zone=ZONE

# Machine types
gcloud compute machine-types list --filter="zone:europe-west9-a"

# Images
gcloud compute images list
gcloud compute images list --project=ubuntu-os-cloud
```

#### Disques

```bash
# Lister
gcloud compute disks list

# Créer
gcloud compute disks create DISK_NAME --size=100GB

# Snapshot
gcloud compute disks snapshot DISK_NAME --snapshot-names=SNAP_NAME

# Supprimer
gcloud compute disks delete DISK_NAME
```

#### Réseau

```bash
# Firewall
gcloud compute firewall-rules list
gcloud compute firewall-rules create RULE_NAME
gcloud compute firewall-rules delete RULE_NAME

# IPs
gcloud compute addresses list
gcloud compute addresses create IP_NAME --region=REGION
gcloud compute addresses delete IP_NAME --region=REGION
```

#### IAM

```bash
# Policy du projet
gcloud projects get-iam-policy PROJECT_ID

# Ajouter membre
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member='user:alice@example.com' \
  --role='roles/viewer'

# Service Accounts
gcloud iam service-accounts list
gcloud iam service-accounts create SA_NAME
gcloud iam service-accounts keys create key.json \
  --iam-account=SA_EMAIL
```

#### APIs

```bash
# Lister activées
gcloud services list --enabled

# Lister disponibles
gcloud services list --available

# Activer
gcloud services enable compute.googleapis.com

# Désactiver
gcloud services disable API_NAME
```

#### Projets

```bash
# Lister
gcloud projects list

# Créer
gcloud projects create PROJECT_ID --name="NAME"

# Supprimer
gcloud projects delete PROJECT_ID

# Labels
gcloud projects update PROJECT_ID --update-labels=KEY=VALUE
```

### One-liners utiles

```bash
# IP externe de toutes les VMs
gcloud compute instances list --format="table(name,EXTERNAL_IP)"

# VMs en running seulement
gcloud compute instances list --filter="status=RUNNING"

# Coût estimé des VMs actives (approximatif)
gcloud compute instances list --format="csv(name,machineType,zone,status)"

# Disques non attachés (potentiel gaspillage)
gcloud compute disks list --filter="-users:*"

# IPs statiques non utilisées (potentiel gaspillage)
gcloud compute addresses list --filter="status=RESERVED"

# Arrêter toutes les VMs d'une zone
gcloud compute instances stop $(gcloud compute instances list --filter="zone:ZONE" --format="value(name)") --zone=ZONE

# Taille totale des disques
gcloud compute disks list --format="value(sizeGb)" | awk '{s+=$1} END {print s " GB"}'
```

---

## 11. Troubleshooting

### 🔍 Problèmes courants

#### 1. Impossible de se connecter en SSH

```yaml
Symptômes:
  - "Connection refused"
  - "Permission denied"

Causes possibles:
  1. Firewall bloque le port 22
  2. VM éteinte
  3. Clés SSH incorrectes

Solutions:
  # Vérifier firewall
  gcloud compute firewall-rules list --filter="allowed[].ports:22"
  
  # Vérifier status VM
  gcloud compute instances describe VM_NAME --format="value(status)"
  
  # Se connecter via console (si firewall OK)
  Console → VM → SSH button
  
  # Vérifier les logs
  gcloud compute instances get-serial-port-output VM_NAME
```

#### 2. VM inaccessible en HTTP

```yaml
Symptômes:
  - Timeout sur http://EXTERNAL_IP
  - "This site can't be reached"

Causes:
  1. Firewall ne permet pas le port 80
  2. Serveur web pas démarré
  3. Mauvaise IP (utiliser External IP)

Solutions:
  # Créer règle firewall HTTP
  gcloud compute firewall-rules create allow-http \
    --allow=tcp:80 \
    --source-ranges=0.0.0.0/0 \
    --target-tags=http-server
  
  # Ajouter tag à la VM
  gcloud compute instances add-tags VM_NAME --tags=http-server
  
  # SSH et vérifier le serveur
  sudo systemctl status nginx
  sudo systemctl start nginx
```

#### 3. Permission denied sur gcloud

```yaml
Symptômes:
  - "Permission denied"
  - "PERMISSION_DENIED"

Causes:
  1. Pas les droits IAM nécessaires
  2. Mauvais compte actif
  3. API pas activée

Solutions:
  # Vérifier le compte actif
  gcloud auth list
  
  # Vérifier les permissions
  gcloud projects get-iam-policy PROJECT_ID | grep $(gcloud config get-value account)
  
  # Activer l'API manquante
  gcloud services enable compute.googleapis.com
```

#### 4. Quota exceeded

```yaml
Symptômes:
  - "Quota exceeded for resource X"

Solutions:
  # Voir les quotas
  gcloud compute project-info describe --format="yaml(quotas)"
  
  # Demander augmentation
  Console → IAM & Admin → Quotas → Select quota → Edit Quotas
```

#### 5. VM lente ou freeze

```yaml
Causes:
  1. CPU/RAM insuffisant
  2. Disque saturé
  3. I/O lent (disque HDD)

Solutions:
  # Arrêter la VM
  gcloud compute instances stop VM_NAME
  
  # Changer machine type
  gcloud compute instances set-machine-type VM_NAME \
    --machine-type=n2-standard-2
  
  # Redémarrer
  gcloud compute instances start VM_NAME
  
  # Upgrade disque vers SSD
  # (nécessite création nouveau disque + migration)
```

### 🩺 Diagnostics

#### Logs VM

```bash
# Serial port output (utile si SSH inaccessible)
gcloud compute instances get-serial-port-output VM_NAME

# Logs système
gcloud logging read "resource.type=gce_instance AND resource.labels.instance_id=INSTANCE_ID"

# SSH et voir logs locaux
sudo journalctl -xe
sudo tail -f /var/log/syslog
```

#### Réseau

```bash
# Depuis la VM, tester connectivité
ping -c 4 8.8.8.8           # Internet
ping -c 4 google.com        # DNS
curl -I https://google.com  # HTTP

# Voir les ports ouverts
sudo netstat -tlnp
sudo ss -tlnp
```

#### Performance

```bash
# CPU/RAM
top
htop  # Plus lisible

# Disque
df -h           # Espace utilisé
du -sh /*       # Taille par dossier
iostat          # I/O stats

# Réseau
iftop           # Trafic réseau
nethogs         # Par process
```

---

## 📚 Ressources officielles

### Documentation

- [GCP Documentation](https://cloud.google.com/docs)
- [Compute Engine](https://cloud.google.com/compute/docs)
- [IAM Best Practices](https://cloud.google.com/iam/docs/best-practices)
- [gcloud CLI Reference](https://cloud.google.com/sdk/gcloud/reference)
- [Pricing Calculator](https://cloud.google.com/products/calculator)

### Formation

- [Google Cloud Skills Boost](https://www.cloudskillsboost.google)
- [Qwiklabs](https://www.qwiklabs.com)
- [Coursera - Google Cloud](https://www.coursera.org/googlecloud)

### Certification

- [Associate Cloud Engineer](https://cloud.google.com/certification/cloud-engineer)
- [Professional Cloud Architect](https://cloud.google.com/certification/cloud-architect)

---

## 📝 Notes personnelles

### Labs terminés

- [ ] A Cloud Guru: Creating GCP Projects
- [ ] Setup Network and HTTP Load Balancers
- [ ] Deploy a Compute Instance with a Remote Startup Script
- [ ] Configure a Firewall and Startup Script with Deployment Manager
- [ ] Working with Virtual Machines
- [ ] Getting Started with Cloud Storage and Cloud SQL

### Points à revoir

_À compléter au fur et à mesure..._

### Questions / Blocages

_À compléter..._

---

## 📸 Organisation des screenshots

**Créer un dossier** `screenshots/` à la racine et y placer :

```
screenshots/
├── 01-projects-list.png          # Liste des projets
├── 02-iam-page.png                # Page IAM avec membres
├── 03-iam-grant-access.png        # Attribution de rôles
├── 04-api-library.png             # Bibliothèque APIs
├── 05-dialogflow-api.png          # Page d'une API
├── 06-compute-create-instance.png # Création VM
└── 07-ssh-browser.png             # SSH via navigateur
```

**Les images seront automatiquement affichées** si elles sont placées dans ce dossier avec les noms exacts.

---

**Version** : 1.0  
**Dernière mise à jour** : Janvier 2026  
**Auteur** : Didir  
**Parcours** : Google Cloud Skills Boost - Essentials
