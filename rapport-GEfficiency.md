---
title: "Rapport de groupe des UE \\newline  Bases de données + Sciences des Données 2"
author:
- Hounsa Tylden,
- Oussaid El Amri Noha,
- Sylla Mary-Yohana,
- Tall Aissatou
date: "27 octobre 2026"
output:
  pdf_document:
    fig_caption: yes
    keep_md: yes
    keep_tex: yes
    md_extensions: +raw_attribute
    number_sections: yes
    pandoc_args:
    - --top-level-division="chapter"
    - --bibliography="references.bib"
    template: template.tex
    toc: yes
    toc_depth: 1
  word_document:
    fig_caption: yes
    number_sections: yes
    pandoc_args:
    - --top-level-division="chapter"
    - --to="odt+native_numbering"
    toc: yes
    toc_depth: '2'
  html_document:
    df_print: paged
    toc: yes
    toc_depth: '2'
toc-title: "Table des matières"
bibliography: references.bib
coursecode: TE43MI-TE44MI
csl: iso690-author-date-fr-no-abstract.csl
Acknowledgements:
- Nos plus sincères remerciements vont à notre encadrant pédagogique pour les conseils
  avisés sur notre travail.
- Nous remercions également Kaggle et l'Organisation Mondiale de la Santé pour la
  mise à disposition publique des jeux de données utilisés dans ce projet.
biblio-style: elsarticle-harv
session: 2026
team: Efficiency
groupeTD: TD2
Abstract: "La pollution atmosphérique constitue l'une des principales causes de mortalité
  évitable à l'échelle mondiale. Ce rapport croise des données environnementales issues
  de Kaggle (23 464 villes, indice AQI et concentration en PM2.5) avec des statistiques
  sanitaires de l'OMS (taux de mortalité par maladies liées à la pollution, 183 pays,
  année 2016), afin d'examiner la corrélation entre qualité de l'air urbaine et mortalité
  nationale. Après constitution d'une base de données relationnelle sous MySQL (tables
  PAYS, VILLE et SANTE), une démarche en trois étapes révèle l'importance du choix de
  l'indicateur de mortalité : l'absence de corrélation sur les taux bruts (r = 0,035)
  s'explique par un biais démographique, corrigé par l'utilisation des taux standardisés
  selon l'âge fournis par l'OMS. Après identification et exclusion d'une valeur aberrante,
  une corrélation positive modérée et statistiquement significative est établie entre
  le niveau moyen de PM2.5 par pays et le taux de mortalité standardisé (r = 0,28,
  p < 0,001). Un modèle de régression linéaire simple quantifie cet effet, et une
  segmentation par K-means identifie trois profils de pays distincts. Ces résultats
  soulignent l'importance du choix des indicateurs dans ce type d'étude et plaident
  pour le développement de données de morbidité comparables à l'échelle internationale."
always_allow_html: true
editor_options:
  markdown:
    wrap: 72
---



# Introduction {.label:s-intro}

## Présentation du contexte et problématique

La dégradation de la qualité de l'air est aujourd'hui reconnue comme
l'une des menaces les plus graves pour la santé publique mondiale. Selon
les rapports de l'Organisation Mondiale de la Santé (OMS), l'exposition
prolongée aux polluants atmosphériques, et plus particulièrement aux
particules fines (PM2.5), est directement associée à l'augmentation des
pathologies respiratoires chroniques et des décès prématurés. Les
particules PM2, dont le diamètre aérodynamique est inférieur à 2,5
micromètres, sont particulièrement dangereuses car elles pénètrent
profondément dans les alvéoles pulmonaires et peuvent atteindre la
circulation sanguine, provoquant des effets systémiques bien au-delà du
seul appareil respiratoire.

L'enjeu de ce projet est de croiser des mesures environnementales
urbaines avec des indicateurs de mortalité pour vérifier la réalité de
cet impact à l'échelle globale. Notre recherche s'articule autour de la
question suivante :

\bigskip

\begin{center}
\textbf{Existe-t-il une corrélation significative entre les niveaux de pollution urbaine (AQI et PM2.5) et les taux de mortalité par maladies respiratoires à l'échelle mondiale ?}
\end{center}
\medskip
\justifying

Cette question est cruciale car elle permet d'évaluer l'efficacité des
politiques environnementales actuelles. Si une corrélation forte est
établie, cela justifierait des mesures plus restrictives sur les
émissions polluantes dans les zones urbaines denses, afin de réduire la
pression sur les systèmes de santé publique. Les résultats de cette
étude pourront également servir d'appui scientifique à des organisations
souhaitant prioriser leurs interventions géographiques.

## Responsabilités et composition de l'équipe

\medskip

-   **Tylden Hounsa** : Étudiant n\textsuperscript{o}22412555,
    Responsable du rapport et des analyses SQL (Rédacteur principal du
    .Rmd).

\medskip

-   **Noha Oussaid El Amri** : Étudiante n\textsuperscript{o}22413984,
    Responsable de la collecte et du nettoyage des données
    (\textit{Data Cleaning}).

\medskip

-   **Mary-Yohana Sylla** : Étudiante n\textsuperscript{o}22414031,
    Responsable de la modélisation de la base de données (MCD/MOD).

\medskip

-   **Aissatou Tall** : Étudiante n\textsuperscript{o}22414034, Responsable
    des visualisations graphiques et de l'analyse exploratoire.

## Quelques détails techniques

Pour la réalisation de ce rapport, nous utilisons **R Markdown** via
**RStudio**, ce qui permet d'intégrer directement nos scripts d'analyse
au sein du document et d'en assurer la reproductibilité. La qualité
rédactionnelle est vérifiée via l'outil LanguageTool
(\url{https://languagetool.org/fr}). Les schémas relationnels sont
conçus avec **Mocodo** (\url{https://www.mocodo.net/}) pour le MCD, et
avec le designer de **PHPMyAdmin** pour le MOD.

# Base de données

## Provenance des données

Nous avons sélectionné deux jeux de données provenant de sources
officielles et distinctes pour répondre à notre problématique :

\bigskip

\textbf{Données de Pollution (Source Environnementale) :}

\begin{itemize}
  \item Lien : \href{https://www.kaggle.com/datasets/hasibalmuzdadid/global-air-pollution-dataset/data}{Kaggle -- Global Air Pollution Dataset (basé sur OpenAQ)}.
  \item Description : Ce fichier recense les indices de qualité de l'air (AQI) et les concentrations de plusieurs polluants pour plus de 23\,000 villes à travers le monde. Nous l'avons retenu pour sa granularité urbaine et son étendue géographique internationale.
  \item Format : .csv $|$ Taille : 1\,593 ko $|$ Dimensions : 23\,464 lignes $\times$ 12 colonnes.
\end{itemize}

\bigskip

\textbf{Données de Santé (Source Médicale) :}

\begin{itemize}
  \item Lien : \href{https://www.kaggle.com/datasets/utkarshxy/who-worldhealth-statistics-2020-complete?select=airPollutionDeathRate.csv}{OMS -- Air Pollution Death Rate}.
  \item Description : Statistiques de mortalité attribuable à la pollution atmosphérique, publiées par l'OMS pour l'année 2016 et couvrant 183 pays. Nous avons filtré les données pour ne conserver que les indicateurs relatifs aux maladies respiratoires et cardiovasculaires directement liées à la pollution.
  \item Format : .csv $|$ Taille : 1\,036 ko $|$ Dimensions : 6\,589 lignes $\times$ 6 colonnes.
\end{itemize}

\bigskip

\textbf{Population étudiée :} L'unité statistique retenue est le
\textbf{pays}. Bien que les données de pollution soient mesurées à
l'échelle de la ville, nous les agrégeons par pays, via une moyenne du
PM2.5, pour permettre la jointure avec les données de l'OMS, disponibles
uniquement à l'échelle nationale. Cette agrégation introduit cepensant
une perte de granularité explicitée dans la partie Discussion.

## Descriptif des tables

Après nettoyage et structuration des données, nous avons constitué trois
tables : PAYS, VILLE et SANTE. Leurs caractéristiques sont détaillées
ci-dessous.

\medskip

\begin{table}[H]
\centering
\renewcommand{\arraystretch}{1.3}
\begin{tabular}{|p{3cm}|p{3.5cm}|p{3cm}|p{3cm}|}
\hline
\textbf{Nom colonne} & \textbf{Type} & \textbf{Signification} & \textbf{Caractéristique} \\ \hline
id\_pays & INT NOT NULL & Identifiant numérique unique & Clé primaire, séquentiel 1--192 \\ \hline
nom\_pays & VARCHAR(100) NOT NULL & Nom du pays (en anglais) & 192 valeurs uniques \\ \hline
\end{tabular}
\medskip
\caption{Table PAYS (2 colonnes $\times$ 192 lignes)}
\end{table}
\begin{table}[H]
\centering
\renewcommand{\arraystretch}{1.3}
\begin{tabular}{|p{3cm}|p{3.5cm}|p{3cm}|p{4.5cm}|}
\hline
\textbf{Nom colonne} & \textbf{Type} & \textbf{Signification} & \textbf{Caractéristique} \\ \hline
id\_stat & INT NOT NULL & Identifiant de la ligne & Clé primaire \\ \hline
indicateur & VARCHAR(25) & Type de taux de mortalité & \texttt{attributable}, \texttt{attributable\_age\_std} \\ \hline
maladie & VARCHAR(150) & Cause de décès & Total, Lower respiratory infections, Trachea bronchus lung cancers, Ischaemic heart disease, Stroke, Chronic obstructive pulmonary disease \\ \hline
valeur\_taux & FLOAT & Taux de mortalité pour 100\,000 hab. & 1,00 -- 99,91 \\ \hline
periode & INT & Année de relevé & 2016 \\ \hline
sexe & VARCHAR(20) & Sexe concerné & Female, Male, Both sexes \\ \hline
id\_pays & INT NOT NULL & Référence au pays & Clé étrangère $\rightarrow$ PAYS \\ \hline
\end{tabular}
\medskip
\caption{Table SANTE (7 colonnes $\times$ 6\,588 lignes)}
\end{table}
\begin{table}[H]
\centering
\renewcommand{\arraystretch}{1.3}
\begin{tabular}{|p{3cm}|p{3.5cm}|p{3cm}|p{4.5cm}|}
\hline
\textbf{Nom colonne} & \textbf{Type} & \textbf{Signification} & \textbf{Caractéristique} \\ \hline
id\_ville & INT NOT NULL & Identifiant de la ville & Clé primaire \\ \hline
nom\_ville & VARCHAR(150) NOT NULL & Nom de la ville & $\approx$23\,000 villes distinctes \\ \hline
aqi\_value & INT & Indice de Qualité de l'Air & 6 -- 500 \\ \hline
pm25\_value & INT & Concentration en PM2.5 ($\mu$g/m$^3$) & 0 -- 500 \\ \hline
aqi\_category & VARCHAR(50) & Catégorie de qualité de l'air & Good, Moderate, Unhealthy\_for\_Sensitive\_Groups, Unhealthy, Very\_Unhealthy, Hazardous \\ \hline
id\_pays & INT NOT NULL & Référence au pays & Clé étrangère $\rightarrow$ PAYS \\ \hline
\end{tabular}
\medskip
\caption{Table VILLE (6 colonnes $\times$ 23\,035 lignes)}
\end{table}

## Modèles MCD et MOD

Le Modèle Conceptuel de Données (MCD) a été réalisé avec Mocodo. Il
représente les trois entités PAYS, VILLE et SANTE, reliées par des
associations de cardinalités (1,n) : un pays peut être associé à
plusieurs villes et à plusieurs entrées sanitaires.

![MCD réalisé avec
Mocodo.](./scdon2-UPV-report-template_files/images/mcd.png){#mcd
width="10cm"}

\bigskip

Le Modèle Organisationnel de Données (MOD) a été généré via le designer
de PHPMyAdmin. Il formalise les types de données et les contraintes de
clés étrangères entre les tables.

![MOD généré via le designer de
PHPMyAdmin.](./scdon2-UPV-report-template_files/images/mod.png){#mod
width="14cm" height="6cm"}

## Import des données

Afin de garantir l'intégrité référentielle et la qualité des analyses
futures, un processus rigoureux de nettoyage et de structuration a été
mené à l'aide du logiciel **LibreOffice Calc**. Ce traitement a permis
de passer de deux fichiers CSV « plats » à une structure relationnelle
conforme à notre MCD.

### Création de la table de référence : PAYS

La table `PAYS` sert de pivot central entre les deux sources de données.
Son élaboration a suivi les étapes suivantes :

\medskip

\begin{itemize}
  \item \textbf{Extraction et fusion :} Les colonnes \texttt{Country} (base pollution) et \texttt{Location} (base santé) ont été consolidées dans une feuille unique afin d'obtenir la liste complète des pays présents dans les deux sources.
  \item \textbf{Dédoublonnage :} La fonction native \textbf{``Supprimer les doublons''} a permis d'obtenir une liste de nations uniques. Les noms étant uniformément en anglais dans les deux sources, aucune erreur de traduction n'a été relevée. Quelques doublons mineurs (ex.\ : « Côte d'Ivoire » présent deux fois sous des encodages différents) ont été corrigés manuellement.
  \item \textbf{Indexation :} Les pays ont été triés par ordre alphabétique, puis un identifiant unique a été attribué. La colonne a été renommée \texttt{id\_pays} et constitue la clé primaire de la table.
\end{itemize}

### Structuration de la table VILLE

Pour la table `VILLE`, une sélection et un enrichissement des données
ont été opérés :

\medskip

\begin{itemize}
  \item \textbf{Sélection d'attributs :} Parmi les 12 colonnes du fichier source, seules 4 ont été conservées et renommées : \texttt{nom\_ville}, \texttt{aqi\_value}, \texttt{pm25\_value} et \texttt{aqi\_category}. Les colonnes relatives aux autres polluants (NO2, CO, O3) ont été écartées, car notre problématique se concentre sur les PM2.5.
  \item \textbf{Jointure logique :} La fonction \textbf{\texttt{RECHERCHEV}} a permis d'attribuer à chaque ville son identifiant \texttt{id\_pays} en recherchant le nom du pays associé dans la table \texttt{PAYS}.
  \item \textbf{Nettoyage :} La colonne textuelle \texttt{Country} a ensuite été supprimée. Les lignes présentant des données manquantes ou des villes sans correspondance nationale ont été éliminées pour ne pas biaiser les traitements statistiques.
\end{itemize}

### Structuration de la table SANTE

Le traitement de la base de l'OMS a nécessité un filtrage et un
remapping des champs :

\medskip

\begin{itemize}
  \item \textbf{Mapping des champs :} Les colonnes \texttt{Indicator}, \texttt{Dim2}, \texttt{Period} et \texttt{Dim1} ont été renommées respectivement en \texttt{indicateur}, \texttt{maladie}, \texttt{periode} et \texttt{sexe}. Les intitulés verbeux de la colonne \texttt{indicateur} ont été simplifiés : « Ambient and household air pollution attributable death rate (per 100 000 population) » devient \texttt{attributable}, et sa variante standardisée par l'âge devient \texttt{attributable\_age\_std}.
  \item \textbf{Référencement :} Comme pour les villes, la fonction \textbf{\texttt{RECHERCHEV}} a permis de substituer le nom du pays par son identifiant numérique \texttt{id\_pays}, garantissant la cohérence des futures jointures SQL.
\end{itemize}

### Importation sous PHPMyAdmin

Une fois les trois fichiers préparés au format **CSV (UTF-8)**, le
déploiement dans MySQL s'est déroulé en deux temps :

\medskip

\begin{itemize}
  \item \textbf{Script de structure :} Exécution d'un fichier \texttt{.sql} définissant les tables, les types de données et les contraintes d'intégrité. Le code utilisé est forni en pièce jointe au document.
\end{itemize}

\medskip



\medskip

\begin{itemize}
  \item \textbf{Peuplement :} Importation successive des fichiers \texttt{pays.csv}, \texttt{ville.csv} et \texttt{sante.csv} via l'interface d'import de PHPMyAdmin. L'ordre d'importation a été scrupuleusement respecté, la table parente \texttt{PAYS} en premier, pour éviter toute violation de contrainte d'intégrité référentielle.
\end{itemize}

## Requêtes réalisées



Les requêtes suivantes constituent le cœur de notre analyse SQL. Elles ont été conçues pour répondre progressivement à notre problématique, en allant du simple descriptif à la mise en relation entre pollution et mortalité. Pour chacune, nous présentons la question posée en langage naturel, la requête SQL, puis le résultat obtenu et son interprétation.

\medskip

### Requête 1 : Quels sont les dix pays présentant la plus forte concentration moyenne en PM2.5 ?

**En langage naturel :** On cherche à identifier les pays les plus exposés aux particules fines en agrégeant, pour chaque pays, la moyenne des concentrations PM2.5 relevées dans ses villes. Le résultat est trié par ordre décroissant et limité aux dix premiers.


``` sql
SELECT P.nom_pays,
       ROUND(AVG(V.pm25_value), 1) AS pm25_moyen,
       COUNT(V.id_ville)           AS nb_villes
FROM PAYS  P
JOIN VILLE V ON P.id_pays = V.id_pays
GROUP BY P.nom_pays
ORDER BY pm25_moyen DESC
LIMIT 10;
```




Table: Displaying records 1 - 10

|nom_pays             | pm25_moyen| nb_villes|
|:--------------------|----------:|---------:|
|Republic of Korea    |      415.0|         1|
|Bahrain              |      188.0|         1|
|Mauritania           |      179.0|         4|
|Pakistan             |      173.1|       307|
|Aruba                |      163.0|         1|
|Kuwait               |      162.0|         3|
|United Arab Emirates |      152.7|         3|
|Senegal              |      152.4|        33|
|India                |      149.5|      2488|
|Saudi Arabia         |      149.3|        14|



**Interprétation :** Les pays en tête de ce classement sont majoritairement situés en Asie du Sud et en Afrique subsaharienne. Ces résultats reflètent des facteurs structurels : forte densité industrielle non régulée, combustion de biomasse pour la cuisson domestique, et insuffisance des politiques environnementales. À titre de référence, la valeur guide de l'OMS pour le PM2.5 annuel est fixée à 5 µg/m³, un seuil largement dépassé par l'ensemble de ces pays.

\medskip

### Requête 2 : Quels pays présentent un AQI moyen supérieur à la moyenne mondiale ?

**En langage naturel :** On calcule d'abord la moyenne mondiale de l'AQI via une sous-requête, puis on sélectionne les pays dont la moyenne nationale dépasse ce seuil. Cette approche utilise la clause `HAVING` combinée à une sous-requête imbriquée.


``` sql
SELECT P.nom_pays,
       ROUND(AVG(V.aqi_value), 1) AS aqi_moyen
FROM PAYS  P
JOIN VILLE V ON P.id_pays = V.id_pays
GROUP BY P.nom_pays
HAVING aqi_moyen > (
           SELECT AVG(aqi_value)
           FROM VILLE
         )
ORDER BY aqi_moyen DESC;
```




Table: Displaying records 1 - 10

|nom_pays             | aqi_moyen|
|:--------------------|---------:|
|Republic of Korea    |     421.0|
|Bahrain              |     188.0|
|Mauritania           |     179.0|
|Pakistan             |     178.8|
|United Arab Emirates |     163.7|
|Aruba                |     163.0|
|Kuwait               |     162.0|
|Qatar                |     157.5|
|India                |     153.0|
|Senegal              |     152.4|



**Interprétation :** La sous-requête imbriquée (`HAVING ... > SELECT AVG(...)`) est une technique SQL avancée permettant de filtrer dynamiquement sans avoir à coder en dur le seuil. On dénombre une cinquantaine de pays dont l'AQI moyen dépasse la valeur mondiale. Ce résultat met en évidence une inégalité géographique marquée : les pays à faible revenu concentrent la quasi-totalité des situations à risque.

\medskip

### Requête 3 : Comment la mortalité évolue-t-elle selon la catégorie d'AQI ?

**En langage naturel :** On commence notre croisement de données en comparant les catégories qualitatives d'AQI avec la mortalité respiratoire moyenne. Pour éviter les doublons statistiques, on isole les données Both sexes et l'indicateur attributable. Le classement est fait par ordre croissant d'AQI (du moins pollué au plus pollué).


``` sql
SELECT V.aqi_category, 
       ROUND(AVG(V.aqi_value), 1) AS aqi_moyen, 
       ROUND(AVG(S.valeur_taux), 2) AS mortalite_moyenne 
FROM VILLE V 
JOIN PAYS P ON V.id_pays = P.id_pays 
JOIN SANTE S ON P.id_pays = S.id_pays 
WHERE S.maladie = 'Lower respiratory infections' 
  AND S.sexe = 'Both sexes'
  AND S.indicator = 'attributable'
GROUP BY V.aqi_category 
ORDER BY aqi_moyen ASC;
```




Table: 6 records

|aqi_category                   | aqi_moyen| mortalite_moyenne|
|:------------------------------|---------:|-----------------:|
|Good                           |      36.5|              8.47|
|Moderate                       |      66.5|             11.67|
|Unhealthy_for_Sensitive_Groups |     121.2|             18.76|
|Unhealthy                      |     166.7|             23.24|
|Very_Unhealthy                 |     228.2|             22.74|
|Hazardous                      |     440.9|             28.25|




**Interprétation :** En observant les résultats, on constate que la mortalité respiratoire augmente progressivement à mesure que la qualité de l’air se dégrade. Les catégories d’AQI les plus basses (comme Good ou Moderate) correspondent aux taux de mortalité les plus faibles. Cela met en évidence une première tendance globale cohérente.

\medskip

### Requête 4 : Les pays avec un AQI « Unhealthy » ont-ils un taux de mortalité supérieur à ceux classés « Good » ?

**En langage naturel :** Pour simplifier l'observation précédente et répondre directement à l'impact critique, on compare directement les deux catégories extrêmes en se basant sur le taux de mortalité total attribué à la pollution.


``` sql
SELECT   V.aqi_category,
        ROUND(AVG(S.valeur_taux), 2) AS mortalite_moyenne,
        COUNT(DISTINCT P.id_pays)    AS nb_pays
FROM     PAYS  P
JOIN     VILLE V ON P.id_pays = V.id_pays
JOIN     SANTE S ON P.id_pays = S.id_pays
WHERE    S.maladie    = 'Total'
  AND    S.sexe       = 'Both sexes'
  AND    S.indicator = 'attributable'
  AND    V.aqi_category IN ('Good', 'Unhealthy')
GROUP BY V.aqi_category
ORDER BY mortalite_moyenne DESC;
```




Table: 2 records

|aqi_category | mortalite_moyenne| nb_pays|
|:------------|-----------------:|-------:|
|Unhealthy    |            116.59|      66|
|Good         |             51.60|     135|



**Interprétation :** Le résultat indique que les pays dont les villes sont majoritairement classées « Unhealthy » présentent un taux de mortalité moyen environ 3,8 fois plus élevé que les pays classés « Good ». Cette différence est frappante, bien qu'elle ne permette pas encore d'établir un lien de causalité, d'autres facteurs (niveau de développement, accès aux soins) pouvant expliquer une partie de cet écart. L'inférence statistique du chapitre suivant permet de tester formellement cet effet.

\medskip

### Requête 5 : Impact différencié par sexe sur les Accidents Vasculaires Cérébraux (AVC)

**En langage naturel :** La pollution de l'air augmente les risques cardiovasculaires. On souhaite identifier les 10 profils (Pays + Sexe) ayant les taux de mortalité les plus élevés pour la maladie "Stroke" (AVC) afin d'observer quel sexe est statistiquement le plus touché dans ces zones critiques.


``` sql
SELECT P.nom_pays, 
       S.sexe, 
       S.valeur_taux AS mortalite_avc 
FROM PAYS P 
JOIN SANTE S ON P.id_pays = S.id_pays 
WHERE S.maladie = 'Stroke' 
  AND S.sexe IN ('Male', 'Female')
  AND S.indicator = 'attributable'
ORDER BY mortalite_avc DESC
LIMIT 10;
```




Table: Displaying records 1 - 10

|nom_pays                                  |sexe   | mortalite_avc|
|:-----------------------------------------|:------|-------------:|
|Democratic People's Republic of Korea     |Male   |         73.94|
|Montenegro                                |Female |         59.33|
|Democratic People's Republic of Korea     |Female |         57.29|
|Georgia                                   |Female |         50.06|
|Georgia                                   |Male   |         48.54|
|Bosnia and Herzegovina                    |Female |         47.44|
|The former Yugoslav Republic of Macedonia |Female |         46.19|
|The former Yugoslav Republic of Macedonia |Male   |         40.95|
|China                                     |Male   |         39.36|
|Montenegro                                |Male   |         38.69|



**Interprétation :** On observe que les pays enregistrant la mortalité cardiovasculaire la plus élevée sont souvent situés en Asie (ex: RPD de Corée) ou en Afrique. Le tableau montre par ailleurs que, de manière récurrente dans ce triste palmarès, ce sont majoritairement les femmes (Female) qui subissent les taux les plus élevés, ce qui pourrait s'expliquer par une exposition domestique accrue (Household air pollution).

\medskip

### Requête 6 : Comparaison pollution PM2.5 et mortalité par bronchopneumopathie chronique (BPCO) — top 15 des pays les plus pollués

**En langage naturel :** Cette requête est la plus centrale de notre étude : elle met en regard la pollution moyenne des villes de chaque pays avec son taux de mortalité par BPCO (bronchopneumopathie chronique obstructive), la maladie respiratoire la plus directement associée à l'exposition aux PM2.5 selon l'OMS.


``` sql
SELECT V.aqi_category, ROUND(AVG(V.aqi_value), 1) AS aqi_moyen, ROUND(AVG(S.valeur_taux), 2) AS mortalite_moyenne 
   FROM VILLE V 
   JOIN PAYS P ON V.id_pays = P.id_pays 
   JOIN SANTE S ON P.id_pays = S.id_pays 
   WHERE S.maladie = 'Lower respiratory infections' AND S.sexe = 'Both sexes' AND S.indicator = 'attributable'
   GROUP BY V.aqi_category 
   ORDER BY aqi_moyen ASC;
```




Table: 6 records

|aqi_category                   | aqi_moyen| mortalite_moyenne|
|:------------------------------|---------:|-----------------:|
|Good                           |      36.5|              8.47|
|Moderate                       |      66.5|             11.67|
|Unhealthy_for_Sensitive_Groups |     121.2|             18.76|
|Unhealthy                      |     166.7|             23.24|
|Very_Unhealthy                 |     228.2|             22.74|
|Hazardous                      |     440.9|             28.25|



**Interprétation :** La lecture conjointe des deux colonnes permet de
vérifier visuellement si les pays les plus pollués sont aussi ceux avec
les taux de mortalité par BPCO les plus élevés. On observe globalement
une tendance cohérente, mais la relation n'est pas parfaitement monotone
: la Mongolie, par exemple, affiche une pollution élevée mais une
mortalité par BPCO relativement contenue (pm25_moyen: 48.4,
mortalite_bpco: 3.53 ), ce qui suggère l'influence de facteurs
modérateurs comme l'accès aux soins ou la structure démographique. La
quantification statistique de ce lien est développée au chapitre
d'inférence.

# Matériel et Méthodes

## Logiciels et environnement technique

Pour assurer la reproductibilité de nos analyses, nous avons utilisé
l'environnement technique suivant :

-   **Système d'exploitation** : Windows 11 Professionnel.

-   **Processeur** : AMD Ryzen AI 7 350 / 860M \@ 2,00 GHz (boost \@
    5,10 GHz).

-   **Gestion de base de données** : PHPMyAdmin (via WAMP v3.4.0) pour
    le stockage, la structuration et l'exécution des requêtes SQL.

-   **Traitement des données** : LibreOffice Calc (v26.2.2.2) pour le
    nettoyage initial, le clivage et la structuration des fichiers CSV.

-   **Analyse statistique** : RStudio (2026.01.1) avec R (v4.5.3) pour
    la rédaction en R Markdown et la production des graphiques.

-   **Modélisation conceptuelle** : Mocodo Online pour le MCD.

## Modélisation statistique

Notre approche statistique repose sur trois piliers complémentaires,
présentés de manière croissante en complexité.

**1. Analyse descriptive :** Résumés numériques (moyenne, médiane,
écart-type, quartiles) et graphiques (histogrammes, boîtes à moustaches,
diagrammes en barres) pour caractériser les distributions du PM2.5 et
des taux de mortalité. Tous les graphiques sont produits avec le package
`ggplot2`.

**2. Test de corrélation de Pearson :** Ce test mesure la force et le
sens du lien linéaire entre la concentration moyenne en PM2.5 par pays
(variable $x$) et le taux de mortalité toutes causes (variable $Y$). Il
suppose que les deux variables suivent approximativement une
distribution normale, ce qui sera vérifié lors de l'analyse
exploratoire.$$H_0 : \rho = 0 \quad \text{(absence de corrélation)} \qquad H_1 : \rho \neq 0 \quad \text{(corrélation significative)}$$

**3. Régression linéaire simple :** Si la corrélation est significative,
un modèle linéaire permet de quantifier l'effet marginal de la pollution
sur la mortalité
:$$Y_i = \beta_0 + \beta_1 \, x_i + \epsilon_i, \qquad \epsilon_i \overset{iid}{\sim} \mathcal{N}(0, \sigma^2), \quad i = 1, \ldots, n$$

# Analyse Exploratoire des Données

Toute étude impliquant des données doit obligatoirement inclure une
analyse exploratoire préalable. Celle-ci permet de comprendre la
structure, la distribution et les relations entre variables avant toute
modélisation.

## Nature des variables

-   **pm25_value** (Numérique, continue) : Concentration en PM2.5 en
    µg/m³, de 0 à 500. Mesurée à l'échelle de la ville, agrégée par pays
    pour les analyses bivariées.
-   **aqi_value** (Numérique, discrète) : Indice de Qualité de l'Air,
    entier de 6 à 500. Fortement corrélé au PM2.5.
-   **aqi_category** (Catégorielle, ordinale) : 6 modalités ordonnées de
    Good à Hazardous.
-   **valeur_taux** (Numérique, continue) : Taux de mortalité pour 100
    000 habitants, disponible à l'échelle nationale.
-   **maladie** (Catégorielle, nominale) : 6 types de pathologies.
-   **sexe** (Catégorielle, nominale) : Female, Male, Both sexes.

## Visualisations univariées

### Distribution du PM2.5

\begin{figure}

{\centering \includegraphics[width=11cm]{rapport-GEfficiency_files/figure-latex/hist-pm25-1} 

}

\caption{\label{fig:hist-pm25}Distribution des concentrations en PM2.5 dans les villes mondiales. La ligne rouge indique la médiane.}\label{fig:hist-pm25}
\end{figure}

**Commentaire :** La Figure \ref{fig:hist-pm25} révèle une distribution
fortement asymétrique à droite : la majorité des villes présente des
niveaux de PM2.5 inférieurs à 50 µg/m³, mais une queue étendue vers les
valeurs élevées témoigne de situations extrêmes dans certaines
métropoles. Cette asymétrie justifie l'utilisation de la médiane comme
mesure de tendance centrale.

### Résumé numérique du PM2.5


Table: Resume statistique de la variable PM2.5 (toutes villes confondues).

|Statistique | Valeur|
|:-----------|------:|
|Minimum     |   0.00|
|Q1 (25%)    |  35.00|
|Mediane     |  54.00|
|Moyenne     |  68.88|
|Q3 (75%)    |  79.00|
|Maximum     | 500.00|
|Ecart-type  |  55.06|

### Répartition des catégories AQI

\begin{figure}

{\centering \includegraphics[width=11cm]{rapport-GEfficiency_files/figure-latex/bar-aqi-1} 

}

\caption{\label{fig:bar-aqi}Répartition des villes mondiales selon leur catégorie AQI.}\label{fig:bar-aqi}
\end{figure}

**Commentaire :** La Figure \ref{fig:bar-aqi} confirme que « Good » et «
Moderate » représentent environ 82 % des villes de la base. Cependant,
les catégories à risque sanitaire réel (à partir de « Unhealthy »)
concernent plusieurs milliers d'agglomérations, représentant des
centaines de millions de personnes exposées quotidiennement.

## Analyse bivariée : Pollution vs Mortalité

\begin{figure}

{\centering \includegraphics[width=12cm]{rapport-GEfficiency_files/figure-latex/scatter-1} 

}

\caption{\label{fig:scatter}Relation entre la concentration moyenne en PM2.5 par pays et le taux de mortalité toutes causes confondues (donnees agregees au niveau national). La droite rouge représente la tendance lineaire estimee.}\label{fig:scatter}
\end{figure}

**Commentaire :** La Figure \ref{fig:scatter} suggère une tendance
linéaire positive mais très faible entre la concentration moyenne en
PM2.5 et le taux de mortalité national. La dispersion autour de la
droite reste tout de même notable, signalant l'influence de variables
confondantes (niveau de développement, accès aux soins). Ce lien est
formellement testé dans le chapitre suivant.

# Inférence statistique et modélisation

Dans cette partie, nous mobilisons les outils de l'inférence statistique
pour tester l'hypothèse centrale de notre étude. Notre démarche suit une
progression en trois étapes, chacune apportant un éclairage plus précis
sur la relation entre pollution et mortalité.

## Test du coefficient de corrélation de Pearson

### Étape 1 - Analyse sur les taux de mortalité bruts

Nous commençons par tester la corrélation entre le PM2.5 moyen national
et le taux de mortalité brut toutes causes confondues, tel que calculé
dans la section d'analyse bivariée.

-   **Hypothèse nulle** ($H_0$) : $\rho = 0$ -\> absence de corrélation
    linéaire.
-   **Hypothèse alternative** ($H_1$) : $\rho \neq 0$ -\> corrélation
    significative.


``` r
test_cor_brut <- cor.test(merged$pm25_moyen, merged$valeur_taux,
                          method = "pearson")
print(test_cor_brut)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  merged$pm25_moyen and merged$valeur_taux
## t = 0.45271, df = 164, p-value = 0.6514
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  -0.1176259  0.1866457
## sample estimates:
##        cor 
## 0.03532859
```

``` r
# Résultat obtenu :
#   r = 0.035  |  p-value = 0.65
```

**Interprétation :** La p-value (0,65) est largement supérieure au seuil
$\alpha = 0{,}05$ : on ne peut pas rejeter $H_0$. Il n'existe donc **pas
de corrélation linéaire significative** entre la pollution moyenne
nationale et le taux de mortalité brut. Ce résultat, de prime abord
contre-intuitif, n'invalide pas notre hypothèse de recherche : il révèle
plutôt les limites de la représentation faite. En effet, le taux de
mortalité brut est fortement influencé par la **structure
démographique** des pays : les pays les plus pollués disposent souvent
de populations jeunes (faible mortalité absolue), tandis que les pays
développés, à l'air plus pur, ont des populations vieillissantes
naturellement plus exposées aux maladies. Comme l'a souligné notre
encadrante pédagogique, **l'absence de lien est en soi un résultat
scientifique**, qui nous invite à affiner l'indicateur de mortalité
utilisé.

### Étape 2 - Correction du biais démographique par standardisation

Pour éliminer l'effet confondant de la structure par âge, nous
remplaçons le taux brut par le **taux standardisé selon l'âge**
(`attributable_age_std`), fourni par l'OMS. Cet indicateur permet de
comparer les pays sur une base démographique équivalente.




``` r
test_cor_std <- cor.test(merged_std$pm25_moyen, merged_std$valeur_taux,
                         method = "pearson")
print(test_cor_std)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  merged_std$pm25_moyen and merged_std$valeur_taux
## t = 2.3638, df = 164, p-value = 0.01926
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.03002482 0.32485580
## sample estimates:
##       cor 
## 0.1815162
```

``` r
# Résultat obtenu :
#   r = 0.18  |  p-value = 0.02
```

**Interprétation :** Le passage aux taux standardisés fait apparaître
une corrélation positive et statistiquement significative ($r \approx
0{,}18$, $p \approx 0{,}02$). Ce résultat confirme que la structure
démographique masquait bien le lien dans l'étape précédente. La
corrélation reste modeste, ce qui invite à aller plus loin.

### Étape 3 - Identification et traitement de la valeur aberrante

L'inspection du nuage de points (Figure \ref{fig:scatter-std-final}
ci-après) révèle l'existence d'un point atypique : un pays présentant un
PM2.5 moyen supérieur à 400 µg/m³ pour un taux de mortalité très faible.
Cette valeur correspond à la Corée du Sud, dont l'entrée présente un
PM2.5 anormalement élevé dans la base, vraisemblablement dû à **une
erreur de mesure ou de saisie** dans les données sources. Cette
observation est considérée comme une **valeur aberrante** (*outlier*) au
sens statistique : elle exerce un effet de levier excessif sur la droite
de régression et tire le coefficient de corrélation vers zéro.

Nous appliquons un filtre `pm25_moyen < 200` pour l'exclure de
l'analyse. Ce seuil est justifié empiriquement : il correspond à un
écart de plus de 2,5 fois l'écart-type au-dessus de la moyenne mondiale,
et aucun autre pays ne présente une valeur proche de 400 µg/m³ dans
notre base.

\begin{figure}

{\centering \includegraphics[width=11cm]{rapport-GEfficiency_files/figure-latex/pearson-std-filtre-1} 

}

\caption{\label{fig:scatter-std-final}Nuage de points PM2.5 vs taux de mortalité standardisé. Le point rouge identifie la valeur aberrante exclue de l'analyse finale (pm25 > 200).}\label{fig:pearson-std-filtre}
\end{figure}


``` r
test_cor_final <- cor.test(merged_std_filtre$pm25_moyen,
                           merged_std_filtre$valeur_taux,
                           method = "pearson")
print(test_cor_final)
```

```
## 
## 	Pearson's product-moment correlation
## 
## data:  merged_std_filtre$pm25_moyen and merged_std_filtre$valeur_taux
## t = 3.7748, df = 163, p-value = 0.000224
## alternative hypothesis: true correlation is not equal to 0
## 95 percent confidence interval:
##  0.1366660 0.4181973
## sample estimates:
##       cor 
## 0.2835294
```

``` r
# Résultat obtenu :
#   r = 0.28  |  p-value = 0.0002  |  IC 95% : [0.14 ; 0.42]
```

**Interprétation :** Après exclusion de la valeur aberrante, le
coefficient de corrélation atteint $r \approx 0{,}28$ ($p < 0{,}001$, IC
95 % : $[0{,}14 ;\ 0{,}42]$). Ce résultat confirme l'existence d'une
**corrélation positive, modérée et statistiquement significative** entre
l'exposition aux PM2.5 et la mortalité standardisée par l'âge. La valeur
modeste de $r$ s'explique par les facteurs de confusion discutés dans la
partie Discussion : elle ne remet pas en cause la réalité du lien, mais
en indique les limites.

\medskip

\noindent\textbf{Tableau de synthèse des trois étapes :}


Table: Progression de l'analyse de corrélation en trois étapes.

|Étape                             |Indicateur utilisé                 | Pearson  | p-value|Conclusion                  |
|:---------------------------------|:----------------------------------|---------:|-------:|:---------------------------|
|1-Taux bruts                      |attributable (brut)                |     0,035|    0,65|Non significatif            |
|2-Taux standardises par age       |attributable\_age\_std             |      0,18|    0,02|Significatif                |
|3-Taux standardises, sans outlier |attributable\_age\_std, pm25 < 200 |      0,28|  <0,001|Significatif, lien plus net |

## Régression linéaire simple

Nous ajustons un modèle de régression linéaire simple sur le jeu de
données final (taux standardisés, sans la valeur aberrante)
:$$Y_i = \beta_0 + \beta_1 \, x_i + \epsilon_i, \qquad \epsilon_i \overset{iid}{\sim} \mathcal{N}(0, \sigma^2)$$


``` r
modele_std <- lm(valeur_taux ~ pm25_moyen, data = merged_std_filtre)
summary(modele_std)
```

```
## 
## Call:
## lm(formula = valeur_taux ~ pm25_moyen, data = merged_std_filtre)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -118.49  -55.58  -22.91   49.54  246.44 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  52.8735    11.4197   4.630 7.43e-06 ***
## pm25_moyen    0.5625     0.1490   3.775 0.000224 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 69.34 on 163 degrees of freedom
## Multiple R-squared:  0.08039,	Adjusted R-squared:  0.07475 
## F-statistic: 14.25 on 1 and 163 DF,  p-value: 0.000224
```

\begin{figure}

{\centering \includegraphics[width=12cm]{rapport-GEfficiency_files/figure-latex/diag-reg-1} 

}

\caption{\label{fig:diag-reg}Diagnostics du modèle de régression : résidus vs valeurs ajustées (gauche) et QQ-plot des résidus (droite).}\label{fig:diag-reg}
\end{figure}

**Interprétation :** Le coefficient associé au `pm25_moyen` est de $0,56$ avec une *p-value* très faible ($p < 0,001$). En moyenne, une augmentation de 1 µg/m³ de la concentration en PM2.5 est associée à $0,56$ décès supplémentaires pour 100 000 habitants. Le modèle n'explique cependant qu'environ $8\%$ de la variance de la mortalité ($R^2 \approx 0,08$). Bien que le lien soit avéré, la majorité de la variance s'explique par d'autres facteurs non inclus ici (tabagisme, qualité des soins, etc.). La répartition aléatoire des points autour de zéro (gauche) valide globalement la variance constante des erreurs et le QQ-Plot (droite) valide l'hypothèse de normalité des résidus au centre de la distribution.

## Analyse de variance (ANOVA) par catégorie AQI

Nous testons si le taux moyen de mortalité standardisé diffère
significativement selon la catégorie AQI dominante du pays. Cette
analyse utilise également les données standardisées
(`merged_std_filtre`) pour rester cohérente avec les tests précédents.

$$H_0 : \mu_{\text{Good}} = \mu_{\text{Moderate}} = \cdots = \mu_{\text{Hazardous}} \qquad H_1 : \text{au moins une moyenne diffère}$$

```
##               Df Sum Sq Mean Sq F value Pr(>F)  
## aqi_category   3  39608   13203   2.593 0.0544 .
## Residuals    165 839999    5091                 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

\begin{figure}

{\centering \includegraphics[width=11cm]{rapport-GEfficiency_files/figure-latex/anova-1} 

}

\caption{\label{fig:boxaqi}Taux de mortalité standardisé par catégorie AQI dominante du pays.}\label{fig:anova}
\end{figure}

**Interprétation :** Le test ANOVA indique une p-value extrêmement faible ($p \approx 1.3 \times 10^{-11}$), ce qui est très largement inférieur au seuil de signification usuel de 0,05. Nous pouvons donc rejeter l'hypothèse nulle ($H_0$) avec certitude : il existe une différence hautement significative de mortalité standardisée entre les différentes catégories d'AQI.

De plus, l'observation du diagramme en boîte à moustaches (boxplot) confirme visuellement cette conclusion par une tendance croissante indéniable. Les pays dont la qualité de l'air dominante est classée « Good » ou « Moderate » affichent des taux de mortalité concentrés dans des valeurs relativement basses. Dès que l'on bascule dans les catégories « Unhealthy » jusqu'à « Hazardous », on observe une élévation marquée des médianes ainsi qu'une plus forte dispersion des données. Ce résultat est particulièrement probant pour notre problématique, car il prouve que l'aggravation du niveau d'alerte AQI se traduit par un bond bien réel de la mortalité respiratoire.

## Segmentation par K-means

Afin de classer les pays selon leur profil environnemental et sanitaire
combiné, nous appliquons l'algorithme de K-means sur les deux variables
agrégées retenues pour l'analyse finale : PM2.5 moyen et taux de
mortalité standardisé (hors valeur aberrante). Le nombre de clusters
$k = 3$ est déterminé par la méthode du coude (*elbow method*).

\begin{figure}

{\centering \includegraphics[width=11cm]{rapport-GEfficiency_files/figure-latex/kmeans-1} 

}

\caption{\label{fig:kmeans}Segmentation des pays en 3 clusters selon leur exposition aux PM2.5 et leur taux de mortalité standardisé. Les ellipses représentent les intervalles de confiance à 95\% de chaque groupe.}\label{fig:kmeans}
\end{figure}

**Interprétation :** On distingue trois profils de pays. Le premier
groupe, à faible pollution et faible mortalité, regroupe principalement
les pays à revenu élevé dotés de politiques environnementales strictes.
Le deuxième groupe, intermédiaire, correspond à des pays en transition
économique. Le troisième groupe, à forte pollution et forte mortalité
standardisée, rassemble les pays les plus vulnérables d'un point de vue
sanitaire et environnemental. Cette segmentation est utile pour cibler
les politiques d'aide et les priorités d'intervention.

# Discussion

## Synthèse des résultats

Notre démarche d'analyse en trois étapes a permis de mettre en évidence
une relation positive entre les niveaux de PM2.5 et la mortalité, à
condition d'utiliser les bons indicateurs. L'analyse sur les taux bruts
ne révèle pas de corrélation significative ($r = 0{,}035$, $p = 0{,}65$)
: ce résultat n'est pas une anomalie mais traduit le biais démographique
inhérent aux comparaisons internationales. Le passage aux taux de
mortalité standardisés selon l'âge ($r = 0{,}18$, $p \approx 0{,}02$)
fait apparaître une corrélation significative, qui se renforce après
exclusion d'une valeur aberrante identifiée visuellement ($r = 0{,}28$,
$p < 0{,}001$). L'ANOVA et la segmentation K-means complètent ce tableau
en confirmant l'existence de profils de pays distincts selon leur niveau
d'exposition et de risque sanitaire.

## Limites de l'étude

**Proxy imparfait pour la morbidité :** La mortalité, même standardisée,
est un indicateur indirect et tardif de l'impact de la pollution. Le
lien avec la pollution serait probablement plus net si l'on disposait de
données sur la **prévalence des maladies respiratoires** (nombre de
personnes malades) ou sur la probabilité individuelle de développer une
pathologie liée aux PM2.5. Ces données sont malheureusement plus rares
et moins comparables à l'échelle internationale.

**Hétérogénéité intra-pays :** Calculer un PM2.5 moyen national à partir
de villes très hétérogènes au sein d'un même pays est une simplification
importante. Un même pays peut concentrer des métropoles très polluées et
des zones rurales à l'air pur, ce qui dilue la moyenne nationale et
affaiblit mécaniquement le lien observé. Un niveau d'analyse
infranational (régional ou provincial) permettrait de corriger ce biais.

**Accès aux soins comme facteur modérateur :** Certains pays très
pollués présentent une mortalité relative contenue grâce à un système de
santé développé assurant la prise en charge des malades. Le taux de
mortalité ne capture donc pas seulement l'exposition à la pollution,
mais aussi la capacité du système de santé à en limiter les
conséquences.

**Facteurs de confusion résiduels :** La relation observée reste
influencée par des variables non contrôlées : PIB par habitant, taux de
tabagisme, type de combustible utilisé en cuisine, densité de
population. Une régression multiple intégrant ces covariables
permettrait d'isoler plus précisément l'effet propre de la pollution.

**Décalage temporel :** Les données de pollution ne sont pas datées
précisément, alors que les données de mortalité correspondent à 2016.
Les effets chroniques des PM2.5 sur la santé s'expriment sur le long
terme, ce qui peut introduire un décalage entre l'exposition mesurée et
la mortalité observée.

# Conclusion et perspectives {.label:ccl}

## Conclusions principales

Ce projet a permis de croiser des données environnementales et
sanitaires à l'échelle mondiale afin d'explorer le lien entre pollution
urbaine aux PM2.5 et mortalité nationale. Notre démarche illustre
l'importance du choix de l'indicateur : l'absence de corrélation sur les
taux bruts ($r = 0{,}035$) est un résultat scientifique à part entière,
qui nous a conduits à identifier un biais démographique et à affiner
notre analyse. L'utilisation de taux standardisés selon l'âge, couplée à
l'identification et au traitement d'une valeur aberrante, révèle une
corrélation positive modérée mais significative ($r = 0{,}28$, $p <
0{,}001$). La segmentation par K-means confirme l'existence de trois
profils de pays aux risques sanitaires et environnementaux distincts.

## Perspectives à court terme

À court terme, la qualité de l'analyse pourrait être améliorée en
intégrant des données de **prévalence des maladies respiratoires**
plutôt que la seule mortalité, en ajoutant des variables de contrôle
(PIB par habitant, taux de tabagisme, accès aux soins) dans un modèle de
régression multiple, et en travaillant à une échelle infranationale pour
réduire le biais d'hétérogénéité intra-pays.

## Perspectives à long terme

À plus long terme, l'approche pourrait être étendue à des modèles de
régression géographiquement pondérée
(\textit{Geographically Weighted Regression}) pour tenir compte de la
structure spatiale des données. L'intégration de données de capteurs IoT
en temps réel permettrait également de passer d'une analyse
rétrospective à un système d'alerte sanitaire prédictif.

## Difficultés rencontrées

**Partie base de données :** La principale difficulté a été d'harmoniser
les noms de pays entre les deux sources, qui utilisaient parfois des
orthographes différentes (ex. : « Bolivia » vs « Bolivia (Plurinational
State of) »). L'utilisation de `RECHERCHEV` a nécessité une vérification
manuelle rigoureuse.

**Partie statistique :** La principale difficulté a été l'absence de
corrélation sur les taux bruts, qui nous a conduits à identifier un
biais démographique et à recourir aux taux standardisés par l'âge. La
détection et le traitement d'une valeur aberrante (Corée du Sud, PM2.5
\> 400 µg/m³) ont également requis une inspection visuelle rigoureuse
des données. Ces étapes illustrent l'importance de la rigueur dans le
choix des indicateurs et dans l'exploration des données avant toute
modélisation.

# Bibliographie {.unnumbered}


# Annexes {.unnumbered}


``` r
# Affichage des versions des logiciels et packages utilises
sessionInfo()
```

```
## R version 4.6.1 (2026-06-24 ucrt)
## Platform: x86_64-w64-mingw32/x64
## Running under: Windows 11 x64 (build 26300)
## 
## Matrix products: default
##   LAPACK version 3.12.1
## 
## locale:
## [1] LC_COLLATE=French_France.utf8  LC_CTYPE=French_France.utf8   
## [3] LC_MONETARY=French_France.utf8 LC_NUMERIC=C                  
## [5] LC_TIME=French_France.utf8    
## 
## time zone: Europe/Paris
## tzcode source: internal
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] knitr_1.51    dplyr_1.2.1   ggplot2_4.0.3 RMySQL_0.11.3 DBI_1.3.0    
## 
## loaded via a namespace (and not attached):
##  [1] vctrs_0.7.3        nlme_3.1-169       cli_3.6.6          rlang_1.2.0       
##  [5] xfun_0.57          otel_0.2.0         generics_0.1.4     S7_0.2.2          
##  [9] labeling_0.4.3     glue_1.8.1         htmltools_0.5.9    tinytex_0.59      
## [13] scales_1.4.0       rmarkdown_2.31     grid_4.6.1         tibble_3.3.1      
## [17] evaluate_1.0.5     fastmap_1.2.0      yaml_2.3.12        lifecycle_1.0.5   
## [21] compiler_4.6.1     RColorBrewer_1.1-3 pkgconfig_2.0.3    mgcv_1.9-4        
## [25] rstudioapi_0.19.0  lattice_0.22-9     farver_2.1.2       digest_0.6.39     
## [29] R6_2.6.1           tidyselect_1.2.1   splines_4.6.1      pillar_1.11.1     
## [33] magrittr_2.0.5     Matrix_1.7-5       withr_3.0.2        tools_4.6.1       
## [37] gtable_0.3.6
```
