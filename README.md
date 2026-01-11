# 🚴‍♂️ Projet : The Paris Vélib' Monitor

Bienvenue. Si vous lisez ceci, c'est que vous voulez passer du côté technique de la Data ("Analytics Engineering").

Ce projet est conçu comme un jeu vidéo en 2 Phases :
1.  **Phase 1 (MVP) :** On fait tout "à la main" pour comprendre la logique métier (SQL, KPI, Dashboard). Objectif : Avoir un dashboard qui marche en 2 jours.
2.  **Phase 2 (Industrialisation) :** On automatise tout avec du code (Python, API, Docker) pour que ça tourne tout seul. C'est le vrai travail d'ingénieur.

---

## 🛠 Pré-requis & Outils

Avant de commencer, installez ces bases.
* **VS Code** (Votre atelier de code) : [Télécharger ici](https://code.visualstudio.com/)
* **Git** (Votre sauvegarde) : [Télécharger ici](https://git-scm.com/downloads)
* **Un compte Google** (Pour le Cloud).

---

# 🏁 Phase 1 : Le MVP (Minimum Viable Product)

**Objectif :** Analyser la donnée statique. On s'en fiche de l'automatisation pour l'instant, on veut voir des chiffres.

## Étape 1 : Le Cloud (BigQuery)
Vous n'allez pas stocker les données sur votre ordi (Excel), mais dans un "Data Warehouse" dans le Cloud. On utilise **Google BigQuery** car ils ont une "Sandbox" gratuite (pas de carte bleue requise).

### 🧠 Le Concept : Data Warehouse
C'est un entrepôt géant capable de traiter des milliards de lignes en secondes. Contrairement à Excel, on ne "voit" pas la donnée, on lui pose des questions via du code (SQL).

### 🎯 Mission
1.  Créer un projet sur Google Cloud Platform (GCP).
2.  Activer l'API BigQuery.
3.  Créer un "Dataset" (un dossier) appelé `raw_velib`.
4.  Télécharger ce fichier JSON sur votre ordi (clic droit > enregistrer sous) : [Data Vélib Temps Réel](https://velib-metropole-opendata.smoove.pro/opendata/Velib_Metropole/station_status.json)
5.  Uploader ce fichier manuellement dans le dataset `raw_velib` pour créer une table `stations`.

### 🔗 Liens utiles
* **Tuto Indispensable :** [Comment activer la BigQuery Sandbox (Gratuit)](https://cloud.google.com/bigquery/docs/sandbox?hl=fr)
* **Tuto :** [Charger un fichier local dans BigQuery](https://cloud.google.com/bigquery/docs/loading-data-local?hl=fr) (Regardez juste la partie "Console Cloud").

---

## Étape 2 : La Transformation (dbt & SQL)
C'est le cœur du métier d'Analytics Engineer. Votre donnée brute est moche (JSON imbriqué, types bizarres). On va utiliser **dbt** (data build tool) pour la nettoyer via du SQL.

### 🧠 Le Concept : dbt
Avant dbt, on écrivait des scripts SQL bordéliques qu'on lançait à la main. dbt permet de structurer le SQL comme du code informatique (versionné, testé, documenté).

### 🎯 Mission
1.  Installer Python et dbt sur votre ordinateur.
2.  Connecter dbt à votre BigQuery.
3.  Écrire un modèle SQL pour nettoyer la table brute.

### 🔗 Liens utiles
* [Tuto dbt pour BigQuery (Suivez les étapes "Installation" et "Connect")](https://docs.getdbt.com/guides/bigquery?step=1)

### 🆘 Cheat Codes

<details>
<summary>👀 <strong>Cheat 1 : L'installation (Terminal)</strong></summary>

Ouvrez votre terminal (dans VS Code) et tapez :
```bash
# Installe dbt pour BigQuery
pip install dbt-bigquery

# Initialise le projet (répondez aux questions)
dbt init velib_project
```
Si la commande pip n'est pas trouvée, vérifiez que vous avez coché "Add Python to PATH" lors de l'installation de Python.

</details>

<details> <summary>👀 <strong>Cheat 2 : La configuration (profiles.yml)</strong></summary>

C'est souvent là que ça bloque. Pour vous connecter à BigQuery sans prise de tête au début, utilisez la méthode OAuth (authentification via le navigateur) lors du dbt init.

</details>

<details> <summary>👀 <strong>Cheat 3 : Le Code SQL (Le modèle)</strong></summary>

Créez un fichier models/staging/stg_stations.sql :

```sql
SELECT
    stationCode as station_id,
    num_bikes_available as nb_velos,
    is_renting = 'OUI' as est_ouverte,
    capacity
FROM `votre-projet-gcp.raw_velib.stations`
```
Puis lancez la commande dbt run dans le terminal.

</details>

## Étape 3 : La Dataviz
La donnée est propre. Montrez-la.

### 🎯 Mission
Ouvrir Google Looker Studio.

Connecter la source de données "BigQuery" -> Votre table créée par dbt.

Faire un graph : "Top 10 des stations avec le plus de capacité".

# 🚀 Phase 2 : L'Industrialisation (Software Engineering)
Vous avez validé la logique. Maintenant, on arrête de charger les fichiers à la main. On veut du temps réel.

## Étape 4 : Python & API
On va remplacer votre clic manuel "Uploader un fichier" par un robot.

### 🧠 Le Concept : API & JSON
Une API est une prise électrique sur le web qui donne de la donnée. Le JSON est le format de cette donnée.

### 🎯 Mission
Créer un script extract.py qui télécharge la donnée Vélib et l'affiche.

### 🆘 Cheat Codes
<details> <summary>👀 <strong>Cheat : Le Script de base</strong></summary>

```python
import requests
import json

# L'URL magique
url = "https://velib-metropole-opendata.smoove.pro/opendata/Velib_Metropole/station_status.json"

# On appelle le serveur
reponse = requests.get(url)

# On lit le contenu
data = reponse.json()

# Affichez pour comprendre la structure
print(data) 
```
</details>

## Étape 5 : Docker & Airbyte (Le niveau Pro)
Vous ne lancerez pas le script Python depuis votre ordi tous les jours. On utilise un outil d'ingestion : Airbyte.

### 🧠 Le Concept : Conteneurs
Docker permet d'installer Airbyte sans polluer votre Mac/PC. C'est une "boîte" étanche.

### 🎯 Mission
Installer Docker Desktop.

Installer Airbyte en local.

Connecter l'API Vélib (Source) à BigQuery (Destination) dans Airbyte.

### 🔗 Liens utiles
Installer Docker

Deployer Airbyte Localement (Le tuto officiel)

### 🆘 Cheat Codes
<details> <summary>👀 <strong>Cheat : Configurer Airbyte</strong></summary>

Dans Airbyte (localhost:8000), créez une Source "File" (Fichier).

URL : L'url du JSON Vélib.

Format : JSON.

Destination : BigQuery (Il faudra créer un "Service Account" sur Google Cloud pour donner la permission à Airbyte d'écrire. C'est l'étape la plus dure du projet, googlez "Create Service Account BigQuery".

</details>

### 🎉 Le Final Boss
Si vous avez réussi à :

Avoir Airbyte qui tourne et envoie la donnée tous les jours.

Avoir dbt qui nettoie cette donnée.

Avoir Looker Studio qui affiche la donnée à jour.