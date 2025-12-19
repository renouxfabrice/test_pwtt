# PWTT – Adaptation Étudiante pour l’Analyse de Changements Sentinel-1

## Auteur
**Fabrice [Nom]** – Étudiant / Chercheur en géomatique

## Date
2025-03-19

## Contexte
Ce projet est une adaptation étudiante du **Personalized Weighted T-Test (PWTT)** développé par **Dr. Ollie Ballinger**.  
Le code original est disponible ici : [PWTT GitHub](https://github.com/oballinger/PWTT)

L’objectif de cette adaptation est de **tester la méthode dans des contextes autres que militaires**, par exemple pour :

- Catastrophes naturelles (séismes, inondations, cyclones)  
- Explosions ou incidents industriels  
- Études urbaines de changement rapide  

> ⚠️ Ceci est un travail académique, non opérationnel, à usage pédagogique et expérimental.

---

## Démarche et méthodologie

La méthode repose sur l’utilisation des **images radar Sentinel-1** pour détecter les changements sur le territoire avant et après un événement.

### 1. Prétraitement des images Sentinel-1

- Filtre **Lee** pour réduire le **bruit radar (*speckle*)** tout en conservant les contours.
- Conversion en logarithme pour homogénéiser les valeurs.
- Option : **correction topographique (terrain flattening)** pour réduire les effets de pente et d’ombre radar.

### 2. Test statistique T (PWTT)

- Pour chaque pixel, comparaison des images **avant et après événement**.
- Calcul de la **moyenne, écart-type et nombre d’observations** pour chaque période.
- Calcul d’un **T-statistic pixel par pixel** pour détecter les changements significatifs.
- Possibilité de filtrer par **zones urbaines** pour ne garder que les zones bâties (Dynamic World).

### 3. Agrégation par orbites

- Les images Sentinel-1 sont regroupées par **orbites relatives** pour éviter les biais liés à l’angle de prise de vue.
- Chaque orbite est traitée indépendamment avant fusion des résultats.

### 4. Post-traitement et analyse spatiale

- Convolution spatiale autour de chaque pixel pour ajouter le **contexte local** (50m, 100m, 150m).
- Détermination d’une **classe de dommage** selon un seuil `T_threshold`.
- Production de la **carte finale** `T_statistic` et du raster `damage`.

### 5. Analyse par footprint (optionnel)

- Si des footprints (bâtiments, zones d’intérêt) sont fournis :
  - Extraction des statistiques pour chaque entité.
  - Calcul de **surface, nombre de pixels endommagés, proportion endommagée**.
  - Classement en **niveau de confiance** (0 à 3).

### 6. Export et visualisation

- **Raster** : T-statistic ou damage  
- **Tableaux** : CSV / GeoJSON par bâtiment  
- **Affichage interactif** via `geemap` (optionnel)

---

## Dépendances

- Python 3.x  
- [Google Earth Engine Python API](https://developers.google.com/earth-engine/python_install)  
- [geemap](https://geemap.org/)  
- datetime (standard Python)

---

## Utilisation

1. **Connexion à Google Earth Engine**
```python
import ee
ee.Authenticate()
ee.Initialize()

    Importer votre zone d’intérêt (AOI)

import geopandas as gpd
aoi = ee.Geometry.Polygon([ ... ])

    Appeler la fonction principale

image_result = filter_s1(
    aoi=aoi,
    event_date='2025-01-01',
    pre_date='2024-01-01',
    pre_interval=12,
    post_interval=2,
    footprints=footprints_fc,  # facultatif
    export_raster=True,
    export_footprint_csv=True,
    show_raster=True
)

    Visualiser ou exporter

    Les exports se font automatiquement sur Google Drive.

    Affichage interactif possible avec show_raster=True et show_footprints=True.

Références

    Ballinger, O. (2020) – PWTT GitHub

    Vollrath, A., Mullissa, A., & Reiche, J. (2020). Angular-Based Radiometric Slope Correction for Sentinel-1. Remote Sensing, 12(11), 1867. doi:10.3390/rs12111867

Notes

    Ce code est entièrement reproductible, transparent et explicite.

    Les résultats dépendent fortement de :

        Qualité et disponibilité des images Sentinel-1

        Paramètres pre_interval, post_interval, T_threshold

        Taille des footprints et couverture urbaine

    Pour un usage pédagogique, il est conseillé de tester sur une petite zone d’intérêt avant de lancer une exportation globale.

Licence

Usage académique et personnel uniquement.
Non destiné à un usage opérationnel ou commercial.
