# DEM (Digital Elevation Model) Visualization Guide

## 📍 Data Source

The data used in this project comes from **IGN (Institut national de l'information géographique et forestière)** - the French National Institute of Geographic and Forest Information.

**Official website:** [https://geoservices.ign.fr/bdtopo](https://geoservices.ign.fr/bdtopo)

### BDALTIV2 - Altimetric Database

- **Product:** BDALTIV2 DEM 25M ASCII Lambert 93
- **Resolution:** 25 meters (cellsize)
- **Projection:** Lambert 93 (EPSG:2154)
- **Altimetric system:** IGN78 Corsica
- **Format:** ASCII Grid (.asc)
- **Study area:** Corsica (departments 2A and 2B)

### ASCII Grid File Structure

Each `.asc` file contains:
- **6 header lines** with metadata
- **Elevation value grid** in meters

```
ncols         2000
nrows         1500
xllcorner     1200000.0
yllcorner     6150000.0
cellsize      25
NODATA_value  -99999
[elevation data line by line...]
```

---

## 🏗️ Code Architecture

### 1. Data Loading

#### `parse_mnt_header(file_path)`
Parses the 6-line header of ASCII DEM files.

**Returns:** Dictionary with `ncols`, `nrows`, `xllcorner`, `yllcorner`, `cellsize`, `nodata_value`

#### `load_mnt_file(file_path)`
Loads a complete ASCII DEM file.

**Returns:** `(header, data)` where `data` is a numpy array with NaN for NODATA values

#### `create_coordinate_grids(header, data)`
Creates X and Y coordinate grids from metadata.

**Important:**
- X: West → East coordinates (increasing)
- Y: North → South coordinates (decreasing in array, as first row = North)
- Coordinates represent the **center of each cell**

**Returns:** `(X, Y)` meshgrids in Lambert 93

#### `load_multiple_mnt_files(folder_list)`
Loads all `.asc` files from a list of folders.

**Returns:** List of dictionaries containing `{'file', 'header', 'X', 'Y', 'Z'}`

---

### 2. Merging and Resampling

#### `merge_mnt_data(all_data)`
Merges multiple DEM tiles into a single unified grid.

**Method:**
1. Calculates bounding box
2. Creates unified grid with constant `cellsize`
3. Places each tile at its correct position
4. Handles overlapping areas

**Returns:** `(X_merged, Y_merged, Z_merged)`

#### `resample_grid(X, Y, Z, factor=4, method='mean')`
Reduces resolution to improve visualization performance.

**Parameters:**
- `factor`: Downsampling factor
  - `2`: 75% reduction (detailed view)
  - `4`: 93.75% reduction (recommended)
  - `8`: 98.4% reduction (large datasets)
  - `10`: 99% reduction (very large datasets)

- `method`: Aggregation method
  - `'mean'`: Smooth terrain, general purpose
  - `'median'`: Removes outliers
  - `'max'`: Preserves peaks (mountains)
  - `'min'`: Preserves valleys

**Returns:** `(X_resamp, Y_resamp, Z_resamp)`

---

### 3. Visualization

#### `plot_elevation_map(X, Y, Z, title, colorscale)`
Creates an interactive 2D elevation map (Plotly heatmap).

**Features:**
- Lambert 93 coordinates on axes
- Hover displays: Lat/Lon (WGS84), Lambert 93, Elevation
- 1:1 aspect ratio (no distortion)
- Customizable colorscale

**Available colorscales:**
- `'Earth'`: Natural earth palette
- `'Turbo'`: High visibility
- `TERRAIN_COLORSCALE`: Custom terrain palette (green → brown → gray → white)

#### `plot_3d_surface(X, Y, Z, title, colorscale, z_scale)`
Creates an interactive 3D surface with relief.

**Features:**
- Lambert 93 → WGS84 coordinate conversion
- Adjustable vertical exaggeration (`z_scale`)
- Correct geographic aspect ratio
- Hover displays: Complete position + elevation

**Parameter `z_scale` (vertical exaggeration):**
- `1.0`: True scale (relief may be hard to see)
- `2.0-3.0`: Good for general visualization
- `5.0+`: Dramatic exaggeration

---

## 🎨 Terrain Color Palette

```python
TERRAIN_COLORSCALE = [
    [0.0, '#2E5C3B'],   # Deep green (low elevation)
    [0.2, '#4A7C4E'],   # Green
    [0.35, '#8B9E5F'],  # Yellow-green
    [0.5, '#C4A57B'],   # Tan/beige
    [0.65, '#9B8B7E'],  # Brown
    [0.8, '#A0A0A0'],   # Gray
    [1.0, '#FFFFFF']    # White (high elevation/snow)
]
```

This palette naturally represents elevation: low vegetation → hills → mountains → snow.

---

## 🚀 Usage

### Installing Dependencies

```bash
pip install numpy plotly pyproj scipy
```

### Basic Usage Example

```python
# 1. Define folders containing DEM files
folders = [
    '/path/to/BDALTIV2_MNT_25M_ASC_LAMB93_IGN78C_D02B',  # Northern Corsica
    '/path/to/BDALTIV2_MNT_25M_ASC_LAMB93_IGN78C_D02A',  # Southern Corsica
]

# 2. Load all files
all_data = load_multiple_mnt_files(folders)

# 3. Merge data
X_merged, Y_merged, Z_merged = merge_mnt_data(all_data)

# 4. Resample (optional but recommended)
X_resamp, Y_resamp, Z_resamp = resample_grid(
    X_merged, Y_merged, Z_merged,
    factor=4,
    method='mean'
)

# 5. 2D Visualization
fig_2d = plot_elevation_map(
    X_resamp, Y_resamp, Z_resamp,
    title="Elevation Map - Corsica",
    colorscale=TERRAIN_COLORSCALE
)
fig_2d.show()

# 6. 3D Visualization
fig_3d = plot_3d_surface(
    X_resamp, Y_resamp, Z_resamp,
    title="3D Elevation Surface - Corsica",
    colorscale=TERRAIN_COLORSCALE,
    z_scale=2.5
)
fig_3d.show()
```

---

## 📊 Sample Console Output

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

## 🔧 Performance Optimizations

### Problem: Large Data = Slow Performance

With 25m resolution over large areas, you can have **several million points**:
- Complete Corsica: ~4-8 million cells
- Rendering time: slow on modest machines
- Memory: several GB

### Solution: Smart Resampling

```python
# For quick exploratory visualization
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=8, method='mean')

# For detailed analysis
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=2, method='max')

# To preserve peaks (mountains)
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=4, method='max')

# To preserve valleys
X, Y, Z = resample_grid(X_merged, Y_merged, Z_merged, factor=4, method='min')
```

### Recommendations

| Data Size | Factor | Final Points | Use Case |
|-----------|--------|--------------|----------|
| < 1M points | 1-2 | 250K-1M | Detailed analysis |
| 1-5M points | 4 | 60K-300K | **Standard visualization** |
| 5-10M points | 8 | 80K-160K | Quick overview |
| > 10M points | 10+ | < 100K | Initial exploration |

---

## 🌍 Coordinate Systems

### Lambert 93 (EPSG:2154)
- **Type:** Lambert conformal conic projection
- **Zone:** Metropolitan France
- **Units:** Meters
- **Origin:** Paris (3°E longitude)
- **Usage:** All official IGN data

### WGS84 (EPSG:4326)
- **Type:** Geographic coordinates
- **Units:** Decimal degrees
- **Usage:** GPS, web maps (Google Maps, OpenStreetMap)

### Conversion Example

```python
from pyproj import Transformer

# WGS84 → Lambert 93
transformer = Transformer.from_crs("EPSG:4326", "EPSG:2154", always_xy=True)
x_lambert, y_lambert = transformer.transform(8.88211, 42.34361)  # lon, lat
# Result: X=1213311.64 m, Y=6174951.51 m

# Lambert 93 → WGS84
transformer_inv = Transformer.from_crs("EPSG:2154", "EPSG:4326", always_xy=True)
lon, lat = transformer_inv.transform(1213311.64, 6174951.51)
# Result: lon=8.88211°, lat=42.34361°
```

---

## 🗺️ Orientation Conventions

### In Code

```
Lambert 93 (Planar Projection)
┌─────────────────────────────┐
│ North (Y increasing)        │
│  ↑                          │
│  │                          │
│  │                          │
│  └────────→ East (X increasing)│
└─────────────────────────────┘
```

### In NumPy Arrays

```
    Columns (j) →  (X increasing = West → East)
Rows (i)   ┌───┬───┬───┬───┐
     ↓     │ 0 │ 1 │ 2 │ 3 │  ← Row 0 = Y maximum (North)
(Y decr.)  ├───┼───┼───┼───┤
           │ 1 │   │   │   │
           ├───┼───┼───┼───┤
           │ 2 │   │   │   │
           └───┴───┴───┴───┘  ← Last row = Y minimum (South)
```

**Important:** Y decreases from North to South in the array, but Plotly displays correctly with Y increasing upward.

---

## 🐛 Troubleshooting

### Issue: Hover displays "%{text}" instead of coordinates

**Cause:** `hovertemplate` incompatibility with `go.Surface`

**Solution:**
```python
# ❌ Incorrect for Surface
fig = go.Figure(data=go.Surface(
    text=hover_text,
    hovertemplate='%{text}<extra></extra>'
))

# ✅ Correct for Surface
fig = go.Figure(data=go.Surface(
    hovertext=hover_text,
    hoverinfo='text'
))
```

### Issue: East and West inverted

**Cause:** Incorrect Y-axis inversion

**Solution:** Do NOT use `autorange='reversed'` for Y-axis in Lambert 93

### Issue: Coordinates don't match between tiles and merged data

**Cause:** Using `np.linspace` instead of `np.arange`

**Solution:** In `merge_mnt_data`, use:
```python
x_merged = min_x + np.arange(n_cols) * cellsize  # Preserves exact cellsize
```

### Issue: Insufficient memory

**Solution:**
1. Increase `RESAMPLE_FACTOR` (try 8 or 10)
2. Process tiles separately
3. Use method='mean' instead of method='median'

---

## 📚 Additional Resources

### IGN - Official Documentation
- [BDALTIV2 Description](https://geoservices.ign.fr/bdaltiv)
- [Lambert 93 Documentation](https://geodesie.ign.fr/index.php?page=lambert93)
- [Data Download](https://geoservices.ign.fr/bdtopo)

### Python Libraries
- [NumPy Documentation](https://numpy.org/doc/)
- [Plotly Python Graphing](https://plotly.com/python/)
- [PyProj - Coordinate Transformations](https://pyproj4.github.io/pyproj/)
- [SciPy Interpolation](https://docs.scipy.org/doc/scipy/reference/interpolate.html)

### Projection Standards
- [EPSG:2154 (Lambert 93)](https://epsg.io/2154)
- [EPSG:4326 (WGS84)](https://epsg.io/4326)

---

## 📄 License and Citation

This code is provided for educational purposes for geospatial data analysis.

**Data:** © IGN - BD ALTIV2  
**IGN Data License:** [Etalab Open License 2.0](https://www.etalab.gouv.fr/licence-ouverte-open-licence)

When using IGN data, please cite:
```
Source: IGN - BD ALTI® Version 2.0
https://geoservices.ign.fr/bdaltiv
```

---

## 🔬 Scientific Applications

This code can be used for:

- **Geomorphology:** Terrain relief and landform analysis
- **Hydrology:** Watershed modeling and flow analysis
- **Ecology:** Habitat studies based on elevation
- **Natural hazards:** Flood zone mapping, landslide risk assessment
- **Urban planning:** Territorial planning considering relief
- **Tourism:** Hiking map creation
- **Telecommunications:** Line-of-sight analysis for antennas
- **Renewable energy:** Wind/solar potential based on slope orientation
- **Climate studies:** Microclimate analysis and temperature gradients
- **Agricultural planning:** Precision farming and terrain-based crop selection
- **Archaeological surveys:** Landscape analysis and site identification
- **Military applications:** Tactical terrain analysis
- **Infrastructure planning:** Road and pipeline route optimization

---

## 🌐 Adapting for Other Regions

While this guide uses French IGN data for Corsica, the code can be adapted for DEMs from other sources:

### Common DEM Sources Worldwide

1. **USGS (United States):** [Earth Explorer](https://earthexplorer.usgs.gov/)
   - SRTM (Shuttle Radar Topography Mission): 30m/90m resolution worldwide
   - ASTER GDEM: 30m resolution global coverage
   - National Elevation Dataset (NED): High-resolution US data

2. **Copernicus (European Union):** [Copernicus Open Access Hub](https://scihub.copernicus.eu/)
   - EU-DEM: 25m resolution for Europe

3. **OpenTopography:** [opentopography.org](https://opentopography.org/)
   - High-resolution LiDAR data for various regions

4. **National Mapping Agencies:**
   - UK: Ordnance Survey
   - Canada: Natural Resources Canada
   - Australia: Geoscience Australia
   - Japan: Geospatial Information Authority

### Adaptation Steps

1. **Check coordinate system:** Replace Lambert 93 (EPSG:2154) with your region's projection
2. **Verify file format:** Ensure ASCII Grid format or convert accordingly
3. **Update header parsing:** Some sources may have different header structures
4. **Adjust cellsize:** Adapt resampling factors based on original resolution
5. **Modify colorscale:** Choose elevation ranges appropriate for your terrain

### Example: Adapting for UTM Zone 10N (US West Coast)

```python
# Replace Lambert 93 with UTM Zone 10N
transformer = Transformer.from_crs("EPSG:32610", "EPSG:4326", always_xy=True)

# Update axis labels
xaxis_title="X (UTM Zone 10N) → East",
yaxis_title="Y (UTM Zone 10N) → North",
```

---

## 🧪 Advanced Features

### Custom Analysis Functions

```python
def calculate_slope(X, Y, Z, cellsize):
    """Calculate slope gradient in degrees."""
    dz_dx = np.gradient(Z, axis=1) / cellsize
    dz_dy = np.gradient(Z, axis=0) / cellsize
    slope = np.arctan(np.sqrt(dz_dx**2 + dz_dy**2)) * 180 / np.pi
    return slope

def calculate_aspect(X, Y, Z, cellsize):
    """Calculate aspect (compass direction of slope)."""
    dz_dx = np.gradient(Z, axis=1) / cellsize
    dz_dy = np.gradient(Z, axis=0) / cellsize
    aspect = np.arctan2(-dz_dy, dz_dx) * 180 / np.pi
    aspect = (90 - aspect) % 360  # Convert to compass bearing
    return aspect

def extract_contour_lines(X, Y, Z, levels):
    """Extract contour lines at specified elevation levels."""
    from matplotlib import pyplot as plt
    fig, ax = plt.subplots()
    contours = ax.contour(X, Y, Z, levels=levels)
    plt.close(fig)
    return contours
```

### Export Options

```python
def export_to_geotiff(X, Y, Z, output_path, epsg=2154):
    """Export to GeoTIFF format for GIS software."""
    from osgeo import gdal, osr
    
    driver = gdal.GetDriverByName('GTiff')
    rows, cols = Z.shape
    dataset = driver.Create(output_path, cols, rows, 1, gdal.GDT_Float32)
    
    # Set geotransform
    cellsize = X[0, 1] - X[0, 0]
    dataset.SetGeoTransform([
        X.min(), cellsize, 0,
        Y.max(), 0, -cellsize
    ])
    
    # Set projection
    srs = osr.SpatialReference()
    srs.ImportFromEPSG(epsg)
    dataset.SetProjection(srs.ExportToWkt())
    
    # Write data
    dataset.GetRasterBand(1).WriteArray(Z)
    dataset.FlushCache()
```

---

## 💡 Tips and Best Practices

### Memory Management
```python
# Process large datasets in chunks
def process_in_chunks(folders, chunk_size=5):
    for i in range(0, len(folders), chunk_size):
        chunk = folders[i:i+chunk_size]
        data = load_multiple_mnt_files(chunk)
        # Process chunk...
        del data  # Free memory
```

### Quality Control
```python
# Check for data gaps
def check_data_quality(Z):
    total_cells = Z.size
    valid_cells = np.sum(~np.isnan(Z))
    coverage = 100 * valid_cells / total_cells
    
    print(f"Data coverage: {coverage:.2f}%")
    
    if coverage < 90:
        print("⚠️ Warning: Low data coverage detected")
    
    return coverage
```

### Performance Profiling
```python
import time

def profile_function(func, *args, **kwargs):
    start = time.time()
    result = func(*args, **kwargs)
    elapsed = time.time() - start
    print(f"{func.__name__} took {elapsed:.2f} seconds")
    return result
```

---

## 🎓 Educational Use Cases

### Teaching Topics

1. **Geographic Information Systems (GIS)**
   - Coordinate system transformations
   - Spatial data structures
   - Raster analysis

2. **Computer Science**
   - Array manipulation and optimization
   - Data visualization techniques
   - Memory management

3. **Earth Sciences**
   - Topographic analysis
   - Terrain characterization
   - Digital elevation modeling

4. **Data Science**
   - Large dataset handling
   - Statistical resampling
   - Interactive visualization

### Student Projects

- Compare elevation profiles across different regions
- Analyze correlation between elevation and vegetation
- Model water flow and drainage patterns
- Create 3D-printed terrain models from DEM data
- Develop terrain-based video game levels
- Study climate patterns related to topography

---

## 🤝 Contributing

Suggestions for code improvements:
- Parallel processing for large datasets
- GPU acceleration for rendering
- Additional terrain analysis algorithms
- Support for more DEM formats
- Multi-language localization
- Interactive web interface

---

## 📞 Support and Community

For questions about:
- **IGN Data:** Contact IGN support via [geoservices.ign.fr](https://geoservices.ign.fr/)
- **Python Libraries:** Check respective documentation and GitHub issues
- **GIS Concepts:** Consult GIS.StackExchange community

---

**Version:** 1.0  
**Date:** December 2024  
**Author:** Neuroscience Researcher, C/C++/Python/Java Expert
