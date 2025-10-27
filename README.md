
# 📚 Cheat Sheets – Site statique

Ce projet est un **site statique** qui regroupe mes différentes *cheat sheets* (fiches pratiques).  
Il est conçu pour être léger, rapide et accessible depuis n’importe quel appareil (desktop ou mobile).
link : https://didircheatsheet.netlify.app/

## 🚀 Objectif
- Avoir un accès rapide à mes fiches techniques (SAS, Git/GitHub, Linux, AZ-900, etc.)
- Pouvoir les consulter facilement sur mobile (format épuré et responsive)
- Centraliser mes notes en un seul endroit, statique et sans dépendance lourde

## 🛠️ Stack technique
- **HTML/CSS** uniquement → pas de backend
- **Structure simple** : chaque cheat sheet est une page HTML
- **Style global** défini dans `/assets/css/style.css`
- **Hébergement** : déployé gratuitement sur [Netlify](https://www.netlify.com/)

## 📂 Structure des fichiers
``` shell
├── index.html # Page d’accueil avec la liste des cheat sheets
├── cheatsheets/
│ ├── sas.html # Fiche SAS
│ ├── git.html # Fiche Git/GitHub
│ ├── linux.html # Fiche Linux
│ ├── az-900.html # Fiche AZ-900 (cloud)
│ └── autres.html # Autres fiches
├── assets/
  ├── css/style.css # Styles globaux
  └── js/copy.js # Script utilitaire (copie de code)
```

## 🔎 Fonctionnalités
- Navigation simple via des **cartes cliquables**
- Contenu **statique et rapide**
- Blocs de code encadrés pour une meilleure lisibilité
- Compatible avec mobile/tablette

## 🌍 Déploiement
Le site est hébergé sur **Netlify**.  
Chaque push sur la branche principale déclenche automatiquement une mise à jour en ligne.

---

✍️ **Note personnelle :** ce projet évolue avec l’ajout de nouvelles fiches au fur et à mesure de mes apprentissages.
