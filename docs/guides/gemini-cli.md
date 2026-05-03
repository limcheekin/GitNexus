# Integrating GitNexus with Gemini CLI

This guide explains how to install **GitNexus** and integrate it with **Gemini CLI** using the Model Context Protocol (MCP). This setup provides Gemini with high-fidelity "code intelligence," allowing it to perform graph-based searches, impact analysis, and execution flow tracing.

---

## 📋 Prerequisites

- **Node.js**: `v20.0.0` or higher.
- **Python 3**: Required for tree-sitter native bindings.
- **Build Tools**: `make` and `g++` (Linux/macOS) or Visual Studio Build Tools (Windows).
- **Disk Space**: ~200MB free for the initial embedding model download.

---

## 🚀 1. Installation & Global Link

Instead of running GitNexus by its file path, use `npm link` to make it a first-class command on your system.

1.  **Clone and Build**:
    ```bash
    git clone https://github.com/abhigyanpatwari/GitNexus.git
    cd GitNexus/gitnexus
    npm install
    npm run build
    ```

2.  **Global Link**:
    ```bash
    # This makes the 'gitnexus' command available globally
    sudo npm link
    ```
    *Note: Windows users should run their terminal (PowerShell/CMD) as **Administrator** for this step.*

---

## 🩺 2. Health Check (Doctor)

GitNexus relies on native C/C++ bindings. Verify your environment is set up correctly:

```bash
gitnexus doctor
```
*If this fails, ensure your Python and C++ build tools are correctly installed and in your PATH.*

---

## 🔍 3. Indexing Your Project

Before Gemini can "understand" a codebase, GitNexus must index it.

1.  **Navigate to your project root**:
    ```bash
    cd /path/to/your/project
    ```

2.  **Run Analysis**:
    ```bash
    gitnexus analyze --embeddings
    ```
    - **Model Download**: The first time you run `--embeddings`, GitNexus will download a ~100MB model (`arctic-embed-xs`) to your local cache.
    - **Results**: This creates a `.gitnexus/` folder and registers the project in the global registry (`~/.gitnexus/registry.json`).

---

## 🛠️ 4. Gemini CLI Integration

Gemini CLI provides a built-in command to manage MCP servers. This is safer and easier than manual JSON editing.

**Run this command**:
```bash
gemini mcp add gitnexus "gitnexus" mcp --trust
```

- **`"gitnexus"`**: The name of the server in Gemini.
- **`mcp`**: The argument passed to the GitNexus command to start the protocol.
- **`--trust`**: Marks the server as trusted so Gemini won't ask for permission before every tool call.

---

## 📦 5. Advanced: Cross-Repository Intelligence (Groups)

If your project is split across multiple repositories (e.g., `api-server` and `web-client`), GitNexus can bridge them so Gemini can trace dependencies across the boundary.

1.  **Index both repos** individually using `gitnexus analyze`.
2.  **Create a group**:
    ```bash
    gitnexus group create my-project
    gitnexus group add my-project backend api-server
    gitnexus group add my-project frontend web-client
    gitnexus group sync my-project
    ```
    *Note: Use the names assigned during `analyze` (default is the folder name).*
3.  **Update Gemini**: (If you haven't already added the server)
    ```bash
    gemini mcp add gitnexus "gitnexus" mcp --trust
    ```
*Gemini will now be able to trace a change in the Backend API all the way to the Frontend consumer.*

---

## 💡 6. Practical Usage in Gemini

Once configured, restart your Gemini CLI session. You can now use GitNexus intelligence directly in your conversation.

| Goal | Example Prompt |
| :--- | :--- |
| **Concept Discovery** | "Find all code responsible for handling file uploads." |
| **Dependency Mapping** | "Give me the context for the `validateUser` function." |
| **Risk Assessment** | "What would break if I change the return type of `getProjectRoot`?" |
| **Code Review** | "Analyze my staged changes and find affected execution flows." |
| **Deep Search** | "Run a Cypher query to find all classes that implement `IStorage`." |

---

## ⚡ 7. Pro-Tips for Speed & Stability

-   **Skip Large Files**: If your repo has massive generated files, use `gitnexus analyze --max-file-size 1024` (size in KB) to skip them and speed up indexing.
-   **Ignoring Files**: GitNexus respects your `.gitignore`. To ignore additional files specifically for the graph, create a `.gitnexusignore` file in your project root.
-   **Visualizing the Graph**: Run `gitnexus serve` and start the `gitnexus-web` project to see an interactive dashboard of your code intelligence.

---

## 🧼 8. Maintenance & Troubleshooting

-   **Index Staleness**: If Gemini says "The index is stale," it means you have made git commits since the last analysis. Simply run `gitnexus analyze` to sync the graph.
-   **Update GitNexus**:
    ```bash
    cd GitNexus && git pull && npm install && npm run build
    ```
-   **Check Connections**: Inside Gemini CLI, use `/mcp list` to verify GitNexus is active.
