# Multithreading & Background Tasks in QGIS

When developing python scripts or plugins for QGIS, running heavy computations on the main application thread will cause the user interface (GUI) to freeze. You must distinguish between short and long processes and apply the correct threading model.

---

## 1. Short vs. Long Processes

### Short Processes (< 500ms)
- Examples: Selecting a few features, changing layer styles, fetching current project info.
- Workflow: Run synchronously directly on the main thread (e.g. from the QGIS Python Console).

### Long Processes (> 500ms)
- Examples: Downloading data from APIs, performing intensive spatial analysis, iterating through millions of geometries, converting file formats.
- Workflow: Must be offloaded to a background thread using `QgsTask` (QGIS Task Manager) or Qt-native classes (`QThread`, `QThreadPool`/`QRunnable`).

---

## 2. The QGIS Standard: `QgsTask`

`QgsTask` is the safest and most idiomatic way to handle multithreading in PyQGIS. Tasks integrate automatically with the QGIS Task Manager GUI (visible to users in the status bar).

### Method A: `QgsTask.fromFunction` (For simpler scripts)

This method offloads a python function to a background thread. You must check `task.isCanceled()` inside the function.

```python
import time
from qgis.core import QgsApplication, QgsTask, QgsMessageLog, Qgis

def long_calculation(task, delay):
    """
    Background function.
    IMPORTANT: First argument must be the task instance itself.
    """
    iterations = 100
    for i in range(iterations):
        # 1. ALWAYS check if the user canceled the task
        if task.isCanceled():
            return False
            
        time.sleep(delay)
        
        # 2. Update progress (0-100)
        task.setProgress((i / iterations) * 100)
        
    return True

def handle_finished(exception, result=None):
    """
    Callback function that runs on the GUI (main) thread when task is done.
    """
    if exception is not None:
        QgsMessageLog.logMessage(f"Task failed with error: {exception}", "MyScript", Qgis.Critical)
        return
        
    if result:
        QgsMessageLog.logMessage("Background task successfully finished!", "MyScript", Qgis.Info)
    else:
        QgsMessageLog.logMessage("Background task was canceled.", "MyScript", Qgis.Warning)

# Instantiate and run task
# QgsTask.fromFunction(description, function, *args, on_finished=callback, **kwargs)
task = QgsTask.fromFunction("Calculating Buffers", long_calculation, 0.05, on_finished=handle_finished)
QgsApplication.taskManager().addTask(task)
```

### Method B: Subclassing `QgsTask` (For complex plugins/state)

Subclassing `QgsTask` is the most powerful method. It allows you to maintain state and define clean logic.

```python
from qgis.core import QgsTask, QgsApplication, QgsMessageLog, Qgis

class DownloaderTask(QgsTask):
    def __init__(self, description, urls):
        # QgsTask.CanCancel flag allows user to cancel it in the GUI
        super().__init__(description, QgsTask.CanCancel)
        self.urls = urls
        self.downloaded_count = 0

    def run(self):
        """
        Runs in the background thread.
        Returns True on success, False on failure/cancel.
        """
        total = len(self.urls)
        for idx, url in enumerate(self.urls):
            if self.isCanceled():
                return False
            
            # Simulate network download
            import time
            time.sleep(1) 
            self.downloaded_count += 1
            
            # Update progress
            self.setProgress((idx + 1) / total * 100)
        return True

    def finished(self, result):
        """
        Runs automatically on the main (GUI) thread after run() completes.
        """
        if result:
            QgsMessageLog.logMessage(f"Successfully downloaded {self.downloaded_count} items.", "Downloader")
        else:
            QgsMessageLog.logMessage("Download failed or was canceled.", "Downloader", Qgis.Warning)

# Run the subclassed task
task = DownloaderTask("Downloading layers...", ["url1", "url2", "url3"])
QgsApplication.taskManager().addTask(task)
```

---

## 3. Qt Threading (`QThread` & `QThreadPool`)

If you are developing complex GUIs/dialogs or need lower-level concurrency, you can use Qt's native threading APIs.

### Thread Safety Warning
> [!CRITICAL]
> **GUI and QGIS Layer Access Rule:**
> Background threads (including `QThread` and `QRunnable`) **MUST NOT** modify or call UI elements, layer selections, or layer geometry/attribute modifications directly. Doing so will crash QGIS instantly. 
> 
> **Correct Pattern:** Run computations in the thread, emit results via a Qt Signal, and handle layer modifications or GUI updates in a slot connected to that signal on the Main Thread.

### Example: Custom `QThread` with Signals

```python
from qgis.PyQt.QtCore import QThread, pyqtSignal

class AnalysisWorker(QThread):
    # Signals must be class variables
    progress_updated = pyqtSignal(int)
    analysis_finished = pyqtSignal(dict)

    def __init__(self, data_list):
        super().__init__()
        self.data_list = data_list

    def run(self):
        """Heavy calculations without touching QGIS layers"""
        results = {}
        total = len(self.data_list)
        for idx, item in enumerate(self.data_list):
            # Process item...
            import time
            time.sleep(0.5)
            results[item] = len(item) # example result
            
            # Emit progress update
            self.progress_updated.emit(int((idx + 1) / total * 100))
            
        self.analysis_finished.emit(results)

# --- Main Thread usage ---
def update_layer_with_results(results):
    # Safe to edit layers on main thread
    print("Writing results back to layer:", results)

# Usage in a dialog or console script:
# worker = AnalysisWorker(["FeatureA", "FeatureB"])
# worker.analysis_finished.connect(update_layer_with_results)
# worker.start()
```

---

## 4. Production Flow Checklist

When moving QGIS code from prototyping to production:

1. **Phase 1: Prototyping (Console)**
   - Test small code blocks on the current active layer (`iface.activeLayer()`).
   - Run synchronously. Keep test datasets small.

2. **Phase 2: Custom Processing Algorithm (Standard Tools)**
   - Wrap the logic inside a `QgsProcessingAlgorithm` subclass.
   - *Why:* QGIS Processing automatically runs algorithms on background threads and manages layer loading/sinks for you, removing the need for manual `QgsTask` setup.

3. **Phase 3: GUI Integrations / Heavy API Plugins**
   - For custom plugins with dock widgets or dialogs that require network requests or custom background computation, subclass `QgsTask` or use `QThread`.
   - Implement slot connections to update UI components when results arrive.
