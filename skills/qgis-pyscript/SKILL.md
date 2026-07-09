name: qgis-pyscript
description: Guidelines and references for writing, debugging, and executing PyQGIS scripts and custom Processing Algorithms in QGIS, handling background threading and version requirements.
---

# QGIS Pyscript & Processing Tool Customization Skill

This skill triggers when you are asked to write or debug Python code for QGIS (PyQGIS) or create custom Processing Algorithms. It guides you to write code that adheres strictly to the QGIS API and standard workflows.

## Key Rules & Guidelines

1. **User Requirement & Version Checks (Mandatory First Step)**:
   - **Before writing any code**, you must ask the user:
     - Which QGIS version they are targetting (e.g., QGIS 3.28 LTR, QGIS 3.34, or older QGIS 2.x since APIs differ significantly).
     - The exact workflow, inputs, and desired outputs of the script/tool.

2. **Adhere to Official PyQGIS APIs**:
   - Always reference the official QGIS API docs or local references for class names and method signatures.
   - Use standard names for core imports (e.g., `from qgis.core import QgsProject, QgsVectorLayer, QgsGeometry`).

3. **QGIS Processing Algorithm Rules**:
   - Custom Processing tools must subclass `QgsProcessingAlgorithm`.
   - Never write processing tools as ad-hoc scripts; follow the standard skeleton: `initAlgorithm()`, `processAlgorithm()`, `name()`, `displayName()`, `createInstance()`.
   - Utilize translation wrappers: `self.tr()` with `QCoreApplication.translate()`.
   - Read the local reference: [processing_algorithm.md](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/processing_algorithm.md).

4. **Data Access & Geometry Safety**:
   - When modifying layers, wrap operations in `startEditing()` and `commitChanges()` or use an edit buffer context manager where available.
   - For batch edits or performance-sensitive tasks, use `QgsVectorLayerEditBuffer` or layer data provider directly.
   - Read the local reference: [pyqgis_basics.md](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/pyqgis_basics.md).

5. **Threading & Performance (Long vs. Short Processes)**:
   - **Short Processes**: Run synchronously on the main thread if they take under a few hundred milliseconds.
   - **Long-Running Processes**: Offload to background threads using `QgsTask` (`QgsTask.fromFunction` or custom subclasses), `QThread`, or `QThreadPool` / `QRunnable` to prevent UI freezing.
   - **Rule**: Never update QGIS GUI elements directly from background threads. Use signals/slots to update GUI elements on the main thread safely.
   - Read the local reference: [multithreading.md](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/multithreading.md).

6. **UI & User Feedback**:
   - Within Processing algorithms, use the `feedback` parameter (e.g., `feedback.setProgress()`, `feedback.pushInfo()`) for logging and status updates. Never use plain `print()` statements.
   - Check `feedback.isCanceled()` in loops to allow users to cancel long-running processes gracefully.

## Reference Documentation

- [PyQGIS Basics & Vector/Raster Layer Manipulation](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/pyqgis_basics.md)
- [Writing Custom QGIS Processing Algorithms](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/processing_algorithm.md)
- [Multithreading & Background Tasks (QgsTask, QThread, QThreadPool)](file:///c:/Users/Admin/Documents/Development/skills/skills/qgis-pyscript/references/multithreading.md)

