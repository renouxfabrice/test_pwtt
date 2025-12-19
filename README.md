
# PWTT – Adaptation Étudiante pour l’Analyse de Changements Sentinel-1

## Auteur

**Fabrice RENOUX** – Étudiant Mastère SILAT

## Date

2025-12-19

## Contexte

Ce projet est une **adaptation étudiante** du **Pixel-Wise T-Test (PWTT)** développé par **Dr. Ollie Ballinger**.
Le code original est disponible ici : [PWTT GitHub](https://github.com/oballinger/PWTT)

L’objectif est de **tester la méthode dans des contextes civils et académiques**, par exemple pour :

* Catastrophes naturelles (séismes, inondations, cyclones)
* Explosions ou incidents industriels
* Études urbaines de changement rapide

> ⚠️ Usage pédagogique uniquement, non opérationnel.

---

## Démarche et méthodologie

1. **Prétraitement Sentinel-1** : Filtre Lee, conversion logarithmique, correction topographique optionnelle.
2. **Test statistique T (PWTT)** : Comparaison pixel par pixel avant/après événement.
3. **Agrégation par orbites** : Éviter les biais liés aux angles de prise de vue.
4. **Post-traitement** : Convolutions spatiales, définition des classes de dommage.
5. **Analyse par footprint** : Statistiques pour chaque bâtiment ou entité (optionnel).
6. **Export et visualisation** : Raster, CSV, GeoJSON, affichage interactif via `geemap`.

---

## Dépendances

* Python 3.x
* [Google Earth Engine Python API](https://developers.google.com/earth-engine/python_install)
* [geemap](https://geemap.org/)
* datetime (standard Python)

---

## Utilisation dans Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://github.com/renouxfabrice/test_pwtt/code/pwtt_colab.ipynb)

### Étapes pour l’utilisateur

1. **Ouvrir le notebook dans Colab** en cliquant sur le badge ci-dessus ou copier le code ci-dessous dans votre notebook colab.
2. **Authentification Google Earth Engine** : suivre le lien généré par `ee.Authenticate()` et copier le code.
3. **Remplir les paramètres utilisateur** (zone, footprints, dates, seuils, export, affichage).ℹ️ Données de test disponibles : dans le dossier GEE_data du dépôt, l’utilisateur trouvera des données sur Gaziantep, incluant la zone AOI (mask_gazientep) et la couche bâtiments (building_gazientep)
4. **Exécuter les cellules dans l’ordre**. Le module `pwtt.py` est cloné et chargé automatiquement.


---

## Exemple de code complet à copier dans Colab

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
ee.Initialize(project='pwtt-test')

# ============================================================
# 🔹 Paramètres utilisateur à remplir
# ============================================================
zone = ee.FeatureCollection('projects/pwtt-test/assets/mask_gazientep')
footprints = ee.FeatureCollection('projects/pwtt-test/assets/bulding_gazientep')
pre_date = '2023-02-05'
event_date = '2023-02-06'
pre_interval = 6
post_interval = 1
export_dir = 'PWTT_TURQUIE_Export'
export_name = 'Gazientep_damage'
export_footprint_csv = False
export_footprint_geojson = False
export_grid = False
export_raster = False
export_scale = 500
urban_threshold = 0.1
T_threshold = 3
apply_terrain_flattening = False
TERRAIN_FLATTENING_MODEL = 'VOLUME'
DEM = ee.Image('USGS/SRTMGL1_003')
TERRAIN_FLATTENING_ADDITIONAL_LAYOVER_SHADOW_BUFFER = 0
show_raster = True
show_footprints = False

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
```

---

## Visualisation et export

* Exports automatiques sur Google Drive (`export_dir`).
* Affichage interactif possible avec `show_raster=True` et `show_footprints=True`.

---

## Références

* Ballinger, O. (2020) – PWTT GitHub
* Vollrath, A., Mullissa, A., & Reiche, J. (2020). Angular-Based Radiometric Slope Correction for Sentinel-1. *Remote Sensing*, 12(11), 1867.

---

## Notes importantes

* Reproductible, transparent et explicite.
* Dépend fortement de la qualité des images Sentinel-1, des paramètres `pre_interval`, `post_interval`, `T_threshold`, et de la taille des footprints.
* Tester sur une **petite zone** avant une exportation globale.

---

## Licence

Usage **académique et personnel uniquement**.
Non destiné à un usage opérationnel ou commercial.


