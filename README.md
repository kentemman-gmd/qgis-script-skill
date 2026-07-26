# QGIS Python & Processing Tool Agent Skill

[![skills.sh](https://skills.sh/b/kentemman-gmd/qgis-script-skill)](https://skills.sh)

A custom agent skill for writing, formatting, and debugging QGIS Python scripts (PyQGIS) and custom Processing Algorithms. This skill instructs coding agents (such as Antigravity, Claude Code, Cursor, etc.) to act as a **Senior QGIS Developer**. It enforces strict adherence to official QGIS APIs through a **Documentation First Policy**, ensures thread-safety conventions (using `QgsTask` or Qt threading), mandates proper Processing Tool structuring, prioritizes native QGIS Qt widgets over custom UIs, and guarantees rigorous validation, testing, and architectural planning before any code is generated.

---

## Directory Structure

```text
├── .agents/
│   └── skills.json                # Local workspace skill registration config
├── skills.sh.json                 # skills.sh page customization config
├── README.md                      # This documentation file
└── skills/
    └── qgis-pyscript/
        ├── SKILL.md               # Core skill rules and guidelines
        └── references/
            ├── architecture_patterns.md  # Reference for Architecture Patterns & Plugin Structure
            ├── coding_standards.md       # Reference for Coding, Naming & Logging Standards
            ├── multithreading.md         # Reference for QgsTask & background operations
            ├── official_resources.md     # Important links to PyQGIS API and Cookbook docs
            ├── processing_algorithm.md   # Reference for custom Processing tools
            ├── pyqgis_basics.md          # Reference for layer loading & feature edits
            └── validation_testing.md     # Reference for geometry validation, CRS handling & testing
```

---

## How to Use the Skill

When this skill is loaded, your AI coding assistant will automatically trigger it whenever you ask QGIS-related python tasks. The agent will act as a Senior QGIS Developer and enforce a strict **9-step development workflow** for every request to prioritize accuracy, reliability, and maintainability:

1. **Requirement Analysis**: Analyze the requirements and verify all assumptions before proceeding.
2. **Architecture Selection**: Determine the right plugin UI/Processing patterns and project structure.
3. **Implementation Plan**: Outline the step-by-step logic and PyQGIS APIs to be used.
4. **Risk Assessment**: Identify potential issues like silent reprojections, UI locking, or missing validation.
5. **Validation Strategy**: Detail how inputs (geometry, CRS, fields) will be validated.
6. **Test Plan**: Generate tests for success cases, empty layers, invalid geometries, missing fields, etc.
7. **Implementation**: Produce the code, adhering to strict coding and logging standards.
8. **Validation Review**: Review the generated code to ensure it meets the strategy.
9. **Self Review**: Perform a final architectural and quality check.

The agent will **not generate code immediately**. It will plan, ask clarifying questions when uncertain, verify assumptions against official documentation, and provide thoroughly validated solutions that wrap changes in `startEditing()` and `commitChanges()`, process data headlessly in `QgsProcessingAlgorithm`, and delegate heavy operations to `QgsTask` or thread pools. For UI, it will always prioritize built-in QGIS Qt widgets.

---

## References Documentation

- [PyQGIS Basics](skills/qgis-pyscript/references/pyqgis_basics.md): Guides on map layers, features, and spatial indexes.
- [Processing Algorithms](skills/qgis-pyscript/references/processing_algorithm.md): Guides on inputs, outputs, sinks, and translation wrappers.
- [Multithreading & Background Tasks](skills/qgis-pyscript/references/multithreading.md): Guides on using QgsTask, QThread, and thread safety.
- [Architecture Patterns](skills/qgis-pyscript/references/architecture_patterns.md): Guides on selecting the right plugin UI/Processing patterns.
- [Validation & Testing](skills/qgis-pyscript/references/validation_testing.md): Guides on geometry validation, CRS handling, and writing proper test plans.
- [Coding & Naming Standards](skills/qgis-pyscript/references/coding_standards.md): Guides on strict coding, structural, and logging standards.
- [Official PyQGIS Resources](skills/qgis-pyscript/references/official_resources.md): Important links to PyQGIS API and Cookbook docs.

## How to Install and Register

### Method A: Install via skills.sh (Global / Public)
Once this repository is public, you and other users can install this skill globally or inside any project workspace using the `skills` CLI:

```bash
npx skills add kentemman-gmd/qgis-script-skill
```

### Method B: Manual Local Setup
If you want to use this skill locally without publishing it:
1. Clone this repository into your workspace.
2. In your workspace's customization root (usually a `.agents` folder at the root of your project), create a `skills.json` file.
3. Reference the path of the cloned `skills` directory:
```json
   {
     "entries": [
       { "path": "path/to/cloned/repo/skills" }
     ]
   }
```


