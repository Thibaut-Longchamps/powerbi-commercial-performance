# 📊 Projet de pilotage commercial & Data Quality - Power BI

Projet BI réalisé avec **Power BI, PostgreSQL, Power Query (M), DAX et SQL** pour construire un reporting de pilotage commercial, suivre les objectifs et intégrer des contrôles de qualité des données.

Le projet couvre toute la chaîne, depuis les fichiers sources jusqu'au dashboard : chargement dans PostgreSQL, préparation dans Power Query, modélisation, mesures DAX, visualisation, Data Quality, RLS et analyse des performances.

---

## 🎯 Objectifs du projet

- suivre le **CA net**, la marge et le taux de marge ;
- comparer les résultats aux **objectifs commerciaux** ;
- analyser les performances par **commercial, région, catégorie et client** ;
- suivre l'évolution mensuelle du CA et la comparaison avec N-1 ;
- accéder au détail d'un commercial avec un **drill-through** ;
- détecter les clients, produits et commerciaux inconnus ;
- mesurer le **taux de conformité des données** ;
- proposer des interactions avec slicers, field parameter et tooltip ;
- appliquer un **Row-Level Security (RLS) dynamique** ;
- contrôler les temps d'exécution avec le **Performance Analyzer**.

---

## 🎬 Démonstrations

### Dashboard Power BI

![Démo du projet Power BI](gif/Demo_Projet_Pilotage_Commercial_Data_Quality.gif)

Le GIF montre notamment :

- l'utilisation des slicers ;
- le changement dynamique du KPI affiché ;
- la mise à jour des graphiques selon le contexte de filtre ;
- le tooltip commercial ;
- le drill-through vers le détail d'un commercial.

### Row-Level Security (RLS)

![Démo RLS](gif/Demo_RLS.gif)

Cette démonstration montre le comportement du rapport avec différents utilisateurs simulés dans Power BI Desktop via **View as / Afficher comme**.

---

# 📈 Dashboard

## 1. Vue Direction

![Vue Direction](screenshot/01_vue_direction.jpg)

Vue synthétique pour suivre l'activité commerciale.

### KPI

- CA Net ;
- Marge ;
- Taux de marge ;
- Nombre de commandes ;
- Évolution du CA Net vs N-1 ;
- Écart CA Net vs objectif.

### Analyses

- performance par région ;
- performance par catégorie ;
- Top 10 clients ;
- évolution mensuelle ;
- sélection dynamique de l'indicateur affiché.

Le **field parameter** permet de basculer entre :

`CA Net` • `Marge` • `Nb Commandes` • `Panier Moyen`

---

## 2. Performance commerciale

![Performance commerciale](screenshot/02_vue_performance_commerciale.jpg)

Cette page compare les résultats des commerciaux à leurs objectifs.

Indicateurs suivis :

- CA Net ;
- Objectif CA ;
- Taux d'atteinte ;
- Écart vs objectif ;
- Panier moyen ;
- Taux de marge ;
- Nombre de commandes.

La vue permet également de comparer les commerciaux dans un tableau détaillé et d'accéder à leur page individuelle par drill-through.

---

## 3. Détail commercial - Drill-through

![Détail commercial](screenshot/03_vue_detail_commercial_drill_through.jpg)

Cette page reçoit le contexte du commercial sélectionné depuis la page Performance commerciale.

Elle présente notamment :

- CA réalisé ;
- objectif ;
- écart à l'objectif ;
- taux d'atteinte ;
- marge ;
- panier moyen ;
- évolution mensuelle ;
- analyse du CA par client.

La page est masquée dans la navigation principale du rapport et utilisée comme page de drill-through.

---

## 4. Conformité & qualité des données

![Qualité et conformité](screenshot/04_vue_qualite_conformite.jpg)

Cette page centralise les contrôles de Data Quality du projet.

Contrôles réalisés :

- clients inconnus ;
- produits inconnus ;
- commerciaux inconnus ;
- transactions présentant au moins une anomalie ;
- taux global de conformité.

Des tables de détail permettent d'identifier les transactions et références concernées.

---

## 5. Tooltip commercial

![Tooltip commercial](screenshot/05_commericla_tooltip.jpg)

Un tooltip dédié complète les graphiques de performance commerciale avec :

- CA Net ;
- objectif ;
- écart vs objectif ;
- taux d'atteinte ;
- nombre de commandes.

---

# 🗄️ Sources et PostgreSQL

Les sources initiales sont fournies dans les dossiers `excel/` et `csv/`.

Elles sont chargées dans une base PostgreSQL locale :

```text
Base : powerbi_commercial
Schéma : raw
```

![PostgreSQL](screenshot/07_postgres.jpg)

Tables du schéma `raw` :

```text
raw.clients
raw.commerciaux
raw.objectifs
raw.produits
raw.ventes_brutes
```

Chaîne d'alimentation :

```text
Excel / CSV
    |
    v
PostgreSQL - schéma raw
    |
    v
Power Query / M
    |
    v
Modèle Power BI
    |
    v
DAX
    |
    v
Dashboard + Data Quality + RLS
```

La couche `raw` conserve les données sources avant les transformations réalisées dans Power Query.

---

# 🔄 Power Query / Langage M

![Power Query FactVentes](screenshot/09_factventes_power_query.jpg)

Power Query est utilisé pour préparer les données avant leur chargement dans le modèle.

Principaux traitements :

- connexion à PostgreSQL ;
- nettoyage des espaces ;
- standardisation des champs texte ;
- conversion des dates ;
- conversion des valeurs numériques ;
- normalisation des remises ;
- traitement des valeurs nulles ;
- création de contrôles de qualité ;
- suppression des doublons sur les transactions ;
- préparation des dimensions et des tables de faits.

Exemple de connexion PostgreSQL en M :

```powerquery
Source =
    PostgreSQL.Database(
        "localhost:5432",
        "powerbi_commercial"
    )
```

Navigation vers la table des ventes :

```powerquery
Source{[Schema="raw", Item="ventes_brutes"]}[Data]
```

---

# 🧩 Modèle de données

![Modèle Power BI](screenshot/08_modele_powerbi.jpg)

Le cœur du modèle suit une logique de **schéma en étoile**, complétée par plusieurs tables techniques déconnectées utilisées pour les contrôles, les interactions et la sécurité.

## Modèle analytique principal

### Dimensions

- `DimClients`
- `DimProduit`
- `DimCommerciaux`
- `DimDate`

### Tables de faits

- `FactVentes`
- `FactObjectifs`

Relations principales :

```text
DimClients       1 -> * FactVentes
DimProduit       1 -> * FactVentes
DimCommerciaux   1 -> * FactVentes
DimDate          1 -> * FactVentes

DimCommerciaux   1 -> * FactObjectifs
DimDate          1 -> * FactObjectifs
```

`DimDate` est générée dans Power Query.

## Tables techniques et de contrôle

Les tables placées à gauche du modèle ne sont pas oubliées : elles répondent à des usages spécifiques et ne font pas partie du cœur du schéma en étoile.

- `SecurityUsers` : table de mapping utilisée par le RLS dynamique avec l'email, le profil et le `Commercial_ID` ;
- `Paramètre` : table générée par le field parameter pour choisir dynamiquement le KPI affiché ;
- `Mesures` : table dédiée au rangement des mesures DAX ;
- `CTRL_Clients_Inconnus` : contrôle des références clients absentes de `DimClients` ;
- `CTRL_Produits_Inconnus` : contrôle des références produits absentes de `DimProduit` ;
- `CTRL_Commerciaux_Inconnus` : contrôle des références commerciales absentes de `DimCommerciaux` ;
- `CTRL_Anomalies_Detail` : consolidation des anomalies au niveau transaction ;
- `Types Anomalies` : liste utilisée pour organiser et filtrer les types d'anomalies.

Des requêtes de détail complémentaires permettent également d'afficher les lignes à l'origine des anomalies dans la page Data Quality.

Cette séparation garde le modèle métier lisible tout en isolant les tables utilisées pour la sécurité, les paramètres et les contrôles de qualité.

---

# 🧮 DAX

Les indicateurs du dashboard sont construits avec des mesures DAX.

Exemples :

- CA brut ;
- CA net ;
- marge ;
- taux de marge ;
- panier moyen ;
- nombre de commandes ;
- objectif CA ;
- écart vs objectif ;
- taux d'atteinte ;
- évolution du CA vs N-1.

DAX est aussi utilisé pour :

- les titres dynamiques ;
- les couleurs conditionnelles ;
- les contextes de filtres ;
- les indicateurs de Data Quality ;
- le RLS dynamique avec `USERPRINCIPALNAME()`.

---

# 🔐 Row-Level Security (RLS)

Le rapport intègre un **RLS dynamique** basé sur une table de sécurité dédiée.

Principe de la démonstration :

- un commercial ne voit que les données associées à son `Commercial_ID` ;
- un manager voit l'ensemble des données ;
- l'utilisateur est simulé dans Power BI Desktop avec **View as / Afficher comme** ;
- `USERPRINCIPALNAME()` permet de récupérer l'identifiant de l'utilisateur dans la règle RLS.

La table `SecurityUsers` reste déconnectée du modèle métier et sert de table de mapping pour déterminer le profil et le `Commercial_ID` correspondant.

![Démo RLS](gif/Demo_RLS.gif)

---

# ⚙️ Fonctionnalités Power BI utilisées

- modèle principal en étoile ;
- PostgreSQL comme source de données ;
- Power Query / langage M ;
- mesures DAX ;
- field parameter ;
- titres dynamiques ;
- slicers ;
- tooltip personnalisé ;
- drill-through ;
- mise en forme conditionnelle ;
- contrôles de Data Quality ;
- RLS dynamique ;
- Performance Analyzer.

---

# 🚀 Performance Analyzer

![Performance Analyzer](screenshot/06_analyseur_performance.jpg)

Le **Performance Analyzer** a été utilisé pour mesurer le temps d'exécution des principaux visuels.

Sur les tests réalisés, les requêtes **DAX sont très rapides**, généralement autour de **100 à 200 ms**. Elles ne constituent donc pas le principal point de ralentissement observé.

La plus grande partie du temps total apparaît dans les catégories **Visual display** et surtout **Other**. `Other` regroupe plusieurs opérations internes de Power BI qui ne correspondent pas directement à l'exécution DAX : attente entre visuels, ordonnancement des requêtes, traitements internes et disponibilité des ressources.

Dans ce projet exécuté localement, la durée élevée de `Other` semble donc davantage liée au rendu du rapport et aux **ressources disponibles sur le PC** qu'à la complexité des mesures DAX elles-mêmes. Cette interprétation reste dépendante de l'environnement d'exécution.

Exemples observés lors des tests :

```text
DAX Query        : environ 100 à 200 ms
Visual display   : plusieurs centaines de ms
Other            : part majoritaire du temps total sur plusieurs visuels
```

Le contrôle avec Performance Analyzer permet ainsi de vérifier que les mesures DAX restent rapides et d'orienter l'optimisation vers le rendu, le nombre de visuels et l'environnement d'exécution plutôt que vers une réécriture inutile des mesures.

---

# 🛠️ Stack technique

| Domaine | Technologies |
|---|---|
| Business Intelligence | Power BI Desktop |
| Base de données | PostgreSQL |
| Administration BDD | pgAdmin |
| Transformation | Power Query / M |
| Mesures et logique analytique | DAX |
| Requêtes | SQL |
| Sources initiales | Excel / CSV |
| Versioning / Portfolio | Git / GitHub |

---

# 📁 Structure du dépôt

```text
Power_BI/
|
├── README.md
├── Projet_Pilotage_Commercial_Data_Quality.pbix
|
├── csv/
|   ├── 01_Clients.csv
|   ├── 02_Produits.csv
|   ├── 03_Commerciaux.csv
|   ├── 04_Ventes_Brutes.csv
|   └── 05_Objectifs_Commerciaux.csv
|
├── excel/
|   ├── 01_Clients.xlsx
|   ├── 02_Produits.xlsx
|   ├── 03_Commerciaux.xlsx
|   ├── 04_Ventes_Brutes.xlsx
|   └── 05_Objectifs_Commerciaux.xlsx
|
├── gif/
|   ├── Demo_Projet_Pilotage_Commercial_Data_Quality.gif
|   └── Demo_RLS.gif
|
└── screenshot/
    ├── 01_vue_direction.jpg
    ├── 02_vue_performance_commerciale.jpg
    ├── 03_vue_detail_commercial_drill_through.jpg
    ├── 04_vue_qualite_conformite.jpg
    ├── 05_commericla_tooltip.jpg
    ├── 06_analyseur_performance.jpg
    ├── 07_postgres.jpg
    ├── 08_modele_powerbi.jpg
    └── 09_factventes_power_query.jpg
```

---

# 🔍 Chaîne de traitement

```text
Sources Excel / CSV
      |
      v
PostgreSQL / schéma raw
      |
      v
Nettoyage et transformations Power Query
      |
      v
Dimensions + tables de faits
      |
      v
Modèle principal en étoile
      |
      v
Mesures DAX
      |
      v
Data Quality + RLS
      |
      v
Reporting Power BI
      |
      v
Performance Analyzer
```

---

# 💡 Compétences mises en œuvre

### Data ingestion
- intégration de plusieurs sources ;
- chargement dans PostgreSQL ;
- requêtes SQL.

### Data preparation
- nettoyage ;
- standardisation ;
- typage ;
- gestion des valeurs nulles ;
- normalisation des remises ;
- suppression des doublons ;
- contrôles de qualité.

### Data modeling
- dimensions ;
- tables de faits ;
- relations `1 -> *` ;
- schéma en étoile ;
- tables techniques déconnectées.

### Analytics
- KPI ;
- comparaison réalisé / objectif ;
- analyse temporelle ;
- segmentation commerciale ;
- analyse N vs N-1.

### Data visualization
- dashboard interactif ;
- field parameter ;
- slicers ;
- drill-through ;
- tooltip personnalisé ;
- titres dynamiques.

### Security & performance
- RLS dynamique ;
- `USERPRINCIPALNAME()` ;
- simulation d'utilisateurs ;
- Performance Analyzer ;
- analyse des temps DAX et du rendu.

---

# 🔜 Évolutions possibles

- actualisation planifiée dans Power BI Service ;
- configuration d'une gateway pour la base PostgreSQL locale ;
- ajout de vues SQL dédiées au reporting ;
- tests sur des volumes de données plus importants ;
- automatisation du pipeline d'alimentation ;
- déploiement et gestion des accès dans un workspace Power BI.

---

## 👤 Auteur

**Thibaut Longchamps**

Data Analyst / Data & AI

GitHub : [Thibaut-Longchamps](https://github.com/Thibaut-Longchamps)
