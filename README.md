
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
Parfait, voici un **texte prêt à être collé tel quel dans ton README GitHub**, structuré par **grandes étapes**, avec un vocabulaire **accessible à des non-spécialistes en mathématiques, géomatique ou télédétection**.
Chaque étape correspond directement à une partie de ton code.

---

##  Étape 1 — Cloner automatiquement le dépôt GitHub dans Google Colab

La première partie du code sert à **récupérer automatiquement le projet depuis GitHub** dans l’environnement Google Colab.

Lorsque l’on relance plusieurs fois un notebook Colab, le dossier cloné peut déjà exister. Pour éviter les erreurs, le script commence par **supprimer le dossier s’il est déjà présent**, puis **reclone proprement le dépôt**.

Cela permet :

* d’avoir toujours la dernière version du code,
* d’éviter les conflits de fichiers,
* de rendre l’exécution reproductible pour tous les utilisateurs.

L’utilisateur **n’a rien à modifier à cette étape**.

---

##  Étape 2 — Chargement du module PWTT (`pwtt.py`)

Une fois le dépôt cloné, le script :

* définit le chemin vers le fichier `pwtt.py`,
* charge ce fichier comme un **module Python**,
* force son rechargement si le notebook a déjà été exécuté auparavant.

Cette étape permet d’utiliser la fonction principale `filter_s1()` comme n’importe quelle fonction Python, sans avoir besoin de copier le code à la main.

Si le message *“Module pwtt chargé avec succès !”* s’affiche, cela signifie que :

* le dépôt est bien cloné,
* le fichier `pwtt.py` est accessible,
* le module est prêt à être utilisé.

---

##  Étape 3 — Authentification à Google Earth Engine (obligatoire)

Le traitement repose sur **Google Earth Engine (GEE)**, qui nécessite une authentification.

Lors de la première exécution :

1. Une fenêtre s’ouvre demandant de se connecter à un compte Google.
2. L’utilisateur doit autoriser l’accès à Earth Engine.
3. Un code d’authentification est fourni et doit être collé dans Colab.

⚠️ **Point très important**
Depuis les évolutions récentes de Google Earth Engine, il est **obligatoire de préciser le nom de son projet GEE** lors de l’initialisation :

```python
ee.Initialize(project='Nom_de_ton_projet_GEE')
```

L’utilisateur doit remplacer `Nom_de_ton_projet_GEE` par :

* le nom exact de son projet Earth Engine,
* visible dans la console Google Cloud associée à Earth Engine.

Sans cela, le script ne pourra pas accéder aux assets ni lancer les exports.

---

##  Étape 4 — Renseigner les assets Earth Engine (AOI et bâtiments)

Le script nécessite **deux couches vectorielles stockées dans Earth Engine** :

1. **La zone d’étude (AOI)**
   → un polygone qui définit la zone analysée.

2. **Les footprints (bâtiments)**
   → une couche de bâtiments (par exemple issue d’OpenStreetMap).
ℹ️ Données de test disponibles : dans le dossier GEE_data du dépôt, l’utilisateur trouvera des données sur Gaziantep, incluant la zone AOI (mask_gazientep) et la couche bâtiments (building_gazientep)
Exemple :

```python
zone = ee.FeatureCollection(
    'projects/Nom_de_ton_projet_GEE/assets/mask_gazientep'
)

footprints = ee.FeatureCollection(
    'projects/Nom_de_ton_projet_GEE/assets/bulding_gazientep'
)
```

### Comment récupérer le bon nom d’un asset ?

Dans l’interface Earth Engine :

1. Aller dans l’onglet **Assets**
2. Cliquer sur l’asset souhaité
3. Copier le chemin affiché (clic droit → *Copy asset ID*)

Le dépôt contient un dossier **`GEE_data`** avec :

* une zone d’étude de test (Gaziantep),
* une couche de bâtiments de test,
  que l’utilisateur peut importer dans son propre projet Earth Engine pour tester rapidement le script.

---

##  Étape 5 — Définir les dates et les périodes d’analyse

L’analyse repose sur une comparaison **avant / après événement**.

L’utilisateur doit fournir :

* une date de référence avant l’événement (`pre_date`),
* la date de l’événement (`event_date`),
* un nombre de mois avant et après pour construire les séries temporelles.

Exemple :

```python
pre_interval = 6   # 6 mois avant
post_interval = 1  # 1 mois après
```

Ces paramètres permettent d’adapter l’analyse :

* à un séisme,
* à une inondation,
* ou à tout autre événement ponctuel.

---

##  Étape 6 — Paramétrer les exports et l’affichage

L’utilisateur peut choisir :

* d’exporter ou non les résultats (CSV, GeoJSON, raster),
* la résolution spatiale du raster,
* le dossier de sortie sur Google Drive.

Les exports sont **optionnels** :
le script peut aussi être utilisé uniquement pour de la **visualisation dans Colab**.

---

##  Étape 7 — Paramètres d’analyse et seuils

Cette étape permet d’adapter l’analyse au contexte :

* `urban_threshold` : limite l’analyse aux zones urbanisées
* `T_threshold` : seuil statistique au-delà duquel un changement est considéré comme significatif
* `apply_terrain_flattening` : corrige les effets liés au relief (utile en zone montagneuse)

Pour une première utilisation, il est recommandé de :

* laisser la correction topographique désactivée,
* tester plusieurs valeurs de `T_threshold`.

---

##  Étape 8 — Lancement de l’analyse

La dernière partie du script appelle la fonction principale :

```python
pwtt.filter_s1(...)
```

À ce stade :

* toutes les données sont chargées,
* les paramètres sont définis,
* l’analyse Sentinel-1 est lancée automatiquement.

Selon les options choisies, le script :

* affiche les résultats dans Colab,
* exporte les fichiers vers Google Drive,
* calcule les statistiques par bâtiment.

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
print("Module pwtt chargé avec succès !")

# ============================================================
# 🔹 Imports nécessaires
# ============================================================
import ee, geemap, ipywidgets as widgets, datetime

# Authentification Google Earth Engine (GEE)
ee.Authenticate()
ee.Initialize(project='Nom_de_ton_projet_GEE')

# ============================================================
# 🔹 Paramètres utilisateur à remplir
# ============================================================

zone = ee.FeatureCollection(
    'projects/Nom_de_ton_projet_GEE/assets/mask_gazientep'
)  
# Zone d’étude (AOI) : polygone qui délimite la zone analysée

footprints = ee.FeatureCollection(
    'projects/Nom_de_ton_projet_GEE/assets/bulding_gazientep'
)  
# Couche de bâtiments / objets (footprints OSM ou équivalent)

pre_date = '2023-02-05'  
# Date de référence AVANT l’événement (fin de la période pré-event)

event_date = '2023-02-06'  
# Date de l’événement ou début de la période post-event

pre_interval = 6  
# Nombre de mois AVANT pre_date utilisés pour calculer la situation normale

post_interval = 1  
# Nombre de mois APRÈS event_date utilisés pour détecter les changements

export_dir = 'PWTT_TURQUIE_Export'  
# Nom du dossier créé sur Google Drive pour stocker les résultats

export_name = 'Gazientep_damage'  
# Nom de base des fichiers exportés (CSV, GeoJSON, raster)

export_footprint_csv = False  
# True → export des résultats par bâtiment en CSV

export_footprint_geojson = False  
# True → export des résultats par bâtiment en GeoJSON

export_grid = False  
# True → export d’une grille régulière (CSV) couvrant la zone d’étude

export_raster = False  
# True → export du raster final T_statistic

export_scale = 500  
# Résolution du raster exporté (en mètres, ex : 10, 30, 100, 500)

urban_threshold = 0.1  
# Seuil d’urbanisation (Dynamic World) :  
# plus la valeur est élevée, plus on se limite aux zones fortement bâties

T_threshold = 3  
# Seuil du T-statistic au-dessus duquel un pixel est considéré comme "changé"

apply_terrain_flattening = False  
# True → corrige les effets de relief (pentes, ombres radar)  
# À activer surtout en zone montagneuse

TERRAIN_FLATTENING_MODEL = 'VOLUME'  
# Modèle de correction topographique :  
# 'VOLUME' (recommandé) ou 'DIRECT'

DEM = ee.Image('USGS/SRTMGL1_003')  
# Modèle Numérique de Terrain utilisé pour la correction topographique

TERRAIN_FLATTENING_ADDITIONAL_LAYOVER_SHADOW_BUFFER = 0  
# Buffer supplémentaire (en mètres) pour élargir les zones masquées  
# liées aux ombres et aux effets radar

show_raster = True  
# True → affiche la carte raster T_statistic dans Colab

show_footprints = False  
# True → affiche les bâtiments :  
# vert = pas de dommage, rouge = dommage (contours uniquement)


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

