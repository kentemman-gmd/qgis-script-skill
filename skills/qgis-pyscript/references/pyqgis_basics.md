# PyQGIS Basics Reference

This document covers common PyQGIS operations for layer manipulation, feature extraction, geometry modification, and transaction safety.

## 1. Project and Layer Management

Accessing the active project instance and loading layers:

```python
from qgis.core import QgsProject, QgsVectorLayer, QgsRasterLayer

# Get the current project
project = QgsProject.instance()

# Load a Vector Layer
# QgsVectorLayer(path, baseName, providerLib)
vector_layer = QgsVectorLayer("C:/data/roads.shp", "Roads Layer", "ogr")
if not vector_layer.isValid():
    print("Layer failed to load!")
else:
    project.addMapLayer(vector_layer)

# Load a Raster Layer
raster_layer = QgsRasterLayer("C:/data/dem.tif", "DEM Layer")
if not raster_layer.isValid():
    print("Raster failed to load!")
else:
    project.addMapLayer(raster_layer)
```

## 2. Accessing & Iterating Features

Always specify fields or geometry constraints if you do not need all of them, for maximum performance.

```python
from qgis.core import QgsFeatureRequest

layer = QgsProject.instance().mapLayersByName("Roads Layer")[0]

# Simple iteration
for feature in layer.getFeatures():
    # Access attributes by name or index
    name = feature['name']
    road_type = feature['type']
    geom = feature.geometry()
    
# Iterating with attribute constraints (Faster)
request = QgsFeatureRequest().setFilterExpression('"type" = \'highway\'')
for feature in layer.getFeatures(request):
    print(feature['name'])
```

## 3. Editing Layers Safely

When modifying geometries or attributes, always wrap the operations inside `startEditing()` and `commitChanges()`.

```python
layer = QgsProject.instance().mapLayersByName("Roads Layer")[0]

# Start editing mode
layer.startEditing()

try:
    for feature in layer.getFeatures():
        if feature['name'] == 'Old Highway Name':
            # Update attributes
            layer.changeAttributeValue(feature.id(), layer.fields().indexOf('name'), 'New Highway Name')
            
            # Update geometry
            # layer.changeGeometry(feature.id(), new_geometry)
            
    # Commit changes to source
    layer.commitChanges()
except Exception as e:
    # Rollback changes if anything goes wrong
    layer.rollBack()
    raise e
```

## 4. Creating New Features

Adding features to an existing layer:

```python
from qgis.core import QgsFeature, QgsGeometry, QgsPointXY

layer = QgsProject.instance().mapLayersByName("Points Layer")[0]

layer.startEditing()

# Create feature
feat = QgsFeature(layer.fields())
feat.setGeometry(QgsGeometry.fromPointXY(QgsPointXY(120.5, 14.2)))
feat.setAttribute('name', 'Sample Site')
feat.setAttribute('value', 42.5)

# Add feature to layer
success, new_features = layer.addFeature(feat)
if success:
    layer.commitChanges()
else:
    layer.rollBack()
```

## 5. Using Spatial Index

For spatial search queries (e.g., finding the nearest feature), use a `QgsSpatialIndex` for high performance.

```python
from qgis.core import QgsSpatialIndex, QgsPointXY

layer = QgsProject.instance().mapLayersByName("Points Layer")[0]

# Build index
index = QgsSpatialIndex(layer.getFeatures())

# Find nearest 3 features to a coordinate point
search_point = QgsPointXY(120.5, 14.2)
nearest_ids = index.nearestNeighbor(search_point, neighbors=3)
```
