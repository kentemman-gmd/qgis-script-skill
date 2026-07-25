# Official QGIS Resources & Documentation

As a Senior QGIS Developer, you must adhere strictly to the official APIs and established practices. Do not invent QGIS classes, methods, or APIs. When uncertain, consult these official resources before generating an implementation plan.

## Core API Documentation

*   **QGIS Python API (PyQGIS) Documentation**:
    [https://qgis.org/pyqgis/master/](https://qgis.org/pyqgis/master/)
    *The definitive source for Python bindings. Use this to verify class names, method signatures, and parameter requirements.*

*   **QGIS C++ API Documentation**:
    [https://qgis.org/api/](https://qgis.org/api/)
    *While you are writing Python, the underlying architecture is C++. The C++ API documentation is often more detailed and explains core framework behaviors.*

## Developer Guides & Cookbooks

*   **PyQGIS Developer Cookbook**:
    [https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/](https://docs.qgis.org/latest/en/docs/pyqgis_developer_cookbook/)
    *The official guide for common tasks like loading layers, iterating features, writing Processing algorithms, and creating plugins. This is your primary reference for standard implementation patterns.*

*   **QGIS Developers Guide**:
    [https://docs.qgis.org/latest/en/docs/developers_guide/](https://docs.qgis.org/latest/en/docs/developers_guide/)
    *Covers overarching development topics such as Qt threading models, plugin repositories, and coding standards.*

## Source Code & Framework References

*   **QGIS GitHub Repository**:
    [https://github.com/qgis/QGIS](https://github.com/qgis/QGIS)
    *When documentation is sparse, search the core repository for usage examples in the C++ or Python source files.*

## QGIS Version Awareness

Always confirm the target QGIS version with the user (e.g., QGIS 3.28 LTR, QGIS 3.34). Be aware that certain methods (especially related to `QgsProcessing` and `QgsTask`) may have different signatures or availability between versions. The URLs above generally point to the latest/master docs; adjust expectations if the user is targeting an older LTR.