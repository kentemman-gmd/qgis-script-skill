# QGIS Coding & Architecture Standards

This document outlines the strict coding, naming, and structural standards for PyQGIS development.

## 1. PyQGIS Coding Standards

**Always:**
*   Use type hints for function arguments and return types.
*   Use standard Python docstrings for all classes and functions.
*   Use descriptive variable names (e.g., `buffer_distance` instead of `d`).
*   Use structured logging (via the `logging` module) rather than standard `print()` statements in production code.
*   Handle exceptions explicitly (e.g., `try...except QgsProcessingException`).
*   Validate all inputs, assumptions, and CRS compatibility before processing.
*   Use comments only where business logic is complex; prefer self-documenting code.
*   Use simple, professional text for comments.

**Avoid:**
*   Magic values (use constants at the top of the file or class).
*   Silent failures (always log or raise exceptions).
*   Excessive globals.
*   Unnecessary complexity.
*   Emojis in source code (comments, docstrings, log messages).
*   Emojis in UI elements (window titles, labels, buttons, tooltips). Use standard QGIS/Qt icons (`QIcon`) instead.

## 2. Naming Standards

*   **Files / Modules:** Use `snake_case.py` (e.g., `detect_polygon_overlaps.py`, `split_gpkg_by_region.py`). Avoid `MyTool.py` or `ToolV2.py`.
*   **Processing Algorithm IDs:** Use lowercase strings with no spaces or special characters except underscores (e.g., `detect_overlaps`).
*   **Classes:** Use `PascalCase` (e.g., `DetectOverlapsAlgorithm`).
*   **Functions / Variables:** Use `snake_case` (e.g., `calculate_centroid()`).

## 3. Plugin Architecture Standards

Avoid flat architectures for large plugins. Separate UI, Business Logic, Processing, Validation, and Testing into distinct packages.

Use scalable plugin architecture:

```text
plugin_name/
├── __init__.py
├── metadata.txt
├── icon.png                  # Mandatory plugin icon
├── plugin.py
├── resources.qrc
├── resources.py
├── gui/
│   ├── dialogs/
│   ├── dock_widgets/
│   ├── widgets/
│   └── ui/
├── processing_provider/
│   ├── provider.py
│   └── algorithms/
├── core/
│   ├── services/
│   ├── models/
│   ├── validators/
│   ├── repositories/
│   └── workflows/
├── docs/
└── tests/
```

## 4. Automated Icon Generation Protocol (Console, Plugin, Processing Tool & Hybrid)

When creating a QGIS Plugin, Processing Algorithm, Hybrid Plugin, or Console Script with custom toolbar buttons, **always generate and configure a custom transparent PNG icon automatically**:

1. **Transparent Image Generation Prompting:**
   Invoke `generate_image` tool with explicit transparent background requirements:
   > *"A modern, minimalist GIS vector icon for a QGIS [Plugin/Processing Tool/Console Script] named [Name]. Features [key GIS symbol] on a **completely transparent background (PNG format)** with crisp outlines, high-contrast colors, and no solid background square/circle. Must render clearly on both light and dark QGIS toolbar themes."*
2. **File Storage & Asset Management:**
   Save/copy the generated transparent image to `icon.png` inside the root directory of the plugin (`plugin_folder/icon.png`) or script workspace directory.
3. **Manifest Integration (`metadata.txt` for Plugins & Hybrid):**
   Include `icon=icon.png` in the `[general]` section of `metadata.txt`.
4. **Dynamic Icon Loading in Python:**
   Always load `icon.png` using dynamic path resolution with fallback:
   ```python
   import os
   from qgis.PyQt.QtGui import QIcon

   icon_path = os.path.join(os.path.dirname(__file__), "icon.png")
   icon = QIcon(icon_path) if os.path.exists(icon_path) else QIcon(":/images/themes/default/mActionBuffer.svg")
   ```

## 5. Lifecycle & Deployment Guidelines by Deliverable Type

### A. Standalone PyQGIS Console Script
- **Target Environment:** Designed for execution directly in the **QGIS Python Console Editor** tab (e.g. `script_name.py`).
- **Global Context:** `iface` and `QgsProject.instance()` are available directly.
- **User Feedback:** Use non-blocking message bar: `iface.messageBar().pushMessage("Title", "Message", level=Qgis.Info)`.
- **Toolbar / Action Cleanup:** If adding a custom action or button to `iface` from the console, assign a unique `objectName` or remove previous instances before re-adding to prevent duplicate buttons when the user re-runs the script tab.

### B. Full QGIS Plugin & Hybrid Plugin
- **Target Environment:** Installed package in QGIS profile directory (`python/plugins/`).
- **Symmetrical Menu Cleanup:** Every `addAction` or `insertMenu` created in `initGui()` **must** have a corresponding cleanup (`removeAction`, `removePluginMenu`, `deleteLater()`, etc.) in `unload()`.
- **Hybrid Provider Lifecycle:** For Hybrid Plugins, register `QgsProcessingProvider` in `initGui()` via `QgsApplication.processingRegistry().addProvider()` and unregister in `unload()`.
- **Python Module Cache Flush Warning:** Always inform the user that editing `.py` files in QGIS requires restarting QGIS or using the **Plugin Reloader** plugin to flush Python memory cache (`sys.modules`).

### C. Custom Processing Tool (`QgsProcessingAlgorithm`)
- **Target Environment:** Headless execution inside QGIS Processing Toolbox / Modeler.
- **Headless Rule:** Never call `iface`, `QMessageBox`, or parent UI widgets inside `processAlgorithm()`.
- **Feedback & Logging:** Use `QgsProcessingFeedback.pushInfo()` and `reportError()`.
- **Toolbox Icon:** Implement `icon()` method in algorithm class returning `QIcon(icon_path)` or SVG fallback.

### D. Standalone Headless PyQGIS Script
- **Target Environment:** Standalone CLI script executed outside QGIS desktop app (OS command line, scheduled cron jobs, server pipelines, Docker containers, or via `qgis_process` CLI).
- **Initialization & Teardown Protocol:**
  ```python
  from qgis.core import QgsApplication

  # Initialize QGIS Application headlessly without GUI
  QgsApplication.setPrefixPath("/path/to/qgis", True)
  qgs = QgsApplication([], False)  # Second parameter False disables GUI
  qgs.initQgis()

  # ... Perform layer processing / PyQGIS analysis ...

  # Clean teardown
  qgs.exitQgis()
  ```
- **Strict Rules:** Zero `iface` references (`iface` is `None`). No Qt GUI widgets or modal dialog dependencies. Uses standard Python `logging` or `print()` for output.

## 6. Logging Standards

Use structured logging to facilitate debugging.

Levels:
*   **DEBUG**: Detailed information, typically of interest only when diagnosing problems.
*   **INFO**: Confirmation that things are working as expected.
*   **WARNING**: An indication that something unexpected happened, but the software is still working.
*   **ERROR**: Due to a more serious problem, the software has not been able to perform some function.

Do not use random `print()` statements in final plugin or algorithm code. For Processing Algorithms, prefer `QgsProcessingFeedback.pushInfo()` and `QgsProcessingFeedback.reportError()`.

## 7. Documentation Requirements

For substantial implementations, always provide:
*   **Purpose**: What the script/tool solves.
*   **Inputs**: Required layers, types, fields, or parameters.
*   **Outputs**: Expected results or modified layers.
*   **Dependencies**: Required libraries or specific QGIS versions.
*   **Usage Notes**: How a user should interact with the tool.
*   **Limitations**: Known constraints (e.g., "Fails on datasets over 1M features").
*   **Performance Notes**: Threading assumptions or indexing strategies.
