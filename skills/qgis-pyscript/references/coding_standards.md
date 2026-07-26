# QGIS Coding & Architecture Standards

This document outlines the strict coding, naming, and structural standards for PyQGIS development.

## 1. PyQGIS Coding Standards

**Always:**
*   **Adapt PyQt Imports:** Dynamically adapt `PyQt5` or `PyQt6` imports based on the user's specified QGIS version (QGIS 3.x uses PyQt5; QGIS 3.40+ uses PyQt6). Example: `try: from PyQt6.QtWidgets import ... except ImportError: from PyQt5.QtWidgets import ...` (or use `qgis.PyQt` shim if available).
*   **Prefer Native QGIS Widgets:** Always use `qgis.gui` widgets (e.g., `QgsMapLayerComboBox`) over standard Qt widgets for GIS inputs.
*   **Adhere to UI Preferences:** Only generate `.ui` files or programmatic Python UI layouts based explicitly on the user's preference. Apply custom CSS/Qt Stylesheets only if requested.
*   **License Compliance:** If integrating third-party plugin code or algorithms, retain their original license headers and document the source clearly in docstrings.
*   Use type hints for function arguments and return types.
*   Use standard Python docstrings for all classes and functions.
*   Use descriptive variable names (e.g., `buffer_distance` instead of `d`).
*   Use structured logging (via the `logging` module) rather than standard `print()` statements in production code.
*   Handle exceptions explicitly (e.g., `try...except QgsProcessingException`).
*   Validate all inputs, assumptions, and CRS compatibility before processing.
*   Use comments only where business logic is complex; prefer self-documenting code.

**Avoid:**
*   Hardcoding PyQt5 or PyQt6 imports without checking the target QGIS version.
*   Reinventing GIS UI components (e.g., building a custom layer dropdown instead of using `QgsMapLayerComboBox`).
*   Magic values (use constants at the top of the file or class).
*   Silent failures (always log or raise exceptions).
*   Excessive globals.
*   Unnecessary complexity.

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

## 4. Logging Standards

Use structured logging to facilitate debugging.

Levels:
*   **DEBUG**: Detailed information, typically of interest only when diagnosing problems.
*   **INFO**: Confirmation that things are working as expected.
*   **WARNING**: An indication that something unexpected happened, but the software is still working.
*   **ERROR**: Due to a more serious problem, the software has not been able to perform some function.

Do not use random `print()` statements in final plugin or algorithm code. For Processing Algorithms, prefer `QgsProcessingFeedback.pushInfo()` and `QgsProcessingFeedback.reportError()`.

## 5. Documentation Requirements

For substantial implementations, always provide:
*   **Purpose**: What the script/tool solves.
*   **Inputs**: Required layers, types, fields, or parameters.
*   **Outputs**: Expected results or modified layers.
*   **Dependencies**: Required libraries or specific QGIS versions.
*   **Usage Notes**: How a user should interact with the tool.
*   **Limitations**: Known constraints (e.g., "Fails on datasets over 1M features").
*   **Performance Notes**: Threading assumptions or indexing strategies.
