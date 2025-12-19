# PWTT – Adaptation Étudiante pour l’Analyse de Changements Sentinel-1

## Auteur
**Fabrice RENOUX ** – Étudiant / Chercheur en géomatique

## Date
2025-03-19

## Contexte
Ce projet est une adaptation étudiante du **Pixel-Wise T-Test T-Test (PWTT)** développé par **Dr. Ollie Ballinger**.  
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
# ============================================================
# 🔹 Supprimer l'ancien dossier (optionnel)
# ============================================================
import shutil, os
repo_path = "/content/test_pwtt"
if os.path.exists(repo_path):
    shutil.rmtree(repo_path)

# ============================================================
# 🔹 Cloner le dépôt GitHub contenant pwtt.py
# ============================================================
!git clone https://github.com/renouxfabrice/test_pwtt.git

# ============================================================
# 🔹 Définir le chemin vers pwtt.py et l'importer
# ============================================================
pwtt_path = "/content/test_pwtt/code/pwtt.py"

import sys, importlib.util

# Forcer le rechargement si déjà importé
if 'pwtt' in sys.modules:
    del sys.modules['pwtt']

spec = importlib.util.spec_from_file_location("pwtt", pwtt_path)
pwtt = importlib.util.module_from_spec(spec)
sys.modules["pwtt"] = pwtt
spec.loader.exec_module(pwtt)
print("🚀 Module pwtt chargé avec succès !")


# ============================================================
# 🔹 Imports nécessaires
# ============================================================
import ee, geemap, ipywidgets as widgets, datetime

# 1️⃣ Authentification Google Earth Engine
ee.Authenticate()
ee.Initialize(project='pwtt-test')         # Remplacer par le nom de votre projet sur GEE

# ============================================================
# 🔹 Paramètres utilisateur à remplir
# ============================================================

# 1️⃣ Zone d'étude (AOI) : une FeatureCollection qui délimite la zone à analyser
zone = ee.FeatureCollection('projects/pwtt-test/assets/mask_gazientep')

# 2️⃣ Footprints : bâtiments ou objets à analyser
footprints = ee.FeatureCollection('projects/pwtt-test/assets/bulding_gazientep')

# 3️⃣ Dates
pre_date = '2023-02-05'       # Date avant l'événement (ex. début des observations pré-event)
event_date = '2023-02-06'     # Date de l'événement ou post-event

# 4️⃣ Intervalles de temps (en mois)
pre_interval = 6               # Combien de mois avant l'événement pour la période pré-event
post_interval = 1              # Combien de mois après l'événement pour la période post-event

# 5️⃣ Export / affichage
export_dir = 'PWTT_TURQUIE_Export'  # Dossier sur Google Drive pour enregistrer les exports
export_name = 'Gazientep_damage'     # Nom de base pour les fichiers exportés

export_footprint_csv = False          # Export des footprints en CSV
export_footprint_geojson = False     # Export des footprints en GeoJSON
export_grid = False                  # Export de la grille CSV
export_raster = False                # Export du raster T_statistic
export_scale = 500                   # Résolution d'export (en mètres)

# 6️⃣ Paramètres d'analyse
urban_threshold = 0.1                # Seuil pour filtrer sur les zones urbanisées
T_threshold = 3                       # Seuil T-statistic pour définir les damages
apply_terrain_flattening = False      # True si on veut corriger le signal sur les pentes
TERRAIN_FLATTENING_MODEL = 'VOLUME'  # Modèle de correction ('VOLUME' ou 'DIRECT')
DEM = ee.Image('USGS/SRTMGL1_003')   # Modèle numérique de terrain pour correction
TERRAIN_FLATTENING_ADDITIONAL_LAYOVER_SHADOW_BUFFER = 0  # Buffer supplémentaire en m

# 7️⃣ Affichage dans Colab
show_raster = True       # True pour voir le raster T_statistic
show_footprints = False  # True pour voir les footprints colorés

# ============================================================
# 🔹 Appel de la fonction filter_s1
# ============================================================
image = pwtt.filter_s1(
    aoi=zone,
    footprints=footprints,
    pre_date=pre_date,
    event_date=event_date,
    pre_interval=pre_interval,
    post_interval=post_interval,
    export_dir=export_dir,
    export_name=export_name,
    export_footprint_csv=export_footprint_csv,
    export_footprint_geojson=export_footprint_geojson,
    export_grid=export_grid,
    export_raster=export_raster,
    export_scale=export_scale,
    urban_threshold=urban_threshold,
    T_threshold=T_threshold,
    apply_terrain_flattening=apply_terrain_flattening,
    TERRAIN_FLATTENING_MODEL=TERRAIN_FLATTENING_MODEL,
    DEM=DEM,
    TERRAIN_FLATTENING_ADDITIONAL_LAYOVER_SHADOW_BUFFER=TERRAIN_FLATTENING_ADDITIONAL_LAYOVER_SHADOW_BUFFER,
    show_raster=show_raster,
    show_footprints=show_footprints
)

```python
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
