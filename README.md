# La ZFE de Strasbourg a-t-elle fait baisser le NO₂ ?

**Résultat :** l'interdiction des véhicules Crit'Air 5 et non classés (1er janvier 2023) n'est pas associée à une baisse supplémentaire du NO₂ à Strasbourg par rapport à Metz, Nancy et Mulhouse. Strasbourg décroche dès la fin de 2021, avant l'entrée en vigueur de la ZFE, avec un creux en 2022 (−11 % par rapport à 2019), puis l'écart se réduit en partie.

**[Lire le rapport complet](https://md-max24.github.io/zfe-strasbourg/)**

![Event study : écart de NO₂ entre Strasbourg et les villes témoins, par rapport à 2019](figures/fig2_event_study.png)

| Mesure | Valeur |
|---|---|
| Baisse du NO₂ à Strasbourg entre 2022 et 2023 (avant/après naïf) | −8,4 % |
| Effet propre de l'interdiction des Crit'Air 5, relativement aux témoins (β₂ − β₁) | +6,1 % (IC 95 % indicatif : −0,9 % ; +13,5 %) |
| p-valeur placebo (13 fausses ruptures 2018-2019) | 0,071 |
| Effet minimum détectable | 2,7 % |
| Données | 11 stations, 24 824 observations station × jour, 2017-2024 |

## Méthode

- **Différence de différences à trois phases** : année pédagogique 2022, interdiction des Crit'Air 5 en 2023, des Crit'Air 4 en 2024, contre une référence 2017-2019. Effets fixes station et date, contrôles vent, température et pluie.
- **Event study annuel**, référence 2019, pour vérifier les tendances parallèles.
- **Inférence par placebos dans le temps**, parce qu'il n'y a qu'une ville traitée.
- **Plan d'analyse pré-enregistré** ([`PLAN_ANALYSE.md`](PLAN_ANALYSE.md)) : hypothèses, échantillon, modèle, règles d'interprétation et tests de robustesse ont été fixés et commités avant toute estimation. Les écarts au plan sont listés en fin de fichier.

## Données

| Données | Source |
|---|---|
| NO₂ et NO horaires, validés (E1a) | Agence européenne pour l'environnement, service de téléchargement |
| Type et position des stations | EEA, `PanEuropean_metadata.csv` |
| Météo quotidienne | Météo-France, données climatologiques de base |

## Organisation du dépôt

```
├── PLAN_ANALYSE.md          plan pré-enregistré
├── index.qmd                rapport Quarto (publié dans docs/)
├── collecte/                scripts Python de téléchargement et de construction de la base
├── data/clean/              base station × jour, couverture des stations, description des variables
├── R/
│   ├── 02_import_nettoyage.R   contrôles et préparation
│   └── 03_estimation.R         modèle, event study, placebos, robustesse
├── sorties/tableaux/        résultats produits par 03_estimation.R
├── figures/                 graphiques du rapport et script qui les produit
└── docs/                    rapport HTML (GitHub Pages)
```

## Reproduire

```r
source("R/02_import_nettoyage.R")
source("R/03_estimation.R")
```

puis, dans le terminal : `quarto render`.

Dans les scripts R, la variable `racine` indique le dossier du projet : adaptez-la à votre ordinateur.

## Limites principales

Une seule ville traitée et deux stations à Strasbourg actives sur toute la période ; décrochage de 2021 non expliqué ; placebos non centrés sur zéro ; dates de verbalisation à confirmer. Détail dans le rapport.

---

Projet réalisé par Djime Mounsif, M2 Statistique et Économétrie, Université de Strasbourg. Réalisé avec l'aide d'un assistant IA, le code et la relecture.
