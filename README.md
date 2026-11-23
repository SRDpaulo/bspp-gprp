# Site BSPP GPRP — Suivi recrutement & formation

Site de suivi des sapeurs de la BSPP sur Grand Paris RP : comptes par rôle, création de comptes en fin d'entretien, et bientôt le suivi des formations et des compétences.

## Fichiers

| Fichier | Rôle |
| --- | --- |
| `index.html` | Le site complet (page de connexion, tableau de bord, fiches). |
| `firestore.rules` | Les règles de sécurité à coller dans Firebase. |
| `README.md` | Ce guide. |

## Rôles

| Rôle | Ce qu'il peut faire |
| --- | --- |
| Responsable recrutement | Voit tous les comptes, crée tous les types de comptes, modifie rôle / grade / affectation, désactive un compte. |
| Recruteur | Voit tous les comptes, crée uniquement des comptes recrue. |
| Formateur | Voit tous les comptes (le suivi des formations viendra ensuite). |
| Recrue / Sapeur (compte de base) | Voit uniquement sa propre fiche. |

---

## Étape 0 — Tester tout de suite (mode démo)

Ouvre `index.html` dans ton navigateur (double-clic).
Connexion : identifiant **responsable**, mot de passe **bspp2026**.

En mode démo, les comptes sont enregistrés uniquement dans **ton** navigateur. Personne d'autre ne les voit. C'est pour tester, pas pour la vraie utilisation.

---

## Étape 1 — Mettre le site en ligne avec GitHub Pages (gratuit)

1. Crée un compte sur [github.com](https://github.com) si tu n'en as pas.
2. En haut à droite : **+** → **New repository**.
   - Nom : `bspp-gprp` (ou ce que tu veux).
   - Visibilité : **Public** (obligatoire pour GitHub Pages en gratuit).
   - Clique **Create repository**.
3. Sur la page du dépôt : **Add file** → **Upload files**. Glisse `index.html`, `firestore.rules` et `README.md`, puis **Commit changes**.
4. Va dans **Settings** → **Pages** (menu de gauche).
   - **Source** : *Deploy from a branch*.
   - **Branch** : `main`, dossier `/ (root)` → **Save**.
5. Attends 1 à 2 minutes et recharge la page. L'adresse du site s'affiche, du type :
   `https://ton-pseudo.github.io/bspp-gprp/`

À ce stade le site est en ligne, mais toujours en mode démo. Le dépôt est public : n'importe qui peut lire le code, mais pas les données (elles seront dans Firebase, protégées).

Pour mettre à jour le site plus tard : **Add file → Upload files** avec le nouveau `index.html` (il remplace l'ancien), puis attendre 1 minute.

---

## Étape 2 — Brancher Firebase (la vraie base de comptes, gratuite)

GitHub Pages n'héberge que des pages, sans base de données. Firebase (Google) fournit la connexion et la base de données. Le forfait gratuit (Spark) suffit largement et ne demande pas de carte bancaire.

### 2.1 Créer le projet

1. Va sur [console.firebase.google.com](https://console.firebase.google.com) avec un compte Google.
2. **Créer un projet** → nom `bspp-gprp` → tu peux désactiver Google Analytics → **Créer**.

### 2.2 Activer la connexion

1. Menu de gauche : **Build** (Créer) → **Authentication** → **Commencer**.
2. Onglet **Sign-in method** → **E-mail/Mot de passe** → active la première option → **Enregistrer**.
3. Onglet **Settings** → **Domaines autorisés** → **Ajouter un domaine** → `ton-pseudo.github.io`.

### 2.3 Créer la base de données

1. **Build** → **Firestore Database** → **Créer une base de données**.
2. Région : une région européenne (ex. `europe-west9` Paris ou `europe-west1`).
3. Mode : **production**.
4. Onglet **Règles** : efface tout, colle le contenu de `firestore.rules`, puis **Publier**.

### 2.4 Récupérer la configuration

1. Roue dentée en haut à gauche → **Paramètres du projet**.
2. En bas, **Vos applications** → icône **`</>`** (Web) → surnom `site` → **Enregistrer l'application** (pas besoin de Firebase Hosting).
3. Firebase affiche un bloc `const firebaseConfig = { apiKey: ..., ... }`. Copie l'objet entre accolades.
4. Dans `index.html`, cherche la ligne :
   ```js
   const FIREBASE_CONFIG = null;
   ```
   et remplace `null` par l'objet copié :
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "AIza...",
     authDomain: "bspp-gprp.firebaseapp.com",
     projectId: "bspp-gprp",
     storageBucket: "...",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
   Ces valeurs ne sont pas secrètes : la sécurité vient des règles Firestore.
5. Renvoie `index.html` sur GitHub (Upload files).

### 2.5 Créer ton compte de responsable (une seule fois, à la main)

Le tout premier compte se crée dans Firebase, car personne n'est encore connecté pour en créer.

1. **Authentication** → **Users** → **Ajouter un utilisateur**.
   - E-mail : `ton.identifiant@bspp-gprp.fr` (ex. `paul.morenti@bspp-gprp.fr`). Aucun mail n'est envoyé, c'est juste un format technique.
   - Mot de passe : au moins 6 caractères.
2. Copie l'**UID** de l'utilisateur créé (colonne « UID utilisateur »).
3. **Firestore Database** → **Commencer une collection** → ID : `users`.
   - **ID du document** : colle l'UID.
   - Ajoute ces champs :

| Champ | Type | Valeur |
| --- | --- | --- |
| `identifiant` | string | `paul.morenti` |
| `prenom` | string | `Paul` |
| `nom` | string | `Morenti` |
| `role` | string | `responsable` |
| `grade` | string | `Caporal` |
| `cs` | string | `CS Ménilmontant` |
| `matricule` | string | *(vide)* |
| `actif` | boolean | `true` |

4. Va sur ton site, connecte-toi avec `paul.morenti` et ton mot de passe. Le bandeau « Mode démo » a disparu : tu es sur la vraie base.

À partir de là, tous les autres comptes se créent depuis le site, avec le bouton **+ Créer un compte**.

---

## Bon à savoir

- **Identifiant** : sur le site, on tape juste `prenom.nom`. Le site ajoute `@bspp-gprp.fr` tout seul.
- **Supprimer quelqu'un** : utilise **Modifier → Statut : Désactivé**. La personne ne peut plus se connecter et l'historique est gardé. Pour une suppression définitive, supprime l'utilisateur dans **Authentication** et son document dans **Firestore**.
- **Mot de passe oublié** : pour l'instant, le responsable le change dans Firebase (Authentication → menu ⋮ de l'utilisateur). Un bouton dans le site pourra être ajouté plus tard.
- **Changer le domaine technique** : la constante `DOMAINE_IDENTIFIANT` en haut du script. À fixer une fois pour toutes avant de créer des comptes.

## Prochaines étapes prévues

- Fiche sapeur détaillée : modules de FI validés (M1 à M6, évaluation finale), FISP (permis VL/PL, SAV, INC), qualifications.
- Accès formateur pour valider un module et noter un élève.
- Historique : qui a validé quoi, et quand.
