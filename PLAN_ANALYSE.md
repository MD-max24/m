# Plan d'analyse pré-enregistré — ZFE de Strasbourg et NO₂

Rédigé le 22 octobre 2026, avant toute estimation et avant toute analyse graphique des données.
Tout écart à ce plan sera signalé et justifié dans le rapport, dans une section « Écarts au plan ».

## 1. Question et hypothèses

**Question principale.** L'interdiction de circuler des véhicules Crit'Air 5 et non classés dans la ZFE de Strasbourg (1er janvier 2023) est-elle associée à une baisse de la concentration de NO₂, par rapport à la phase pédagogique qui la précède, une fois retirées la tendance commune à Metz, Nancy et Mulhouse et l'influence de la météo ?

**Hypothèses.**

- H1 : l'effet propre de l'interdiction des Crit'Air 5 (β₂ − β₁, voir section 5) est négatif.
- H2 : cet effet est plus marqué sur les stations trafic que sur les stations de fond urbain.
- H3 (complément) : l'interdiction des Crit'Air 4 en 2024 (β₃ − β₂) est associée à une baisse supplémentaire.

La conclusion portera sur une **association** observée dans une étude de cas (une ville traitée, trois villes témoins), et non sur l'efficacité des ZFE en général.

## 2. Données

| Données | Source | Détail |
|---|---|---|
| NO₂, NO | Agence européenne pour l'environnement (EEA), service de téléchargement | Mesures horaires validées (E1a), 2017–2024 ; heures retenues : codes de validité 1, 2, 3 |
| Type et position des stations | EEA, `PanEuropean_metadata.csv` | Type trafic / fond / industrielle, zone |
| Météo | Météo-France, données climatologiques quotidiennes | Strasbourg-Entzheim, Metz-Frescaty, Nancy-Essey, Mulhouse |

Collecte effectuée le 22 octobre 2026. Fichier d'analyse : `data/clean/base_zfe_station_jour.csv`.

**Règles de construction (déjà appliquées dans la base) :**

- moyenne journalière calculée seulement si au moins 18 heures valides sur 24 ;
- moyenne aux heures de pointe (7 h, 8 h, 17 h, 18 h) seulement si au moins 3 de ces 4 heures sont valides ;
- aucune imputation dans le modèle principal : les jours manquants sont laissés de côté.

## 3. Échantillon de stations

Stations retenues : `panel_complet = 1` (au moins 80 % des jours couverts en 2017, 2018, 2019, 2022, 2023 et 2024) et `a_exclure = 0`. Soit 11 stations :

| Ville | Rôle | Stations trafic | Stations de fond |
|---|---|---|---|
| Strasbourg | Traitée | FR16034 | FR16029 |
| Metz | Témoin | FR01064 | FR01011, FR01012 |
| Nancy | Témoin | FR30016, FR30036 | FR30034 |
| Mulhouse | Témoin | FR16069 | FR16057, FR16066 |

Stations écartées : FR01022 (industrielle, arrêtée en 2020), FR16001 et FR16038 (arrêtées en 2020), FR16040 (arrêtée fin 2023), FR16215 (ouverte en 2022), FR42010 (ouverte fin 2019). Cette liste est figée et ne sera pas modifiée au vu des résultats.

## 4. Fenêtre et phases

| Période | Phase | Usage |
|---|---|---|
| 1er janv. 2017 – 29 févr. 2020 | P0 : référence | Référence |
| 1er mars 2020 – 30 juin 2021 | Covid | Exclue |
| 1er juil. 2021 – 30 nov. 2021 | P0 : référence | Référence |
| 1er déc. 2021 – 31 janv. 2022 | Ouverture du GCO (17 déc. 2021), entrée en vigueur de la ZFE (1er janv. 2022) | Exclue |
| 1er févr. 2022 – 14 déc. 2022 | P1 : phase pédagogique | Traitement 1 |
| 15 déc. 2022 – 15 janv. 2023 | Fêtes autour de la rupture | Exclue |
| 16 janv. 2023 – 14 déc. 2023 | P2 : interdiction Crit'Air 5 | Traitement 2 (principal) |
| 15 déc. 2023 – 15 janv. 2024 | Fêtes | Exclue |
| 16 janv. 2024 – 31 déc. 2024 | P3 : interdiction Crit'Air 4 | Traitement 3 (complément) |

**Dates restant à confirmer, et règle fixée dès maintenant :**

- date de début de la verbalisation des Crit'Air 5 : si elle est postérieure au 1er janvier 2023, P2 commence 15 jours après cette date, et la zone tampon précédente est prolongée jusqu'à elle ;
- date d'interdiction des Crit'Air 4 : même règle pour P3 ;
- date d'interdiction des poids lourds en transit sur l'A35 : notée dans les limites ; elle ne modifie pas les phases.

## 5. Modèle principal

Différence de différences à trois phases, estimée sur données journalières, jours exclus retirés :

```
log(NO2_st) = β1·(S_s × P1_t) + β2·(S_s × P2_t) + β3·(S_s × P3_t)
              + γ1·vent_ct + γ2·temp_ct + γ3·pluie_ct
              + α_s + δ_t + ε_st
```

- s = station, t = jour, c = ville ; S = 1 pour une station de Strasbourg ;
- α_s : effet fixe station ; δ_t : effet fixe date ;
- seuls contrôles : vent, température, pluie de la ville. Le trafic et le NO ne sont pas des contrôles ;
- **résultat principal : β₂ − β₁**, converti en pourcentage par 100 × (exp(β₂ − β₁) − 1) ;
- résultats secondaires : β₁ (phase pédagogique, mêlée aux effets du GCO, non interprétée comme un effet ZFE pur) et β₃ − β₂ ;
- écarts-types regroupés par station, présentés à titre indicatif (11 grappes seulement).

Estimation en R avec le package `fixest`.

## 6. Inférence : placebos dans le temps

Avec une seule ville traitée et trois témoins, l'inférence principale repose sur des placebos dans le temps, et non sur les écarts-types classiques.

**Procédure :**

1. On ne garde que la période de référence 2017-01-01 – 2019-12-31.
2. Pour chaque fausse date de rupture D, du 1er janvier 2018 au 1er janvier 2019, mois par mois (13 dates), on garde les 12 mois avant D et les 12 mois après D.
3. On estime le même modèle avec une seule indicatrice S × (date ≥ D), en excluant le mois de décembre précédant D et le mois de janvier suivant D, comme pour la vraie rupture.
4. On obtient 13 effets placebo, qui mesurent la variabilité naturelle d'un saut d'un an à l'autre de l'écart entre Strasbourg et les témoins.

**Statistiques rapportées :**

- p-valeur placebo : part des valeurs |placebo| ≥ |β₂ − β₁|, l'effet réel compris (plus petite valeur possible : 1/14 ≈ 0,07) ;
- effet minimum détectable : 2,8 × écart-type des 13 effets placebo (puissance de 80 %, test bilatéral à 5 %).

Ces placebos se chevauchent et ne sont donc pas indépendants : cette limite sera indiquée.

## 7. Règles d'interprétation (fixées à l'avance)

- **Effet mis en évidence** : p-valeur placebo ≤ 0,10 et |β₂ − β₁| supérieur à l'effet minimum détectable.
- **Absence d'effet détectable** : |β₂ − β₁| inférieur à l'effet minimum détectable. On écrira « non concluant », et jamais « la ZFE n'a pas d'effet ».
- **Tendances parallèles** : jugées sur l'event study (section 8). Si un coefficient 2017 ou 2018 dépasse en valeur absolue l'effet minimum détectable, l'hypothèse est considérée comme fragile et le résultat sera présenté comme tel.

## 8. Event study

Un coefficient par année pour Strasbourg (2017, 2018, 2021, 2022, 2023, 2024), **année de référence 2019**, mêmes contrôles et effets fixes, mêmes exclusions. 2021 ne contient que le second semestre.

Lecture attendue : coefficients 2017, 2018 et 2021 proches de zéro (tendances parallèles) ; un coefficient 2022 négatif signalera de l'anticipation ou un effet du GCO.

## 9. Tests de robustesse

Tous seront présentés, quel que soit leur résultat.

| # | Test | Résultat attendu si l'effet principal est solide |
|---|---|---|
| R1 | Stations trafic seules, puis stations de fond seules | Effet plus fort sur le trafic (H2) |
| R2 | Retrait de chaque ville témoin tour à tour (3 estimations) | Même signe et ordre de grandeur |
| R3 | Sans les contrôles météo | Coefficient peu modifié |
| R4 | NO à la place du NO₂ | Même signe |
| R5 | NO₂ aux heures de pointe | Effet au moins aussi fort |
| R6 | Données agrégées par station et par mois (mois couverts à 75 % au moins) | Même ordre de grandeur |
| R7 | Placebos de rupture au 1er janvier 2018 et au 1er janvier 2019 | Aucun effet |
| R8 | Fenêtre 2017–2023 (sans P3) | Même β₂ − β₁ |
| R9 | Rupture à la date de verbalisation, si elle diffère du 1er janvier 2023 | Même ordre de grandeur |

## 10. Avant / après naïf

Variation du NO₂ moyen à Strasbourg entre P1 et P2, sans villes témoins. Présenté uniquement pour montrer l'écart avec l'effet estimé par la différence de différences.

## 11. Limites connues dès maintenant

- Une seule ville traitée, deux stations à Strasbourg sur toute la période.
- Le GCO ouvre 15 jours avant l'entrée en vigueur de la ZFE : β₁ mêle les deux.
- P1 et P2 durent environ 10,5 mois chacune : un seul cycle saisonnier par phase.
- Intensité réelle des contrôles inconnue.
- Autres politiques propres à Strasbourg (tram, plan vélo, chantiers) non isolées.
- Déplacement possible des vieux véhicules vers la périphérie.

## 12. Écarts au plan

| Date | Écart | Raison |
|---|---|---|
| 22 octobre 2026 | Moyennes annuelles de NO₂ par station calculées avant la rédaction du plan (`couverture_stations.csv`) | Contrôle de la qualité des données ; aucune estimation ni aucun graphique produit |
| 22 octobre 2026 | Coefficient ajouté pour 2020 (janvier-février) dans l'event study (section 8) | Ces deux mois appartiennent à P0 ; sans coefficient propre, ils auraient été comptés avec l'année de référence 2019 |
| 22 octobre 2026 | Zone tampon des placebos (section 6, point 3) appliquée comme 15 jours de part et d'autre de chaque fausse date | La rédaction visait une rupture au 1er janvier ; la même règle que pour la vraie rupture est appliquée aux autres dates |
| 22 octobre 2026 | Test R9 non réalisé | La date de début de la verbalisation des Crit'Air 5 n'a pas encore été confirmée |
