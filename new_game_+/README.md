# 🦅 Projet Solo : Le Radar du Marché Data

🚨 **Attention : Mode Tutoriel désactivé.**

Vous avez terminé le projet Vélib'. Vous savez connecter des briques (API -> Python -> BQ -> dbt -> Viz). C'est bien.
Maintenant, on passe au niveau réel. Pas de cheat codes, pas de code à copier-coller. Juste une documentation officielle et votre cerveau.

---

## 🎯 L'Objectif Business

Vous cherchez à vous reconvertir dans la Data.
**Votre mission :** Construire un outil qui analyse en temps réel le marché de l'emploi Data en France via l'API officielle de France Travail (ex-Pôle Emploi).

**Les questions auxquelles votre Dashboard doit répondre :**
1.  Quel est le salaire moyen *réel* mentionné dans les offres "Data Analyst" vs "Data Engineer" ?
2.  Quelles sont les compétences techniques (Hard Skills) les plus demandées ? (SQL apparait-il plus souvent que Python ?)
3.  Où sont les jobs ? (Paris vs Reste de la France).

---

## ⚙️ La Stack Technique (Imposée)

* **Source :** API "Offres d'emploi v2" (France Travail).
* **Ingestion :** Python (Script avec gestion d'authentification OAuth2 & Pagination).
* **Storage :** BigQuery.
* **Transformation :** dbt (Gros focus sur le nettoyage de texte et les Regex).
* **Viz :** Looker Studio.

---

## 💀 Les 3 Boss Techniques du projet

Ce projet est difficile à cause de ces trois points. Si vous les passez, vous êtes techniquement prêts pour un job.

### 1. L'Authentification OAuth2 (Python)
L'API France Travail n'est pas ouverte comme celle des Vélib'. Elle est protégée.
Vous ne pouvez pas juste faire un `requests.get(url)`.
Il faut implémenter le protocole **OAuth2 (Client Credentials)** :
1.  Envoyer votre `client_id` et `client_secret` à une URL d'authentification.
2.  Récupérer un "Token" (jeton) temporaire.
3.  Utiliser ce Token pour interroger l'API des offres.
4.  Gérer l'expiration du Token (il ne dure que 20 minutes !).

### 2. La Pagination (Python)
L'API ne vous donnera pas toutes les offres d'un coup. Elle donne les 150 premières.
Vous devez écrire une **boucle** en Python qui :
* Récupère la page 1.
* Vérifie s'il y a une suite.
* Récupère la page 2, etc.
* S'arrête quand il n'y a plus rien.

### 3. Le Parsing de Texte & Regex (SQL/dbt)
Le champ "Salaire" dans les offres est un champ texte libre rempli par des humains.
Exemples de ce que vous allez recevoir :
* *"35k à 45k selon profil"*
* *"Annuel de 30000.0 Euros à 40000.0 Euros sur 12.0 mois"*
* *"Selon expérience"*

Vous allez devoir utiliser des **Regular Expressions (Regex)** dans dbt pour extraire les chiffres et calculer une moyenne fiable.

---

## 🗺️ Roadmap & Ressources

### Étape 1 : Obtenir les clés du camion
Il faut créer un compte développeur sur la plateforme de l'État.
* **Site :** [France Travail / Emploi Store Dev](https://www.emploi-store-dev.fr/portail-developpeur)
* Créez un compte.
* Abonnez-vous à l'API **"Offres d'emploi v2"**.
* Récupérez vos `Client ID` et `Client Secret`.

### Étape 2 : Le Script d'Extraction (Python)
C'est l'étape la plus dure.
* Documentation officielle de l'API : [Lire la doc (Swagger)](https://www.emploi-store-dev.fr/portail-developpeur/detail-api/offres-d-emploi-v2)
* Indice : Cherchez "Python requests OAuth2 client credentials example" sur Google.

### Étape 3 : Stockage (BigQuery)
Envoyez le JSON brut dans une table `raw_jobs`.
* *Conseil :* Ajoutez une colonne `ingestion_date` pour pouvoir suivre l'évolution du marché jour après jour.

### Étape 4 : Le Nettoyage (dbt)
Créez vos modèles `staging`.
* **Challenge SQL :** Le champ `competences` est souvent une liste imbriquée (Array) dans le JSON. Vous allez devoir utiliser la fonction `UNNEST()` de BigQuery pour "aplatir" cette liste et compter les compétences.
* **Challenge SQL :** Utilisez `REGEXP_EXTRACT` pour trouver les salaires.

### Étape 5 : La Viz
Faites parler les données.
* Un graphique barre : Top 10 des compétences demandées.
* Une carte : Densité des offres.

---

## 🏆 Definition of Done

Le projet est validé si :
1.  Vous pouvez lancer le script Python et il récupère **toutes** les offres (pas juste 150) contenant le mot clé "Data".
2.  Vous avez une table propre dans BigQuery avec une colonne `salaire_moyen_estime` (type FLOAT/INT) et non du texte.
3.  Vous avez un dashboard qui montre quel langage (Python, R, SQL, Java) paye le mieux.

Bonne chance. Google est votre meilleur ami.