# Prévision de la consommation électrique d'une maison avec TSMixer et RevIN

Prévision, au pas de 10 minutes, de la consommation des appareils électriques d'une maison basse consommation, et recommandations pour décaler les usages hors de la pointe du soir.

Projet ST7 « Smart Cities » de CentraleSupélec, proposé par RTE, réalisé par **Farah Messioui**(2025).

## Contexte

RTE proposait de concevoir un système de recommandation pour optimiser la consommation énergétique des ménages. Nous avons choisi l'angle des séries temporelles : d'abord prévoir la consommation, puis s'appuyer sur ses régularités pour formuler des recommandations.

Les données décrivent une maison passive en Belgique suivie pendant 4,5 mois (janvier à mai 2016) : température et humidité de 9 pièces, météo d'une station voisine, consommation de l'éclairage, et consommation des appareils, la variable à prédire.

## Démarche

1. **Reconstitution de la série temporelle.** Les fichiers fournis sont un découpage aléatoire du jeu complet, avec des lignes mélangées dans le temps. Réunis et triés, ils forment une série complète de 19 735 mesures au pas de 10 minutes.
2. **Exploration** des profils journaliers et des liens entre variables.
3. **Évaluation chronologique** : entraînement sur les 70 premiers % de la période, validation sur les 10 % suivants, test sur les 20 derniers %. Les normalisations sont calculées sur l'entraînement uniquement.
4. **Références simples** : moyenne, profil horaire, persistance (dernière valeur connue).
5. **Modèles** : Extra Trees, et TSMixer implémenté en Keras d'après l'article original.
6. **Comparaison de trois normalisations** pour TSMixer : min-max, z-score et RevIN (normalisation d'instance réversible).
7. **Recommandations** tirées des profils de consommation.

## Résultats

Prévision de la consommation à l'instant *t* à partir des 24 heures précédentes, sur la période de test (30 avril au 27 mai 2016) :

| Modèle | R² | Erreur absolue moyenne (Wh / 10 min) |
|---|---|---|
| Moyenne | −0,01 | 44,4 |
| Profil horaire | 0,21 | 39,9 |
| Persistance | 0,57 | 29,6 |
| Extra Trees | **0,60** | 30,8 |
| TSMixer + min-max | 0,58 | 29,6 |
| TSMixer + z-score | 0,57 | 30,6 |
| TSMixer + RevIN | **0,59** | 29,8 |

**Ce qu'on en retient :**

- RevIN donne le meilleur TSMixer : normaliser chaque fenêtre puis dé-normaliser la sortie aide le modèle à suivre le changement de niveau entre l'hiver et le printemps.
- Les modèles appris ne dépassent la persistance que de peu. Ils suivent bien le niveau de base mais ne savent pas anticiper les pics, qui correspondent à la mise en marche d'un appareil, une décision humaine que les capteurs de température et d'humidité ne permettent pas de prévoir.
- La tranche 17 h - 21 h représente 17 % du temps mais 27 % de la consommation des appareils : c'est le levier principal des recommandations (décaler lave-linge, sèche-linge et lave-vaisselle, réduire les veilles).

Ces résultats reposent sur une seule exécution et une seule période de test : les écarts de quelques centièmes de R² entre les meilleurs modèles ne sont pas significatifs.

## Structure du dépôt

```
.
├── data/
│   ├── train.csv          # mesures avec la consommation (80 % des pas de temps)
│   └── test.csv           # mesures sans la consommation (20 % restants)
├── prevision_consommation_energie.ipynb
├── requirements.txt
└── README.md
```

## Lancer le projet

```bash
git clone https://github.com/CtrlAltJoy/appliances-energy-forecasting.git
cd appliances-energy-forecasting
pip install -r requirements.txt
jupyter notebook prevision_consommation_energie.ipynb
```

L'exécution complète prend une quinzaine de minutes sur un processeur classique, l'essentiel étant l'entraînement des trois TSMixer.

## Pistes d'amélioration

- Validation glissante sur plusieurs périodes et plusieurs graines aléatoires
- Horizons de prévision plus longs (plusieurs heures), où RevIN apporte le plus d'après la littérature
- Données d'occupation (présence, CO₂) pour mieux anticiper les pics
- Croisement avec un signal de tension du réseau ou un tarif heures creuses pour personnaliser les recommandations

## Références

- L. M. Candanedo, V. Feldheim, D. Deramaix. *Data driven prediction models of energy use of appliances in a low-energy house.* Energy and Buildings, 2017. Source des données.
- S.-A. Chen et al. *TSMixer: An All-MLP Architecture for Time Series Forecasting.* TMLR, 2023.
- T. Kim et al. *Reversible Instance Normalization for Accurate Time-Series Forecasting against Distribution Shift.* ICLR, 2022.

## Technologies

Python, pandas, scikit-learn, Keras / TensorFlow, Matplotlib, Jupyter
