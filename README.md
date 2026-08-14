# 🌬️ Analyse de performance et détection d'anomalies — Parc éolien

> Modélisation de la courbe de puissance et détection d'anomalies multivariées sur des données
> SCADA réelles d'un parc éolien, à l'aide d'un Isolation Forest non supervisé.

![Score d'anomalie dans le temps](assets/Score_temporelle.png)
![courbe de puissance](assets/courbe_de_puissance.png)

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![scikit--learn](https://img.shields.io/badge/scikit--learn-1.3+-orange)
![Plotly](https://img.shields.io/badge/Plotly-Interactive-3f4f75)
![License](https://img.shields.io/badge/License-MIT-green)

---

## 🧠 En bref

Ce projet exploite des données SCADA réelles (capteurs, production, températures, statuts) d'une
turbine du parc éolien de Kelmarsh pour :

1. Construire la **courbe de puissance** de la turbine — la référence de comportement normal
   reliant vitesse du vent et puissance produite.
2. Détecter des **anomalies multivariées** via un Isolation Forest, en combinant plusieurs
   variables SCADA plutôt qu'un seul indicateur de performance.
3. Fournir des **visualisations interactives** (Plotly) permettant d'explorer les anomalies
   détectées dans leur contexte temporel et opérationnel.

L'objectif est de démontrer une approche de data science appliquée à la maintenance
industrielle : partir de données brutes non labellisées, construire une référence métier
interprétable, puis y superposer une détection d'anomalies plus fine.

---

## 📋 Sommaire

- [Le problème](#-le-problème)
- [Méthodologie](#-méthodologie)
- [Résultats](#-résultats)
- [Limites](#-limites)
- [Installation et utilisation](#-installation-et-utilisation)
- [Structure du projet](#-structure-du-projet)
- [Dataset](#️-dataset)
- [Pistes d'amélioration](#-pistes-damélioration)
- [Stack technique](#-stack-technique)

---

## 🌱 Le problème

Le suivi de performance d'un parc éolien repose en grande partie sur la détection précoce de
sous-performances ou de dérives mécaniques, avant qu'elles ne se traduisent par une panne ou une
perte de production significative. Les données SCADA (Supervisory Control and Data Acquisition)
fournissent un volume important de mesures en continu, mais sans label indiquant explicitement
les périodes anormales — un contexte typique en maintenance industrielle, où la détection
d'anomalies doit s'appuyer sur des méthodes non supervisées.

## 🔍 Méthodologie

**1. Courbe de puissance** 
Construction de la courbe de puissance selon la méthode standard de binning par tranche de
vitesse de vent (largeur 0.5 m/s, cohérente avec la norme IEC 61400-12), après exclusion des
périodes d'arrêt de la turbine. Cette courbe sert de référence de comportement attendu.

**2. Détection d'anomalies multivariée** 
Un Isolation Forest est entraîné sur sept variables couvrant différents sous-systèmes de la
turbine (vitesse du vent, puissance, vitesse de rotor, température de l'huile du multiplicateur,
température du palier du générateur, température de la nacelle, angle de pale). Ce choix permet
de détecter des anomalies invisibles sur la seule courbe de puissance — par exemple une
combinaison de légers écarts sur plusieurs variables, chacun insuffisant isolément pour être
qualifié d'anormal.

**3. Visualisation** 
Les résultats sont présentés via deux graphiques Plotly interactifs (zoom, survol, navigation
temporelle) : la localisation des anomalies sur la courbe de puissance, et leur répartition dans
le temps — utile pour distinguer des anomalies isolées (souvent du bruit de mesure) de véritables
clusters temporels, plus évocateurs d'un phénomène réel.

## 📊 Résultats

| Indicateur | Valeur |
|---|---|
| Période analysée | 2016 – 2020 |
| Lignes utilisées pour la détection | ~216 000 |
| Anomalies détectées | ~2% des observations (paramètre `contamination`) |

La courbe de puissance obtenue est cohérente avec le comportement théorique attendu (démarrage
progressif dès ~3 m/s, plateau proche de la puissance nominale du modèle de turbine autour de
2 000 kW), ce qui valide la qualité du nettoyage préalable.

Les anomalies détectées ne présentent pas de clusters temporels marqués sur la période observée,
ce qui suggère des événements ponctuels et récurrents (transitoires, bruit de mesure, conditions
de vent atypiques) plutôt qu'une dégradation progressive continue sur cette turbine.

## ⚠️ Limites

Isolation Forest évalue chaque observation indépendamment, sans notion de continuité temporelle.
Une dérive lente et progressive (par exemple une température augmentant très graduellement sur
plusieurs jours) peut ne jamais franchir individuellement le seuil d'anomalie, même si la
tendance est significative sur la durée. La vue temporelle des scores permet d'atténuer
partiellement cette limite en révélant les tendances visuellement, mais une approche
complémentaire (analyse de séries temporelles, détection de rupture) serait nécessaire pour la
traiter plus rigoureusement.

Le paramètre `contamination` (fixé à 2%) reste une hypothèse de départ raisonnable mais
arbitraire, faute de vérité terrain disponible pour la calibrer précisément.

## 🚀 Installation et utilisation

```bash
# Cloner le repo
git clone https://github.com/TON_USERNAME/wind-scada-anomaly-detection.git
cd wind-scada-anomaly-detection

# Installer les dépendances
pip install -r requirements.txt

# Lancer le notebook
jupyter notebook 01_analysis.ipynb
```

Le notebook attend les données SCADA du parc de Kelmarsh dans un dossier `dataset/`, organisées
par année (voir [Dataset](#️-dataset) pour le téléchargement).

## 📁 Structure du projet

```
wind-scada-anomaly-detection/
├── README.md
├── requirements.txt
├── 01_analysis.ipynb       # Notebook principal : courbe de puissance et détection d'anomalies
├── dataset/                 # Données SCADA (non versionnées, à télécharger séparément)
└── assets/
    └── ...                  # Captures d'écran pour ce README
```

## 🗂️ Dataset

[Kelmarsh Wind Farm Data](https://zenodo.org/record/8252025) — données SCADA réelles du parc
éolien de Kelmarsh (Royaume-Uni), en libre accès, couvrant plusieurs années et plusieurs
turbines à pas de temps 10 minutes.

## 🔭 Pistes d'amélioration

- **Requêtage SQL** : exploiter DuckDB ou SQLite pour des agrégations et comparaisons entre
  turbines directement en SQL, pertinent pour des analyses à l'échelle d'un parc entier.
- **Analyse d'importance des variables** : comparer les distributions des variables entre
  points normaux et anormaux pour identifier lesquelles contribuent le plus aux anomalies
  détectées.
- **Détection de rupture / tendance** : compléter l'approche ponctuelle actuelle par des
  méthodes de séries temporelles capables de capter les dérives lentes.
- **Extension multi-turbines** : généraliser l'analyse à l'ensemble du parc pour comparer les
  profils d'anomalies entre turbines et identifier des effets de sillage ou des singularités
  propres à une machine.

## 🛠️ Stack technique

- **Traitement de données** : pandas, NumPy
- **Modélisation** : scikit-learn (Isolation Forest, StandardScaler)
- **Visualisation** : Plotly (interactif), Matplotlib
- **Format** : Jupyter Notebook

---

*Projet réalisé dans le cadre de mon portfolio en data science, avec un intérêt particulier pour
les applications à l'énergie et à la maintenance industrielle.*
*N'hésite pas à me contacter pour toute question : vergne.clement49@gmail.com*