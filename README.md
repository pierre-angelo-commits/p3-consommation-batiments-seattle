# Anticipez les besoins en consommation de bâtiments

Projet 3 du parcours Data Scientist d'OpenClassrooms.

Seattle veut atteindre la neutralité carbone en 2050, mais mesurer la consommation d'énergie de chaque bâtiment coûte cher. Le but du projet est de prédire la consommation totale (`SiteEnergyUse(kBtu)`) des bâtiments non résidentiels à partir des informations qu'on connaît déjà sur eux (surface, usage, année de construction, quartier, nombre d'étages), sans avoir à les mesurer. C'est un problème de régression supervisée, sur les relevés de 2016.

## Résultat

Le modèle retenu est un **SVR** optimisé par GridSearch. Il obtient un **R² de 0,78** sur le jeu de test (cible en log10).

Les variables qui comptent le plus sont la surface totale du bâtiment, très loin devant, puis son usage (hôpital, bureau, entrepôt…).

J'ai aussi testé le modèle avec l'ENERGY STAR Score : il monte à 0,84, mais je ne l'ai pas gardé, car ce score est calculé à partir de la consommation mesurée et n'existe pas pour un bâtiment qui n'a jamais été mesuré.

## Notebooks

Ils sont à lancer dans l'ordre : le premier écrit un CSV que le second relit.

| Notebook | Contenu | Lit | Écrit |
|---|---|---|---|
| `01_analyse_exploratoire.ipynb` | Périmètre non résidentiel, relevés fiables, choix des colonnes, choix de la cible, consommation par type de bâtiment, bâtiments incohérents | `data/2016_Building_Energy_Benchmarking.csv` | `data/batiments_cible.csv` |
| `02_modelisation.ipynb` | Feature engineering, préparation des données, comparaison de 4 modèles, GridSearch, permutation importance | `data/batiments_cible.csv` | — |

Si on modifie le premier notebook, il faut aussi relancer le second.

## Filtres appliqués

| Étape | Avant | Après | Raison |
|---|---|---|---|
| Bâtiments non résidentiels (`BuildingType`) | 3 376 | 1 668 | Les logements ne font pas partie de la mission |
| Relevés fiables (`ComplianceStatus == "Compliant"`) | 1 668 | 1 548 | Qualité de la donnée |
| Bâtiments incohérents (0 étage ou 0 bâtiment) | 1 548 | 1 481 | Fiches impossibles |

## Comparaison des modèles

Validation croisée en 5 plis sur le jeu d'entraînement :

| Modèle | R² validation |
|---|---|
| Dummy (moyenne) | 0,00 |
| Régression linéaire | 0,71 |
| SVR | 0,73 |
| RandomForest | 0,71 |

Après GridSearch, le SVR passe à 0,74 en validation et 0,78 sur le test.

## Lancer le projet

Il faut [uv](https://docs.astral.sh/uv/), qui installe Python 3.12 et les bonnes versions des paquets (fixées dans `uv.lock`) :

```
uv sync
uv run jupyter lab
```

Ensuite, ouvrir chaque notebook et faire **Run → Run All Cells**.

## Données

`data/2016_Building_Energy_Benchmarking.csv` : relevés 2016 de la ville de Seattle (3 376 bâtiments, 46 colonnes), fournis par OpenClassrooms.
