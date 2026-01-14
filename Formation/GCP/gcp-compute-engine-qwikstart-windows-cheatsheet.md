# Compute Engine — Qwik Start (Windows) : VM + RDP (Cheatsheet / Tuto)

> Objectif : **créer une VM Windows Server sur Google Cloud** et **s’y connecter en RDP** (bureau distant).  
> Ce tuto est volontairement **concret** (mode cheatsheet), avec explications de concepts quand c’est utile.

---

## TL;DR (checklist rapide)

1. Ouvrir **Cloud Shell** → vérifier projet/compte  
2. Créer une VM **Windows Server 2022** (Compute Engine)  
3. Attendre que Windows soit prêt (serial port logs)  
4. Générer / reset le **mot de passe** de l’utilisateur `admin`  
5. Se connecter en **RDP** sur l’**External IP**  
6. (Optionnel) sécuriser : règles firewall / IAP / IP restrictions

---

## Sommaire
- [0) Pré-requis](#0-pré-requis)
- [1) Concepts à connaître](#1-concepts-à-connaître)
- [2) Activer Cloud Shell + vérifier le contexte](#2-activer-cloud-shell--vérifier-le-contexte)
- [3) Créer une VM Windows Server (Console)](#3-créer-une-vm-windows-server-console)
- [4) Savoir quand Windows est prêt (Serial Port Output)](#4-savoir-quand-windows-est-prêt-serial-port-output)
- [5) Créer / reset le mot de passe Windows](#5-créer--reset-le-mot-de-passe-windows)
- [6) Se connecter en RDP (Windows / Mac / Chrome)](#6-se-connecter-en-rdp-windows--mac--chrome)
- [7) Copier/coller en RDP](#7-copiercoller-en-rdp)
- [8) Commandes utiles](#8-commandes-utiles)
- [9) Dépannage (rapide)](#9-dépannage-rapide)
- [10) Sécurité (bonnes pratiques)](#10-sécurité-bonnes-pratiques)
- [11) Nettoyage](#11-nettoyage)


---

## 1) Concepts à connaître

### VM Windows sur Compute Engine
Une VM = un ordinateur dans un datacenter Google.  
En choisissant **Windows Server 2022 Datacenter**, tu lances un OS serveur (comme un serveur physique), mais managé au niveau infra par Google.

### External IP
C’est l’IP publique pour joindre ta VM depuis Internet (RDP/HTTP/etc).  
⚠️ Elle peut changer si tu stop/start la VM (sauf IP statique).

### RDP (Remote Desktop Protocol)
Protocole bureau distant (port le plus courant : **TCP 3389**).  
Pour que ça marche :
- VM “ready”
- firewall autorise le trafic RDP (souvent déjà présent en lab)
- tu as un user + password valides

---

## 2) Activer Cloud Shell & vérifier le contexte

Console → bouton **Activate Cloud Shell** → **Authorize**.  
Cloud Shell = VM gérée avec `gcloud` préinstallé + authentifiée au projet.

📸 *Zone capture*  
`TODO: ./images/gcp/windows-qwikstart/activate-cloud-shell.png`
```text
![Activate Cloud Shell](./images/gcp/windows-qwikstart/activate-cloud-shell.png)
```

### Vérifier le compte actif
```bash
gcloud auth list
```

### Vérifier le project id
```bash
gcloud config list project
```

---

## 3) Créer une VM Windows Server (Console)

Chemin : **Compute Engine → VM instances → Create instance**

Paramètres (ceux du lab, à retenir) :
- **Region** : `us-west4`
- **Zone** : `us-west4-a`
- **Series** : `E2`
- **OS and storage** → **Change**
  - Operating system : **Windows Server**
  - Version : **Windows Server 2022 Datacenter**
- Puis **Create**

📸 *Zone captures*  
`TODO: ./images/gcp/windows-qwikstart/create-instance.png`  
`TODO: ./images/gcp/windows-qwikstart/os-storage-windows-2022.png`
```text
![Create instance](./images/gcp/windows-qwikstart/create-instance.png)
![Windows Server 2022 selection](./images/gcp/windows-qwikstart/os-storage-windows-2022.png)
```

**Pourquoi on choisit bien la zone/région ?**
- latence (plus proche = mieux)
- disponibilité des types de machines
- coûts (peuvent varier)

---

## 4) Savoir quand Windows est prêt (Serial Port Output)

Après création, la VM passe en **RUNNING** (icône verte).  
Mais Windows peut ne pas accepter tout de suite le RDP : il lui faut du temps pour initialiser les composants OS.

### Vérifier l’état “ready for RDP” via les logs
Dans Cloud Shell :

```bash
gcloud compute instances get-serial-port-output <INSTANCE_NAME> --zone=us-west4-a
```

- Si on te demande une confirmation, répondre `N` (comme dans le lab).
- Relancer la commande plusieurs fois jusqu’à voir un message indiquant que l’instance est prête à accepter le RDP.

📸 *Zone capture*  
`TODO: ./images/gcp/windows-qwikstart/serial-port-ready.png`
```text
![Serial port ready](./images/gcp/windows-qwikstart/serial-port-ready.png)
```

**À retenir :**  
Le *serial port output* est un “journal bas niveau” utile quand une VM ne boote pas, ou quand un OS met du temps à se configurer.

---

## 5) Créer / reset le mot de passe Windows

Pour te connecter en RDP, il te faut un user + password.

Dans Cloud Shell, on (re)génère un mot de passe pour l’utilisateur **admin** :

```bash
gcloud compute reset-windows-password <INSTANCE_NAME> --zone us-west4-a --user admin
```

- Quand demandé : répondre `Y`
- **Copie le mot de passe** affiché (tu en auras besoin pour te connecter)

📸 *Zone capture*  
`TODO: ./images/gcp/windows-qwikstart/reset-windows-password.png`
```text
![Reset windows password](./images/gcp/windows-qwikstart/reset-windows-password.png)
```

---

## 6) Se connecter en RDP (Windows / Mac / Chrome)

Tu as besoin de :
- **External IP** de la VM (Compute Engine → VM instances)
- user : `admin`
- password : celui généré

📸 *Zone capture*  
`TODO: ./images/gcp/windows-qwikstart/external-ip.png`
```text
![External IP](./images/gcp/windows-qwikstart/external-ip.png)
```

### Option A — Tu es sur Windows
1. Ouvre **Remote Desktop Connection**
2. Computer = `EXTERNAL_IP`
3. User = `admin`
4. Password = celui généré
5. Accepte l’avertissement certificat si demandé

### Option B — Tu es sur Mac
1. Installe **Microsoft Remote Desktop**
2. Ajoute un PC avec l’IP externe
3. Username : `admin`
4. Domain : (laisser vide / ignorer si demandé)
5. Connect

### Option C — Tu es sur Chrome (Spark View)
1. Installer l’extension **Spark View (RDP)**
2. Launch app
3. Domain / Address : mets l’**External IP**
4. Username : `admin`
5. Password : celui généré
6. Connect

✅ Une fois connecté, tu dois voir le **bureau Windows Server**.

📸 *Zone capture*  
`TODO: ./images/gcp/windows-qwikstart/windows-desktop.png`
```text
![Windows desktop](./images/gcp/windows-qwikstart/windows-desktop.png)
```

---

## 7) Copier/coller en RDP

Tu vas souvent copier/coller des commandes depuis tes notes.

- Coller : **CTRL + V**
- Sur Mac : **CMD + V ne marche pas** dans certains clients RDP → utiliser CTRL+V
- Dans un terminal PowerShell : clique d’abord dedans sinon le raccourci ne s’applique pas
- Avec PuTTY : souvent *clic droit* pour coller

---

## 8) Commandes utiles

### Lister les VM
```bash
gcloud compute instances list
```

### Décrire une VM (voir ses détails)
```bash
gcloud compute instances describe <INSTANCE_NAME> --zone us-west4-a
```

### Voir les règles firewall (utile si RDP ne passe pas)
```bash
gcloud compute firewall-rules list
```

---

## 9) Dépannage (rapide)

### “RDP ne se connecte pas”
Checklist :
- [ ] VM est bien **RUNNING**
- [ ] Tu utilises la **bonne External IP**
- [ ] Windows est “ready” (serial port output)
- [ ] Mot de passe correct (sinon re-run `reset-windows-password`)
- [ ] Firewall autorise TCP 3389 (ou règle équivalente)

### “Je ne vois pas de message ready dans les logs”
- Attends + relance la commande
- Si ça dure trop : vérifier que l’instance n’est pas en boot-loop (serial output te donnera des indices)

---

## 10) Sécurité (bonnes pratiques)

En lab on fait simple, mais en vrai :

- Éviter d’ouvrir RDP au monde (`0.0.0.0/0`)
- Restreindre la règle firewall à **ton IP**
- Utiliser **IAP TCP forwarding** (plus safe) pour éviter une IP publique
- Mettre des mots de passe forts, rotation, et idéalement une gestion centralisée (AD/SSO selon contexte)


