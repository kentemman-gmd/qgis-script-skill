# QGIS Architecture Standards

Before generating code, a Senior QGIS Developer must determine the most appropriate QGIS integration pattern based on user requirements.

## 1. Processing Algorithm (`QgsProcessingAlgorithm`)

**Use when:**
- Spatial analysis, data transformation, batch operations, reusable workflows.
- Example: Buffer generation, overlap detection, clipping workflows.
- Location: Processing Toolbox.

*Rules:* Never use GUI dialogs inside. Support headless execution. Return outputs via `QgsFeatureSink`.

## 2. Processing Provider (`QgsProcessingProvider`)

**Use when:**
- Multiple related algorithms exist and require grouping.
- A toolbox category is required.
- Example: "Validation Tools" grouping, "Data Preparation Tools".

*Structure:*
```text
processing_provider/
├── provider.py
├── algorithms/
│   ├── validate_geometry_alg.py
│   └── detect_overlap_alg.py
└── icons/
```

## 3. Toolbar Action

**Use when:**
- A quick utility is required.
- The action is simple and the user expects a one-click operation.
- Example: Refresh layer, open report, zoom utility.
- *Rule:* Toolbar actions should NOT replace Processing algorithms. If it modifies geometries heavily, it should be an algorithm.

## 4. Dock Widget

**Use when:**
- Persistent interaction is required.
- Users manage workflows or monitor information.
- Multiple actions exist in a single interface (e.g., QA Dashboard, Dataset Manager).
- *Preferred Pattern:* Toolbar Action -> Open Dock Widget.

## 5. Hybrid Plugin

**Use when:**
- Processing and UI must work together.
- Users manage workflows (via Dock Widget) and run heavy analysis (via Processing algorithms).
- *Pattern:* Toolbar -> Dock Widget -> Calls Processing Algorithms headless.

*Scalable Architecture Structure:*
```text
hybrid_plugin_name/
├── __init__.py
├── metadata.txt
├── plugin.py
├── gui/
│   ├── dialogs/
│   ├── dock_widgets/
│   └── widgets/
├── processing_provider/
│   ├── provider.py
│   └── algorithms/
├── core/
│   ├── services/
│   ├── models/
│   └── validators/
└── tests/
```
