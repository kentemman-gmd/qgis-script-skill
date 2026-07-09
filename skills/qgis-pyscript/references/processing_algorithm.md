# Writing QGIS Custom Processing Algorithms

This document provides a template and explanation for creating custom geoprocessing tools in QGIS that integrate directly into the **Processing Toolbox**.

## 1. Processing Algorithm Template

Every custom processing tool must inherit from `QgsProcessingAlgorithm`. Below is the standard, production-ready skeleton for a Processing algorithm.

```python
from qgis.PyQt.QtCore import QCoreApplication
from qgis.core import (
    QgsProcessing,
    QgsProcessingAlgorithm,
    QgsProcessingException,
    QgsProcessingParameterFeatureSource,
    QgsProcessingParameterFeatureSink,
    QgsFeature,
    QgsFeatureSink
)

class BufferPointsAlgorithm(QgsProcessingAlgorithm):
    """
    A custom QGIS Processing Algorithm that buffers point layers.
    """
    
    # Define parameter name constants
    INPUT = 'INPUT'
    OUTPUT = 'OUTPUT'

    def tr(self, text):
        """
        Helper method to return translated strings.
        """
        return QCoreApplication.translate('Processing', text)

    def createInstance(self):
        """
        Must return a new instance of this algorithm class.
        """
        return BufferPointsAlgorithm()

    def name(self):
        """
        The unique ID of the algorithm (lowercase, no spaces).
        """
        return 'bufferpoints'

    def displayName(self):
        """
        The user-visible name shown in the Toolbox interface.
        """
        return self.tr('Buffer Points Algorithm')

    def group(self):
        """
        The group name containing this algorithm.
        """
        return self.tr('Vector Utilities')

    def groupId(self):
        """
        The unique ID of the group containing this algorithm.
        """
        return 'vectorutilities'

    def shortHelpString(self):
        """
        A description of the algorithm to display in the help panel.
        """
        return self.tr("This algorithm buffers features of an input vector layer.")

    def initAlgorithm(self, config=None):
        """
        Define input parameters and output destination sinks.
        """
        # Add Input layer parameter
        self.addParameter(
            QgsProcessingParameterFeatureSource(
                self.INPUT,
                self.tr('Input layer'),
                [QgsProcessing.TypeVectorPoint] # Restrict to points
            )
        )

        # Add Output destination layer parameter
        self.addParameter(
            QgsProcessingParameterFeatureSink(
                self.OUTPUT,
                self.tr('Buffered output')
            )
        )

    def processAlgorithm(self, parameters, context, feedback):
        """
        The core geoprocessing logic.
        """
        # Retrieve input layer as a Source
        source = self.parameterAsSource(parameters, self.INPUT, context)
        if source is None:
            raise QgsProcessingException(self.tr("Invalid input layer source."))

        # Initialize output Destination Sink
        (sink, dest_id) = self.parameterAsSink(
            parameters,
            self.OUTPUT,
            context,
            source.fields(),
            source.wkbType(),
            source.crs()
        )
        if sink is None:
            raise QgsProcessingException(self.tr("Invalid output destination sink."))

        # Setup progress reporting
        total = 100.0 / source.featureCount() if source.featureCount() else 0
        features = source.getFeatures()

        for current, feature in enumerate(features):
            # Check for user cancellation
            if feedback.isCanceled():
                break

            # Process features (e.g., clone feature and process geometry)
            out_feature = QgsFeature(feature)
            
            # Write to output sink
            sink.addFeature(out_feature, QgsFeatureSink.FastInsert)

            # Update progress bar
            feedback.setProgress(int(current * total))

        return {self.OUTPUT: dest_id}
```

## 2. Common Parameter Types

| Parameter Class | Purpose | Example |
|---|---|---|
| `QgsProcessingParameterFeatureSource` | Vector layer input | Select a vector layer |
| `QgsProcessingParameterRasterLayer` | Raster layer input | Select a DEM or imagery |
| `QgsProcessingParameterDistance` | Distance values (respects map units) | Buffer size, search distance |
| `QgsProcessingParameterNumber` | Integer/float value inputs | Max iterations, threshold value |
| `QgsProcessingParameterString` | General text inputs | Attribute name, SQL queries |
| `QgsProcessingParameterBoolean` | Checkbox toggles | Clip bounds (True/False) |
| `QgsProcessingParameterFile` | Path to an external file | Log file export, config JSON |
| `QgsProcessingParameterFeatureSink` | Output vector layer path | Destination file or temp layer |

## 3. Best Practices

- **Never modify input layers directly**: Always stream features into a `QgsFeatureSink` (output sink).
- **Check for cancellations**: In loops, checking `feedback.isCanceled()` is critical to prevent QGIS UI lockups.
- **Translate strings**: Use the translation helper `self.tr()` for parameter descriptions, algorithm display names, and exceptions.
- **Log messages correctly**: Instead of `print()`, use `feedback.pushInfo("Message")` or `feedback.reportError("Error detail")`.
