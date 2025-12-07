# Guide de Visualisation des Données MNT (Modèle Numérique de Terrain)

## 🏔️ Origine du Projet

Ce code a été développé pour vérifier que les montagnes visibles depuis ma maison à Cagnes-sur-Mer sont bien la Corse. Par temps clair, on peut apercevoir une silhouette montagneuse à l'horizon au-dessus de la Méditerranée. La visualisation 3D du relief corse permet de confirmer cette hypothèse en comparant le profil des montagnes avec ce qui est visible depuis la Côte d'Azur.

**🤖 Note importante :** L'intégralité du code Python de ce projet a été générée par **Claude.ai** (Anthropic) à travers un dialogue interactif. Ceci démontre les capacités actuelles des assistants IA pour le développement logiciel, particulièrement pour des tâches nécessitant l'intégration de multiples bibliothèques (NumPy, Plotly, PyProj) et la résolution de problèmes techniques complexes (systèmes de coordonnées, optimisation de performance, visualisation 3D).

![Vue depuis Cagnes-sur-Mer](../uploads/corse_december.jpeg)
*Vue de la Corse depuis Cagnes-sur-Mer en décembre - Les montagnes corses sont visibles à l'horizon*

La distance entre Cagnes-sur-Mer et la côte est de la Corse est d'environ **170-200 km**, ce qui rend la Corse visible uniquement lors de conditions météorologiques exceptionnelles (air très clair, peu d'humidité). Les sommets visibles incluent probablement le Monte Cinto (2706m), point culminant de la Corse.

## 📍 Source des Données

Les données utilisées dans ce projet proviennent de l'**IGN (Institut national de l'information géographique et forestière)**.

**Site officiel :** [https://geoservices.ign.fr/bdtopo](https://geoservices.ign.fr/bdtopo)

### BDALTIV2 - Base de Données Altimétriques

- **Produit :** BDALTIV2 MNT 25M ASCII Lambert 93
- **Résolution :** 25 mètres (cellsize)
- **Projection :** Lambert 93 (EPSG:2154)
- **Système altimétrique :** IGN78 Corse
- **Format :** ASCII Grid (.asc)
- **Zone d'étude :** Corse (départements 2A et 2B)

### Structure des Fichiers ASCII Grid

Chaque fichier `.asc` contient :
- **6 lignes d'en-tête** avec les métadonnées
- **Grille de valeurs d'altitude** en mètres

```
ncols         2000
nrows         1500
xllcorner     1200000.0
yllcorner     6150000.0
cellsize      25
NODATA_value  -99999
[données d'altitude ligne par ligne...]
```

---

## 🏗️ Architecture du Code

### 1. Chargement des Données

#### `parse_mnt_header(file_path)`
Parse l'en-tête à 6 lignes des fichiers MNT ASCII.

**Retour :** Dictionnaire avec `ncols`, `nrows`, `xllcorner`, `yllcorner`, `cellsize`, `nodata_value`

#### `load_mnt_file(file_path)`
Charge un fichier MNT ASCII complet.

**Retour :** `(header, data)` où `data` est un array numpy avec NaN pour les valeurs NODATA

#### `create_coordinate_grids(header, data)`
Crée les grilles de coordonnées X et Y à partir des métadonnées.

**Important :**
- X : coordonnées Ouest → Est (croissantes)
- Y : coordonnées Nord → Sud (décroissantes dans le tableau, car la première ligne = Nord)
- Les coordonnées représentent le **centre de chaque cellule**

**Retour :** `(X, Y)` meshgrids en Lambert 93

#### `load_multiple_mnt_files(folder_list)`
Charge tous les fichiers `.asc` depuis une liste de dossiers.

**Retour :** Liste de dictionnaires contenant `{'file', 'header', 'X', 'Y', 'Z'}`

---

### 2. Fusion et Rééchantillonnage

#### `merge_mnt_data(all_data)`
Fusionne plusieurs tuiles MNT en une seule grille unifiée.

**Méthode :**
1. Calcule la boîte englobante (bounding box)
2. Crée une grille unifiée avec `cellsize` constant
3. Place chaque tuile à sa position correcte
4. Gère les zones de recouvrement

**Retour :** `(X_merged, Y_merged, Z_merged)`

#### `resample_grid(X, Y, Z, factor=4, method='mean')`
Réduit la résolution pour améliorer les performances de visualisation.

**Paramètres :**
- `factor` : Facteur de sous-échantillonnage
  - `2` : 75% de réduction (vue détaillée)
  - `4` : 93.75% de réduction (recommandé)
  - `8` : 98.4% de réduction (grands jeux de données)
  - `10` : 99% de réduction (très grands jeux de données)

- `method` : Méthode d'agrégation
  - `'mean'` : Terrain lisse, usage général
  - `'median'` : Supprime les valeurs aberrantes
  - `'max'` : Préserve les pics (montagnes)
  - `'min'` : Préserve les vallées

**Retour :** `(X_resamp, Y_resamp, Z_resamp)`

---

### 3. Visualisation

#### `plot_elevation_map(X, Y, Z, title, colorscale)`
Crée une carte d'élévation 2D interactive (heatmap Plotly).

**Caractéristiques :**
- Coordonnées Lambert 93 sur les axes
- Hover affiche : Lat/Lon (WGS84), Lambert 93, Altitude
- Rapport d'aspect 1:1 (pas de distorsion)
- Échelle de couleur personnalisable

**Colorscales disponibles :**
- `'Earth'` : Palette terre naturelle
- `'Turbo'` : Haute visibilité
- `TERRAIN_COLORSCALE` : Palette terrain personnalisée (vert → brun → gris → blanc)

#### `plot_3d_surface(X, Y, Z, title, colorscale, z_scale)`
Crée une surface 3D interactive avec relief.

**Caractéristiques :**
- Conversion Lambert 93 → WGS84 pour les coordonnées
- Exagération verticale réglable (`z_scale`)
- Rapport d'aspect géographique correct
- Hover affiche : Position complète + altitude

**Paramètre `z_scale` (exagération verticale) :**
- `1.0` : Échelle vraie (relief peut être difficile à voir)
- `2.0-3.0` : Bon pour visualisation générale
- `5.0+` : Exagération dramatique

---

## 🎨 Palette de Couleurs Terrain

```python
TERRAIN_COLORSCALE = [
    [0.0, '#2E5C3B'],   # Vert foncé (basse altitude)
    [0.2, '#4A7C4E'],   # Vert
    [0.35, '#8B9E5F'],  # Vert-jaune
    [0.5, '#C4A57B'],   # Beige/tan
    [0.65, '#9B8B7E'],  # Brun
    [0.8, '#A0A0A0'],   # Gris
    [1.0, '#FFFFFF']    # Blanc (haute altitude/neige)
]
```

Cette palette représente naturellement l'élévation : végétation basse → collines → montagnes → neige.

---

## 🚀 Utilisation

### Installation des Dépendances

```bash
pip install numpy plotly pyproj scipy --break-system-packages
```

### Exemple d'Utilisation Basique

```python
# 1. Définir les dossiers contenant les fichiers MNT
folders = [
    '/path/to/BDALTIV2_MNT_25M_ASC_LAMB93_IGN78C_D02B',  # Corse Nord
    '/path/to/BDALTIV2_MNT_25M_ASC_LAMB93_IGN78C_D02A',  # Corse Sud
]

# 2. Charger tous les fichiers
all_data = load_multiple_mnt_files(folders)

# 3. Fusionner les données
X_merged, Y_merged, Z_merged = merge_mnt_data(all_data)

# 4. Rééchantillonner (optionnel mais recommandé)
X_resamp, Y_resamp, Z_resamp = resample_grid(
    X_merged, Y_merged, Z_merged,
    factor=4,
    method='mean'
)

# 5. Visualiser en 2D
fig_2d = plot_elevation_map(
    X_resamp, Y_resamp, Z_resamp,
    title="Carte d'Élévation - Corse",
    colorscale=TERRAIN_COLORSCALE
)
fig_2d.show()

# 6. Visualiser en 3D
fig_3d = plot_3d_surface(
    X_resamp, Y_resamp, Z_resamp,
    title="Surface 3D - Corse",
    colorscale=TERRAIN_COLORSCALE,
    z_scale=2.5
)
fig_3d.show()
```

---

## 📊 Exemple de Sortie Console

```
============================================================
Loading MNT files...
============================================================
Found 45 files in /path/to/Nord/
  Loading: BDALTIV2_25M_FXX_1218_6174_MNT_LAMB93_IGN78C.asc
  Loading: BDALTIV2_25M_FXX_1218_6199_MNT_LAMB93_IGN78C.asc
  ...

============================================================
Merging all files...
============================================================

🗺️  Merged Grid Bounds:
   X: 1190000.0 to 1240000.0 (range: 50000.0 m)
   Y: 6150000.0 to 6210000.0 (range: 60000.0 m)
   Cell size: 25 m
   Grid size: 2400 rows × 2000 cols = 4,800,000 cells

📈 Original Merged Data:
   Grid size: (2400, 2000)
   Total points: 4,800,000
   Valid cells: 3,245,678 (67.6%)
   Min elevation: 0.5 m
   Max elevation: 2706.3 m
   Mean elevation: 542.8 m

============================================================
Resampling merged data...
============================================================

📊 Resampling Statistics:
   Original: (2400, 2000) = 4,800,000 points
   Resampled: (600, 500) = 300,000 points
   Reduction: 93.8% fewer points
   Method: mean, Factor: 4

🌍 Converting coordinates Lambert 93 → WGS84...

📐 3D Aspect Ratios:
   X range: 50000.0 m → aspect: 0.833
   Y range: 60000.0 m → aspect: 1.000
   Z range: 2705.8 m → aspect: 0.113 (scale factor: 2.5x)
   Geographic ratio X/Y: 0.833

🌍 Geographic Bounds:
   Latitude:  42.123456° to 42.987654° (0.864198° span)
   Longitude: 8.234567° to 9.012345° (0.777778° span)

============================================================
✅ Done!
============================================================
```

---

## 🔧 Optimisations de Performance

### Problème : Grandes Données = Lenteur

Avec des résolutions de 25m sur de grandes zones, vous pouvez avoir **plusieurs millions de points** :
- Corse complète : ~4-8 millions de cellules
- Temps de rendu : lent sur des machines modestes
- Mémoire : plusieurs Go

### Solution : Rééchantillonnage Intelligent

```python
# Pour visualisation exploratoire rapide
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=8, method='mean')

# Pour analyse détaillée
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=2, method='max')

# Pour préserver les pics (montagnes)
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=4, method='max')

# Pour préserver les vallées
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=4, method='min')
```

### Recommandations

| Taille Données | Factor | Points Finaux | Usage |
|----------------|--------|---------------|-------|
| < 1M points | 1-2 | 250K-1M | Analyse détaillée |
| 1-5M points | 4 | 60K-300K | **Visualisation standard** |
| 5-10M points | 8 | 80K-160K | Vue d'ensemble rapide |
| > 10M points | 10+ | < 100K | Exploration initiale |

---

## 🌍 Systèmes de Coordonnées

### Lambert 93 (EPSG:2154)
- **Type :** Projection conique conforme de Lambert
- **Zone :** France métropolitaine
- **Unités :** Mètres
- **Origine :** Paris (longitude 3°E)
- **Usage :** Toutes les données officielles IGN

### WGS84 (EPSG:4326)
- **Type :** Coordonnées géographiques
- **Unités :** Degrés décimaux
- **Usage :** GPS, web maps (Google Maps, OpenStreetMap)

### Conversion Exemple

```python
from pyproj import Transformer

# WGS84 → Lambert 93
transformer = Transformer.from_crs("EPSG:4326", "EPSG:2154", always_xy=True)
x_lambert, y_lambert = transformer.transform(8.88211, 42.34361)  # lon, lat
# Résultat: X=1213311.64 m, Y=6174951.51 m

# Lambert 93 → WGS84
transformer_inv = Transformer.from_crs("EPSG:2154", "EPSG:4326", always_xy=True)
lon, lat = transformer_inv.transform(1213311.64, 6174951.51)
# Résultat: lon=8.88211°, lat=42.34361°
```

---

## 🗺️ Conventions d'Orientation

### Dans le Code

```
Lambert 93 (Projection Plane)
┌─────────────────────────────┐
│ Nord (Y croissant)          │
│  ↑                          │
│  │                          │
│  │                          │
│  └────────→ Est (X croissant)│
└─────────────────────────────┘
```

### Dans les Tableaux NumPy

```
    Colonnes (j) →  (X croissant = Ouest → Est)
Lignes (i)  ┌───┬───┬───┬───┐
     ↓      │ 0 │ 1 │ 2 │ 3 │  ← Ligne 0 = Y maximum (Nord)
(Y décr.)   ├───┼───┼───┼───┤
            │ 1 │   │   │   │
            ├───┼───┼───┼───┤
            │ 2 │   │   │   │
            └───┴───┴───┴───┘  ← Dernière ligne = Y minimum (Sud)
```

**Important :** Y décroît du Nord au Sud dans le tableau, mais Plotly affiche correctement avec Y croissant vers le haut.

---

## 🐛 Dépannage

### Problème : Hover affiche "%{text}" au lieu des coordonnées

**Cause :** Incompatibilité du `hovertemplate` avec `go.Surface`

**Solution :**
```python
# ❌ Incorrect pour Surface
fig = go.Figure(data=go.Surface(
    text=hover_text,
    hovertemplate='%{text}<extra></extra>'
))

# ✅ Correct pour Surface
fig = go.Figure(data=go.Surface(
    hovertext=hover_text,
    hoverinfo='text'
))
```

### Problème : Est et Ouest inversés

**Cause :** Inversion incorrecte de l'axe Y

**Solution :** Ne PAS utiliser `autorange='reversed'` pour l'axe Y en Lambert 93

### Problème : Coordonnées ne correspondent pas entre tuiles et fusion

**Cause :** Utilisation de `np.linspace` au lieu de `np.arange`

**Solution :** Dans `merge_mnt_data`, utiliser :
```python
x_merged = min_x + np.arange(n_cols) * cellsize  # Préserve cellsize exact
```

### Problème : Mémoire insuffisante

**Solution :**
1. Augmenter `RESAMPLE_FACTOR` (essayer 8 ou 10)
2. Traiter les tuiles séparément
3. Utiliser method='mean' au lieu de method='median'

---

## 📚 Ressources Complémentaires

### IGN - Documentation Officielle
- [BDALTIV2 Description](https://geoservices.ign.fr/bdaltiv)
- [Documentation Lambert 93](https://geodesie.ign.fr/index.php?page=lambert93)
- [Téléchargement données](https://geoservices.ign.fr/bdtopo)

### Bibliothèques Python
- [NumPy Documentation](https://numpy.org/doc/)
- [Plotly Python Graphing](https://plotly.com/python/)
- [PyProj - Coordinate Transformations](https://pyproj4.github.io/pyproj/)
- [SciPy Interpolation](https://docs.scipy.org/doc/scipy/reference/interpolate.html)

### Standards de Projection
- [EPSG:2154 (Lambert 93)](https://epsg.io/2154)
- [EPSG:4326 (WGS84)](https://epsg.io/4326)

---

## 📄 Licence et Citation

Ce code est fourni à titre éducatif pour l'analyse de données géospatiales.

**Données :** © IGN - BD ALTIV2  
**Licence des données IGN :** [Licence Ouverte Etalab 2.0](https://www.etalab.gouv.fr/licence-ouverte-open-licence)

Lorsque vous utilisez les données IGN, merci de citer :
```
Source: IGN - BD ALTI® Version 2.0
https://geoservices.ign.fr/bdaltiv
```

---

## 🔬 Applications Scientifiques

Ce code peut être utilisé pour :

- **Géomorphologie :** Analyse du relief et des formes de terrain
- **Hydrologie :** Modélisation des bassins versants et écoulements
- **Écologie :** Étude des habitats en fonction de l'altitude
- **Risques naturels :** Cartographie des zones inondables, glissements de terrain
- **Urbanisme :** Planification territoriale tenant compte du relief
- **Tourisme :** Création de cartes de randonnée
- **Télécommunications :** Analyse de visibilité pour antennes
- **Énergies renouvelables :** Potentiel éolien/solaire selon orientation des pentes

### 🔭 Analyse de Visibilité à Longue Distance

Un cas d'usage pratique illustré par ce projet : vérifier si les montagnes de Corse sont visibles depuis Cagnes-sur-Mer (170-200 km de distance).

**Facteurs influençant la visibilité :**

1. **Distance à l'horizon (courbure terrestre)**
   ```python
   # Formule : d = 3.57 × √h (d en km, h en mètres)
   h_observer = 50  # Altitude de l'observateur à Cagnes-sur-Mer
   h_target = 2706  # Monte Cinto (point culminant de Corse)
   d_observer = 3.57 * np.sqrt(h_observer)  # ~25 km
   d_target = 3.57 * np.sqrt(h_target)      # ~186 km
   d_total = d_observer + d_target          # ~211 km
   
   # Distance réelle Cagnes-Corse : ~170-200 km
   # Conclusion : Théoriquement visible !
   ```

2. **Réfraction atmosphérique** : L'air agit comme une lentille, augmentant la distance visible de ~7-8%

3. **Conditions météorologiques** : Visibilité optimale en hiver avec air sec et froid (comme sur la photo)

4. **Obstacles intermédiaires** : Mer Méditerranée sans obstacles terrestres

**Application avec le code DEM :**
```python
def calculate_horizon_distance(h_observer, h_target):
    """Calcule la distance maximale de visibilité entre deux points."""
    # Rayon terrestre : 6371 km
    R = 6371000  # en mètres
    d_observer = np.sqrt(2 * R * h_observer)
    d_target = np.sqrt(2 * R * h_target)
    return (d_observer + d_target) / 1000  # en km

def extract_visible_peaks(X, Y, Z, observer_lat, observer_lon, azimuth_range):
    """Extrait les sommets visibles depuis un point d'observation."""
    # Convertir position observateur en Lambert 93
    transformer = Transformer.from_crs("EPSG:4326", "EPSG:2154", always_xy=True)
    obs_x, obs_y = transformer.transform(observer_lon, observer_lat)
    
    # Calculer azimut et distance pour chaque point du DEM
    dx = X - obs_x
    dy = Y - obs_y
    distances = np.sqrt(dx**2 + dy**2)
    azimuths = np.degrees(np.arctan2(dx, dy)) % 360
    
    # Filtrer par azimut (direction de la Corse : ~100-130° depuis Cagnes)
    az_min, az_max = azimuth_range
    mask = (azimuths >= az_min) & (azimuths <= az_max)
    
    # Calculer visibilité en fonction de l'altitude et distance
    visible_mask = mask.copy()
    for i in range(Z.shape[0]):
        for j in range(Z.shape[1]):
            if mask[i, j] and not np.isnan(Z[i, j]):
                d_horizon = calculate_horizon_distance(50, Z[i, j])
                if distances[i, j] / 1000 > d_horizon:
                    visible_mask[i, j] = False
    
    return visible_mask, Z[visible_mask]

# Exemple : Cagnes-sur-Mer (43.6647°N, 7.1481°E)
visible_mask, visible_elevations = extract_visible_peaks(
    X_resamp, Y_resamp, Z_resamp, 
    43.6647, 7.1481, 
    azimuth_range=(100, 130)
)

print(f"Nombre de sommets visibles : {np.sum(visible_mask)}")
print(f"Altitude max visible : {np.max(visible_elevations):.1f} m")
print(f"Altitude min visible : {np.min(visible_elevations):.1f} m")
```

Ce type d'analyse est également utile pour :
- Placement d'antennes de télécommunication
- Planification de points de vue panoramiques
- Études d'impact visuel de constructions
- Optimisation de parcs éoliens

---

**Version :** 1.0  
**Date :** Décembre 2024  
**Auteur :** Chercheur en neurosciences, expert C/C++/Python/Java  
**Développement :** Code entièrement généré par Claude.ai (Anthropic) via dialogue interactif
