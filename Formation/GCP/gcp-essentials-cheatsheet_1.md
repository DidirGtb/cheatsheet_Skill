# GCP Essentials — Cheatsheet (Google Cloud Skills Boost / Essentials)

> Cette page sert de mémo rapide (projet, IAM, APIs, VM).  
> Elle est basée sur tes notes de lab et complétée avec des rappels pratiques.

---

## Sommaire
- [1) Projets GCP](#1-projets-gcp)
- [2) IAM (accès & rôles)](#2-iam-accès--rôles)
- [3) APIs & Services](#3-apis--services)
- [4) Compute Engine : créer une VM](#4-compute-engine--créer-une-vm)
- [5) Se connecter à la VM](#5-se-connecter-à-la-vm)
- [6) Coûts & bonnes pratiques](#6-coûts--bonnes-pratiques)
- [7) Nettoyage (fin de lab)](#7-nettoyage-fin-de-lab)
- [8) Mini glossaire](#8-mini-glossaire)

---

## 1) Projets GCP

Dans Google Cloud, la plupart des ressources (VM, buckets, services…) vivent dans un **Project**.

### Pourquoi segmenter en projets ?
- **Séparer les environnements** (dev/test/prod) → évite de casser la prod
- **Facturation & coûts** → dépenses suivies **par projet**
- **Gestion des accès (IAM)** plus simple → rôles attribués au niveau projet
- **Sécurité** : isolation + réduction des risques
- **Maintenance** : meilleure lisibilité

### Où créer / voir ses projets ?
Console → **Project selector** (en haut) → liste des projets → **New Project**

📸 *Zone capture*  
`TODO: ./images/gcp/projects-list.png`
```text
![Projects list](./images/gcp/projects-list.png)
```

**Bonnes pratiques**
- Nommer clairement : `myapp-dev`, `myapp-staging`, `myapp-prod`
- Ajouter des **labels** : `env=dev`, `team=data`, `owner=idir`
- Mettre un **budget + alertes** dès le début (même en perso)

---

## 2) IAM (accès & rôles)

IAM = **Identity and Access Management** : qui peut faire quoi, sur quelles ressources.

### Rôles “Basic” (ceux des labs)
> En entreprise, on évite souvent ces rôles car ils sont trop larges. En lab, ils sont très pratiques.

#### Viewer
- **Lecture seule**
- Aucune action modifiant l’état des ressources

#### Editor
- Inclut toutes les permissions du Viewer
- Peut **créer / modifier / supprimer** des ressources Google Cloud
- ❌ Ne peut pas **ajouter/supprimer des membres** d’un projet

#### Owner
- Inclut toutes les permissions Editor
- Peut **gérer les rôles IAM** du projet et des ressources
- Peut **configurer la facturation** (billing)

📸 *Zone capture*  
`TODO: ./images/gcp/iam-page.png`
```text
![IAM page](./images/gcp/iam-page.png)
```

### Ajout utile : rôles “spécifiques”
Au lieu d’Editor, préfère souvent des rôles ciblés :
- `Compute Admin`, `Storage Admin`, `BigQuery Admin`, etc.

Principe clé : **Least privilege** = donner le minimum nécessaire.

---

## 3) APIs & Services

Sur GCP, un projet ne peut pas utiliser “n’importe quel service” par défaut.  
La plupart des services passent par une **API** à **activer**.

### Points importants
- Il faut parfois un rôle IAM adapté :
  - `Editor` (souvent suffisant en lab)
  - ou `Service Usage Admin` (spécifique à l’activation/gestion d’APIs)
- Activer une API **ne facture pas forcément**,
  mais **l’usage** peut coûter selon le service & le volume.

### Activer une API (console)
1. Navigation menu → **APIs & Services** → **Library**
2. Rechercher l’API (ex : *Dialogflow API*)
3. Cliquer **Enable**

📸 *Zone captures*  
`TODO: ./images/gcp/api-library.png`  
`TODO: ./images/gcp/api-dialogflow.png`
```text
![API Library](./images/gcp/api-library.png)
![Dialogflow API](./images/gcp/api-dialogflow.png)
```

---

## 4) Compute Engine : créer une VM

Console → **Compute Engine** → **VM instances** → **Create instance**

📸 *Zone capture*  
`TODO: ./images/gcp/compute-create-instance.png`
```text
![Create instance](./images/gcp/compute-create-instance.png)
```

### Paramètres essentiels à comprendre
- **Name** : ex `vm-lab-1`
- **Region / Zone** : proche de tes utilisateurs (latence) et selon dispo/prix
- **Machine type** : CPU/RAM (plus tu montes, plus ça coûte)
- **Boot disk** : OS (Debian/Ubuntu/Windows), taille du disque
- **Firewall** :
  - cocher **Allow HTTP traffic** si tu veux exposer un serveur web (port 80)
  - cocher **Allow HTTPS traffic** si tu veux exposer du TLS (port 443)

### Erreurs fréquentes
- VM créée mais inaccessible : firewall pas ouvert (HTTP/HTTPS/SSH) ou IP mal utilisée
- Choix Windows : nécessite souvent RDP + règles + mot de passe Windows

---

## 5) Se connecter à la VM

### Option A — SSH dans le navigateur (le plus simple)
Compute Engine → VM instances → bouton **SSH**

📸 *Zone capture*  
`TODO: ./images/gcp/ssh-browser.png`
```text
![SSH browser](./images/gcp/ssh-browser.png)
```

### Option B — Cloud Shell + gcloud (pratique en lab)
1. Ouvrir **Cloud Shell** (icône terminal)
2. Se connecter :
```bash
gcloud compute ssh <VM_NAME> --zone <ZONE>
```

**Lister les VMs :**
```bash
gcloud compute instances list
```

**Définir un projet par défaut :**
```bash
gcloud config set project <PROJECT_ID>
```

### Vérifier l’IP externe
Compute Engine → VM instances → colonne **External IP**  
(utile pour accéder en HTTP/HTTPS si un serveur tourne dessus)

---

## 6) Coûts & bonnes pratiques

### Ce qui coûte souvent (même en “basique”)
- VM qui tourne 24/7
- Disques persistants (même si VM éteinte)
- IP externe réservée (surtout si statique)
- Trafic réseau sortant (egress)

### Bonnes pratiques simples
- **Arrêter** la VM quand tu n’en as pas besoin (Stop)
- Supprimer les ressources de lab à la fin
- Mettre des **budgets** et alertes sur le projet (Billing)

---

## 7) Nettoyage (fin de lab)

À la fin d’un lab :
- Stop / Delete les VMs
- Supprimer les ressources créées (firewall rules spécifiques, disques, IP statiques)
- Vérifier la page Billing si tu es hors environnement lab

Checklist rapide :
- [ ] VM supprimée
- [ ] Disque supprimé (si créé séparément)
- [ ] IP statique libérée (si réservée)
- [ ] Règles firewall custom supprimées

---

## 8) Mini glossaire

- **Project** : conteneur logique (ressources, facturation, IAM)
- **IAM** : gestion des identités et permissions
- **API** : interface à activer pour utiliser un service
- **Compute Engine** : service de VM
- **Zone/Region** : localisation des ressources
- **Firewall rule** : autorisations réseau (ports/protocoles)
- **Cloud Shell** : terminal dans le navigateur avec gcloud prêt

---

## Notes perso (à compléter)
- ✅ Lab terminé : …
- ❗ Points à revoir : …
- 🔗 Liens utiles : …
