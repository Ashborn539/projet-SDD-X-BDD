# Impact de la pollution atmosphérique sur la mortalité respiratoire

Projet universitaire regroupant les UE Bases de données (TE43MI) et Sciences des données 2 (TE44MI) à l'Université Paul-Valéry Montpellier 3 (promotion 2026, groupe TD2, équipe Efficiency).

## Problématique

L'exposition chronique aux particules fines (PM2.5) augmente les risques d'affections respiratoires et cardiovasculaires. Ce travail croise des relevés environnementaux urbains et des données sanitaires nationales pour répondre à une question :

**Existe-t-il une corrélation mesurable entre la pollution de l'air en ville (indices AQI et PM2.5) et les taux de mortalité respiratoire observés à l'échelle des pays ?**

## Équipe

- Tylden Hounsa (22412555) : coordination du rapport et requêtes SQL.
- Noha Oussaid El Amri (22413984) : collecte et nettoyage des données brutes.
- Mary-Yohana Sylla (22414031) : modélisation relationnelle (MCD et MOD).
- Aissatou Tall (22414034) : visualisations et analyse exploratoire.

## Données et modélisation

L'étude s'appuie sur deux sources ouvertes :

- Données de pollution : Global Air Pollution Dataset (Kaggle, issu d'OpenAQ), avec 23 464 villes recensées.
- Données de santé : Air Pollution Death Rate (OMS, année 2016), couvrant 183 pays.

Les fichiers ont été nettoyés avec LibreOffice Calc puis importés dans MySQL (`pollution_health_db`) sous trois tables reliées par clés étrangères :

- `PAYS` (192 pays répertoriés) : `id_pays`, `nom_pays`.
- `VILLE` (23 035 enregistrements urbains) : `id_ville`, `nom_ville`, `aqi_value`, `pm25_value`, `aqi_category`, `id_pays`.
- `SANTE` (6 588 lignes de statistiques OMS) : `id_stat`, `indicator`, `maladie`, `valeur_taux`, `periode`, `sexe`, `id_pays`.

Le script SQL complet de création et d'insertion est disponible dans `Data/pollution_health_db.sql`.

## Résultats statistiques

L'analyse montre que le choix de l'indicateur de mortalité conditionne entièrement les conclusions :

1. Taux bruts de mortalité : le test de Pearson donne r = 0,035 (p = 0,65). Aucun lien linéaire n'apparaît. La structure par âge masque la relation, car les pays à l'air pur comptent des populations plus âgées, donc un taux brut naturellement élevé.
2. Taux standardisés par l'âge (`attributable_age_std`) : la correction du biais démographique fait ressortir un lien significatif avec r = 0,18 (p = 0,02).
3. Exclusion de la valeur aberrante : un pays présente un PM2.5 moyen supérieur à 400 µg/m³ (erreur dans les données sources). En filtrant `pm25_moyen < 200`, la corrélation monte à r = 0,28 (p < 0,001, IC 95 % : [0,14 ; 0,42]).
4. Régression linéaire : chaque hausse de 1 µg/m³ de PM2.5 moyen est associée à 0,56 décès supplémentaire pour 100 000 habitants (R² = 0,08, p < 0,001).
5. ANOVA sur les catégories d'AQI : l'écart de mortalité selon la tranche de pollution est hautement significatif (p = 1,3e-11).
6. K-means (k = 3) : la classification sépare trois groupes de pays (risque faible, modéré, et élevé).

## Organisation des fichiers

```text
.
├── Data/
│   └── pollution_health_db.sql            # Script SQL de la base de données
├── images/
│   ├── MCD.png, MOD.png                   # Schémas relationnels
│   └── *.pdf                              # Graphiques générés pour le document
├── diapositives-GEfficiency.pdf           # Diaporama de soutenance
├── rapport-GEfficiency.Rmd                # Code source du rapport en R Markdown
├── rapport-GEfficiency.pdf                # Rapport final compilé
├── references.bib                         # Références bibliographiques BibTeX
├── template.tex, header.tex               # Modèles de mise en page LaTeX
├── unswthesis.cls                         # Classe LaTeX
├── elsarticle-harv.bst                    # Style bibliographique
├── iso690-author-date-fr-no-abstract.csl  # Style de citation CSL
└── README.md
```

## Reproduction des analyses

### Base de données

Importer le dump dans une instance MySQL :

```bash
mysql -u root -p < Data/pollution_health_db.sql
```

Par défaut, le document Rmd se connecte en local sur le port 3306 avec l'utilisateur `root` sans mot de passe.

### Dépendances R

Installer les packages requis dans R :

```r
install.packages(c("DBI", "RMySQL", "ggplot2", "dplyr", "knitr"))
```

### Compilation

Le rapport peut être compilé depuis RStudio via le bouton Knit, ou en ligne de commande :

```r
rmarkdown::render("rapport-GEfficiency.Rmd", output_format = "pdf_document")
```
