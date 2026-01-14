# Lab GSP001 — Create a VM on Google Cloud (Compute Engine)

> Cheatsheet / mémo rapide basé sur le lab **Create a VM on Google Cloud (GSP001)**.  
> Objectifs : créer une VM via la **Console**, créer une VM via **gcloud**, déployer **NGINX** et tester l’accès web.

---

## Sommaire
- [0) Prérequis & règles du lab](#0-prérequis--règles-du-lab)
- [1) Activer Cloud Shell & vérifier gcloud](#1-activer-cloud-shell--vérifier-gcloud)
- [2) Régions & zones](#2-régions--zones)
- [3) Task 1 — Créer une VM via la Console](#3-task-1--créer-une-vm-via-la-console)
- [4) Task 2 — Installer NGINX et tester](#4-task-2--installer-nginx-et-tester)
- [5) Task 3 — Créer une VM via gcloud](#5-task-3--créer-une-vm-via-gcloud)
- [6) Commandes utiles (gcloud / compute)](#6-commandes-utiles-gcloud--compute)
- [7) Erreurs fréquentes](#7-erreurs-fréquentes)
- [8) Nettoyage (hors lab / perso)](#8-nettoyage-hors-lab--perso)

---


## 1) Activer Cloud Shell & vérifier gcloud

### Activer Cloud Shell
Console → bouton **Activate Cloud Shell** (barre du haut) → **Authorize**

Cloud Shell = une VM prête à l’emploi avec `gcloud`, `git`, etc.  
+ un home persistant (~5GB).

📸 *Zone capture*
- `TODO: ./images/gcp/activate-cloud-shell.png`

```text
![Activate Cloud Shell](./images/gcp/activate-cloud-shell.png)
```

### Vérifier l’account actif
```bash
gcloud auth list
```

### Vérifier le project ID
```bash
gcloud config list project
```

---

## 2) Régions & zones

- **Region** = zone géographique (ex : `us-central1`)
- **Zone** = sous-division dans la région (ex : `us-central1-c`)
- Les ressources **zonal** (VM, persistent disk) doivent être dans **la même zone** pour s’attacher.
- Une IP statique doit être dans **la même région** que la VM.

### Set region/zone (dans Cloud Shell)
```bash
gcloud config set compute/region us-central1
export REGION=us-central1
export ZONE=us-central1-c
```

📸 *Zone capture*
- `TODO: ./images/gcp/regions-zones.png`

```text
![Regions & zones](./images/gcp/regions-zones.png)
```

---

## 3) Task 1 — Créer une VM via la Console

### Chemin
Console → **Compute Engine** → **VM instances** → **Create Instance**

### Configuration (lab)
**Machine configuration**
- Name: `gcelab`
- Region: `us-central1`
- Zone: `us-central1-c`
- Series: `E2`
- Machine type: `e2-medium` (2 vCPU, 4GB RAM)

**OS and storage**
- OS: `Debian`
- Version: `Debian GNU/Linux 12 (bookworm)`
- Boot disk type: `Balanced persistent disk`
- Size: `10 GB`

**Networking**
- Firewall: ✅ **Allow HTTP traffic** (ouvre le port 80 via une règle firewall automatique)

Puis **Create**.

📸 *Zone captures*
- `TODO: ./images/gcp/create-instance-form.png`
- `TODO: ./images/gcp/allow-http-traffic.png`
- `TODO: ./images/gcp/vm-list-gcelab.png`

```text
![Create instance form](./images/gcp/create-instance-form.png)
![Allow HTTP traffic](./images/gcp/allow-http-traffic.png)
![VM list](./images/gcp/vm-list-gcelab.png)
```

### Se connecter en SSH (browser)
Dans la liste des VM → bouton **SSH** en face de `gcelab`.

📸 *Zone capture*
- `TODO: ./images/gcp/ssh-button.png`

```text
![SSH button](./images/gcp/ssh-button.png)
```

---

## 4) Task 2 — Installer NGINX et tester

Dans le terminal SSH de la VM :

### 1) Update
```bash
sudo apt-get update
```

### 2) Installer nginx
```bash
sudo apt-get install -y nginx
```

### 3) Vérifier que nginx tourne
```bash
ps auwx | grep nginx
```

Tu dois voir un process **master** et des **worker** nginx.

### 4) Tester depuis le navigateur
Dans la Console → récupère la colonne **External IP** de `gcelab`, puis ouvre :

- `http://EXTERNAL_IP/`

Tu dois voir : **Welcome to nginx!**

📸 *Zone captures*
- `TODO: ./images/gcp/external-ip.png`
- `TODO: ./images/gcp/nginx-welcome.png`

```text
![External IP](./images/gcp/external-ip.png)
![Welcome to nginx](./images/gcp/nginx-welcome.png)
```

---

## 5) Task 3 — Créer une VM via gcloud

### Créer une VM (gcloud)
Depuis Cloud Shell :

```bash
gcloud compute instances create gcelab2 --machine-type e2-medium --zone=$ZONE
```

### Defaults (dans le lab)
- Image : **Debian GNU/Linux 12 (bookworm)**
- Machine type : **e2-medium**
- Disk root : créé automatiquement, même nom que l’instance, attaché automatiquement

📸 *Zone capture*
- `TODO: ./images/gcp/gcloud-create-instance.png`

```text
![gcloud create](./images/gcp/gcloud-create-instance.png)
```

### Aide gcloud
```bash
gcloud compute instances create --help
# Quitter l'aide :
# CTRL + C
```

### SSH via gcloud
```bash
gcloud compute ssh gcelab2 --zone=us-central1-c
```

Si demandé :
- `Y` pour continuer
- Passphrase : appuyer sur Enter (vide) si tu ne veux pas en mettre

### Quitter la VM
```bash
exit
```

---

## 6) Commandes utiles (gcloud / compute)

### Lister les VM
```bash
gcloud compute instances list
```

### Définir le projet actif
```bash
gcloud config set project <PROJECT_ID>
```

### Définir zone/région par défaut (évite de mettre --zone)
```bash
gcloud config set compute/zone us-central1-c
gcloud config set compute/region us-central1
```

---

## 7) Erreurs fréquentes

### Je n’ai pas la page nginx
- Tu as oublié ✅ **Allow HTTP traffic**
- Tu ouvres `https://` au lieu de `http://`
- Tu n’utilises pas la **bonne External IP**

### SSH ne marche pas
- VM pas “RUNNING”
- Problème d’auth (clé SSH en cours de génération)
- Zone incorrecte avec `gcloud compute ssh` (mettre `--zone`)

### Je perds mes variables REGION/ZONE
- Normal dans Cloud Shell si nouvelle session → refaire :
```bash
export REGION=us-central1
export ZONE=us-central1-c
```

---



