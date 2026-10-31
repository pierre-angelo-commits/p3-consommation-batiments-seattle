# Anticipez les besoins en consommation de bâtiments

Projet 3 du parcours Data Scientist d'OpenClassrooms.

Seattle veut atteindre la neutralité carbone en 2050, mais mesurer la consommation d'énergie de chaque bâtiment coûte cher. Le but du projet est de prédire la consommation totale (`SiteEnergyUse(kBtu)`) et les émissions de CO2 (`TotalGHGEmissions`) des bâtiments non résidentiels à partir des informations qu'on connaît déjà sur eux (surface, usage, année de construction, quartier, nombre d'étages), sans avoir à les mesurer. C'est un problème de régression supervisée, sur les relevés de 2016.

## Résultat

Pour les deux cibles, le meilleur modèle est un **SVR**, après un GridSearch sur chacun des modèles testés (cibles en log10) :

| Cible | R² validation | R² test |
|---|---|---|
| Consommation d'énergie | 0,76 | 0,74 |
| Émissions de CO2 | 0,71 | 0,72 |

D'après SHAP, la surface totale du bâtiment est la variable la plus importante pour les deux cibles, suivie de son usage. Pour les émissions, le raccordement au gaz compte presque autant que la surface : l'électricité de Seattle est surtout hydroélectrique et émet peu.

J'ai aussi testé les modèles avec l'ENERGY STAR Score : ils s'améliorent (0,79 au lieu de 0,74 pour la consommation, 0,75 au lieu de 0,72 pour le CO2), mais je ne l'ai pas gardé, car ce score est calculé à partir de la consommation mesurée et n'existe pas pour un bâtiment qui n'a jamais été mesuré.

## Notebooks

Ils sont à lancer dans l'ordre : le premier écrit un CSV que les deux autres relisent.

| Notebook | Contenu | Lit | Écrit |
|---|---|---|---|
| `01_analyse_exploratoire.ipynb` | Périmètre, nettoyage, étude des deux cibles, outliers, feature engineering, corrélation, transformations (log des surfaces) | `data/2016_Building_Energy_Benchmarking.csv` | `data/batiments_cible.csv` |
| `02_modelisation.ipynb` | Consommation : GridSearch sur 4 modèles, comparaison, test, SHAP, test de l'ENERGY STAR Score | `data/batiments_cible.csv` | — |
| `03_modelisation_co2.ipynb` | Émissions de CO2 : même méthode, puis comparaison des deux cibles | `data/batiments_cible.csv` | — |

Si on modifie le premier notebook, il faut relancer les deux autres.

## Filtres appliqués

| Étape | Avant | Après | Raison |
|---|---|---|---|
| Bâtiments non résidentiels (`BuildingType`) | 3 376 | 1 668 | Les logements ne font pas partie de la mission |
| Relevés fiables (`ComplianceStatus == "Compliant"`) | 1 668 | 1 548 | Qualité de la donnée |
| Bâtiments incohérents (0 étage ou 0 bâtiment) | 1 548 | 1 481 | Fiches impossibles |
| Émissions de CO2 nulles ou négatives | 1 481 | 1 479 | Erreurs de saisie |

## Comparaison des modèles

Chaque modèle est d'abord optimisé par GridSearch (validation croisée en 5 plis sur le jeu d'entraînement), puis comparé aux autres :

| Modèle | R² validation (consommation) | R² validation (CO2) |
|---|---|---|
| Dummy (moyenne) | 0,00 | 0,00 |
| Régression linéaire | 0,74 | 0,67 |
| SVR | 0,76 | 0,71 |
| RandomForest | 0,73 | 0,69 |

## Lancer le projet

Il faut [uv](https://docs.astral.sh/uv/), qui installe Python 3.12 et les bonnes versions des paquets (fixées dans `uv.lock`) :

```
uv sync
uv run jupyter lab
```

Ensuite, ouvrir chaque notebook et faire **Run → Run All Cells**.

## Données

`data/2016_Building_Energy_Benchmarking.csv` : relevés 2016 de la ville de Seattle (3 376 bâtiments, 46 colonnes), fournis par OpenClassrooms.
