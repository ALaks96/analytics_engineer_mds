# 🏗️ NG++ : The Scale-Up Challenge (E-commerce & Performance)

🚨 **Niveau : Expert.**

Vous savez faire des pipelines qui marchent. Maintenant, on va faire des pipelines qui **tiennent la charge**.
Dans ce projet, vous n'êtes plus un stagiaire. Vous êtes le Lead Analytics Engineer.

**Le scénario :**
Votre site E-commerce "TheLook" explose.
Votre Data Warehouse commence à ramer.
Votre CFO gueule parce que la facture BigQuery a doublé.
Votre mission : Refondre les modèles pour gérer l'**Incrémentalité** et répondre à des questions business complexes.

---

## 💾 La Source de Données

Pas besoin d'ingestion ici, la donnée est déjà dans BigQuery (Public Datasets).
* **Projet :** `bigquery-public-data`
* **Dataset :** `thelook_ecommerce`
* **Tables clés :** `orders`, `order_items`, `products`, `users`, `events`.

---

## 🧱 Partie 1 : Complex Modeling (L'OBT)

Le marketing ne veut pas faire 4 jointures à chaque fois qu'ils veulent un chiffre. Ils veulent une **One Big Table (OBT)**.

### 🎯 Mission Business
Créer un modèle **Facts** (`fct_orders_details`) ultra-complet qui répond à ces questions sans refaire de jointures :
1.  Quel est le panier moyen par catégorie de produit ?
2.  Quelle est la marge brute par commande ? (`prix de vente - coût de production`).
3.  Combien de temps s'écoule entre la création du compte user et sa première commande ?

### 🛠️ Le Défi Technique
Vous devez joindre : `orders` + `order_items` + `products` + `users`.
* *Attention :* Une commande peut avoir plusieurs items. La granularité de votre table finale doit être `order_item_id` (une ligne par article acheté), mais enrichie avec les infos de la commande et de l'utilisateur.

---

## ⚡ Partie 2 : L'Incrémentalité (Le cœur du sujet)

C'est là que ça se corse. La table `events` (clics sur le site) contient des millions de lignes.
Si vous faites un `DROP TABLE` + `CREATE TABLE` (Full Refresh) tous les matins, ça prend 1 heure.

### 🧠 Le Concept : Incremental Models
Au lieu de tout recalculer, on ne traite que **ce qui s'est passé depuis la dernière fois**.
* **Jour J :** On charge tout l'historique.
* **Jour J+1 :** On charge uniquement les événements d'hier et on les "colle" (Append) à la suite.



### 🎯 Mission : Setup Incremental
1.  Configurez votre modèle dbt `fct_events` en `materialized='incremental'`.
2.  Utilisez la logique Jinja `{% if is_incremental() %}` pour filtrer uniquement les nouvelles lignes.
    * *Condition :* Charger les lignes où `created_at` est supérieur à la date max déjà présente dans la table.

### 🧪 Le Test de vérité (Simulation)
Comme la donnée publique est statique, vous allez simuler l'avancée du temps.
1.  Lancez dbt en filtrant les données avant 2023 : `dbt run --vars 'max_date: 2023-01-01'` (il faudra coder cette variable dans le WHERE de votre SQL).
2.  Vérifiez le nombre de lignes.
3.  Relancez dbt avec `2023-02-01`.
4.  Vérifiez que seules les lignes de janvier ont été ajoutées (regardez les logs BigQuery "Bytes processed", ça doit être minuscule).

---

## 🔄 Partie 3 : Gestion des Updates (Merge & Unique Key)

L'incrémental "Append-only" (ajout simple), c'est facile. Mais dans le E-commerce, les commandes changent de statut !
Une commande passe de `Processing` à `Shipped` puis potentiellement à `Returned`.

Si vous faites juste "Append", vous aurez deux lignes pour la même commande (une en processing, une en shipped). C'est faux.
Il faut faire un **Merge** (Mise à jour).

### 🎯 Mission : Handle Returns
1.  Modifiez votre modèle `fct_orders`.
2.  Configurez la `unique_key` dans dbt.
3.  Le but : Si l'ID de commande existe déjà, on met à jour la ligne (pour changer le statut). Si elle n'existe pas, on l'ajoute.

### 🧠 Le Concept : Merge vs Append
* **Append :** J'ajoute les nouvelles lignes au fond du fichier. Rapide, mais ne gère pas les modifications du passé.
* **Merge :** Je compare les nouvelles lignes avec les anciennes. Si l'ID matche, j'écrase (UPDATE). Sinon, j'ajoute (INSERT). C'est plus lourd, mais c'est exact.

---

## 🕵️ Partie 4 : Le Snapshotting (SCD Type 2)

Votre boss vous demande : *"Est-ce que les utilisateurs qui habitent à Paris achètent plus cher ?"*
Facile, vous regardez l'adresse.
MAIS, si un utilisateur déménage de Paris à Lyon, son adresse change dans la base.
Si vous analysez ses commandes de l'an dernier (quand il était à Paris) avec son adresse d'aujourd'hui (Lyon), votre analyse est fausse.

Il faut historiser les changements d'adresse.

### 🧠 Le Concept : Slowly Changing Dimensions (SCD)
* **Type 1 :** On écrase l'ancienne valeur (On perd l'historique).
* **Type 2 :** On crée une nouvelle ligne avec une date de début et de fin (`valid_from`, `valid_to`).

### 🎯 Mission
1.  Créer un "Snapshot" dbt sur la table `users`.
2.  Stratégie : `timestamp` (basé sur la colonne de mise à jour) ou `check` (si vous voulez surveiller une colonne spécifique comme `city`).
3.  Lancez le snapshot (`dbt snapshot`).
4.  Observez comment dbt crée automatiquement les colonnes `dbt_valid_from` et `dbt_valid_to`.

---

## 🏆 Boss Final : La Question Piège

Si vous avez réussi tout ça, répondez à cette question business en SQL en utilisant vos modèles :

> "Calculez le taux de retour (Refund Rate) par Cohorte mensuelle d'inscription.
> Est-ce que les utilisateurs inscrits en Janvier retournent plus leurs colis que ceux inscrits en Juin ?"

*Indices pour réussir :*
* Il faut prendre la date de création de l'user (Cohort).
* Il faut regarder le statut de ses commandes (Returned).
* Attention : Un user inscrit en Janvier peut commander en Mars. Le retour compte pour la cohorte Janvier.

Bon courage. Si vous savez gérer l'incrémentalité et les snapshots, vous êtes techniquement au-dessus de 80% des juniors.