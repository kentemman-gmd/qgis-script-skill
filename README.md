# QGIS Python & Processing Tool Agent Skill

[![skills.sh](https://skills.sh/b/vercel-labs/agent-skills)](https://skills.sh) <!-- Replace vercel-labs/agent-skills with your-username/your-repo-name once public -->

A custom agent skill for writing, formatting, and debugging QGIS Python scripts (PyQGIS) and custom Processing Algorithms. This skill instructs coding agents (such as Antigravity, Claude Code, Cursor, etc.) to always adhere strictly to official QGIS APIs, follow thread-safety conventions (using `QgsTask` or Qt threading), and structure Processing Tools properly.

## Recommended Repository Names
We recommend using one of the following names when creating your public GitHub repository:
- `qgis-agent-skills` (Highly recommended)
- `qgis-pyscript-skill`
- `agent-skills-qgis`

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
            ├── pyqgis_basics.md   # Reference for layer loading & feature edits
            ├── processing_algorithm.md # Reference for custom Processing tools
            └── multithreading.md  # Reference for QgsTask & background operations
```

---

## How to Install and Register

### Method A: Install via skills.sh (Global / Public)
Once this repository is public, you and other users can install this skill globally or inside any project workspace using the `skills` CLI:

```bash
npx skills add <your-github-username>/<your-repo-name>
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

---

## How to Use the Skill

When this skill is loaded, your AI coding assistant will automatically trigger it whenever you ask QGIS-related python tasks. The agent will:

1. **Ask for QGIS version & script goals**: Before writing any code, it will confirm your target environment (e.g. QGIS 3.28 LTR) and clear workflows.
2. **Follow safe geometry & attribute transactions**: Wrapping changes in `startEditing()` and `commitChanges()`.
3. **Use QGIS background tasks**: Directing heavy tasks to `QgsTask` or thread pools to avoid locking up the QGIS user interface.
4. **Build Processing skeleton classes**: Structuring tools by subclassing `QgsProcessingAlgorithm`.

---

## References Documentation

- [PyQGIS Basics](skills/qgis-pyscript/references/pyqgis_basics.md): Guides on map layers, features, and spatial indexes.
- [Processing Algorithms](skills/qgis-pyscript/references/processing_algorithm.md): Guides on inputs, outputs, sinks, and translation wrappers.
- [Multithreading & Background Tasks](skills/qgis-pyscript/references/multithreading.md): Guides on using QgsTask, QThread, and thread safety.
