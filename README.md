# Blueprints for Power and Agents

> A plugin for GitHub Copilot CLI and Claude Code that analyzes Power Platform, Copilot Studio, and Azure AI Foundry solutions and generates interactive HTML architecture dashboards.

<img width="1747" height="1272" alt="Screenshot 2026-03-11 193626" src="https://github.com/user-attachments/assets/be60e636-85f2-4232-a0d0-93131b14eb8c" />


[![View Demo Site](https://img.shields.io/badge/View-Demo_Site-FF6F00?style=for-the-badge&logo=github-pages&logoColor=white)](https://promptclickrun.github.io/power-agents-blueprint/)
[![Install in Copilot CLI](https://img.shields.io/badge/Copilot_CLI-Install_Plugin-2088FF?style=for-the-badge&logo=github&logoColor=white)](#-copilot-cli-quick-install)
[![Install in Claude Code](https://img.shields.io/badge/Claude_Code-Install_Plugin-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](#-claude-code-quick-install)
[![Open in VS Code](https://img.shields.io/badge/Open_in-VS_Code-007ACC?style=for-the-badge&logo=visual-studio-code&logoColor=white)](https://vscode.dev/github/promptclickrun/power-agents-blueprint)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

---

## ⚡ Quick Install

### 🟦 Copilot CLI Quick Install

Open a terminal in VS Code and run:

```bash
copilot plugin install promptclickrun/power-agents-blueprint
```

That's it. The agent and skills are ready to use.

<details>
<summary><strong>Alternative: Marketplace install</strong></summary>

```bash
# 1. Register the marketplace
copilot plugin marketplace add promptclickrun/power-agents-blueprint

# 2. Install the plugin
copilot plugin install power-agents-blueprint@power-agents-blueprint-marketplace
```
</details>

<details>
<summary><strong>Alternative: Manual project-level install (no global install)</strong></summary>

Copy the plugin files directly into your project's `.github/` directory:

```
your-project/
└── .github/
    ├── agents/
    │   └── power-copilot-architect.agent.md
    └── skills/
        ├── architecture-html-dashboard/
        │   └── SKILL.md
        └── pagedrop-upload/
            └── SKILL.md
```
</details>

**Verify:**

```bash
copilot plugin list
```

---

### 🟧 Claude Code Quick Install

In a Claude Code session, run:

```
/plugin marketplace add promptclickrun/power-agents-blueprint
/plugin install power-agents-blueprint@power-agents-blueprint-marketplace
```

Or from your terminal:

```bash
claude plugin marketplace add promptclickrun/power-agents-blueprint
claude plugin install power-agents-blueprint@power-agents-blueprint-marketplace
```

<details>
<summary><strong>Alternative: Manual project-level install</strong></summary>

Copy the plugin files into your project's `.claude/` or `.github/` directory:

```
your-project/
└── .github/
    ├── agents/
    │   └── power-copilot-architect.agent.md
    └── skills/
        ├── architecture-html-dashboard/
        │   └── SKILL.md
        └── pagedrop-upload/
            └── SKILL.md
```
</details>

**Verify:**

```
/plugin list
```

---

## What It Does

Give this plugin a Power Platform solution — a Copilot Studio agent directory, a solution ZIP, or a description — and it will:

1. **Analyze** all components: agents, topics, actions, flows, connectors, data entities, knowledge sources
2. **Generate** a self-contained, interactive HTML dashboard with:
   - Mermaid architecture diagrams and ERDs
   - Clickable stat cards with detailed modal drawers
   - Data flow timelines with decision branches
   - Full component inventory with IDs and configurations
   - Architecture notes (strengths, risks, dependencies, ROI)
3. **Share** the dashboard via a 3-day PageDrop link, or open it locally

---

## Usage

### Invoke the Agent

In either Copilot CLI or Claude Code, just describe what you need:

- *"Analyze the HR Onboarding Assistant agent and generate an architecture dashboard"*
- *"Create an architecture diagram for this solution ZIP"*
- *"Review the architecture of my Power Platform solution"*

### Supported Inputs

| Input Type | How to Provide |
|---|---|
| Copilot Studio agent directory | Point to a folder with `.mcs.yml` files |
| Power Platform solution ZIP | Provide a path to a `.zip` export |
| Solution description | Describe your components in chat |

### Output

A single `{solution-name}-architecture.html` file with 5 interactive tabs:

| Tab | Content |
|---|---|
| **Architecture** | Mermaid flowchart with layered component diagram |
| **ERD** | Entity Relationship Diagram with all data entities |
| **Data Flows** | Timeline visualization of data movement and decisions |
| **Components** | Categorized inventory cards with full details |
| **Notes** | Architecture strengths, considerations, dependencies |

After generation, you'll be asked to:
- **Share** via a 3-day PageDrop link (no signup required)
- **Open locally** in the default browser

---

## Plugin Contents

```
power-agents-blueprint/
├── plugin.json                                  # Plugin manifest
├── agents/
│   └── power-copilot-architect.agent.md         # Agent definition (~130 lines)
└── skills/
    ├── architecture-html-dashboard/
    │   └── SKILL.md                             # HTML template + rendering rules (~480 lines)
    └── pagedrop-upload/
        └── SKILL.md                             # PageDrop API + upload logic (~160 lines)
```

| Component | Purpose |
|---|---|
| **Agent** | Analysis methodology — explores solutions, maps components, traces data flows, orchestrates output |
| **Dashboard Skill** | Full HTML/CSS/JS template with dark theme, Mermaid diagrams, modal drawers, responsive layout |
| **PageDrop Skill** | API reference for uploading HTML to shareable time-limited links |

---

## Prerequisites

- **GitHub Copilot CLI** or **Claude Code** with plugin support
- **PowerShell** (for `Start-Process` and `Invoke-RestMethod`)
- **Internet access** for Mermaid CDN and PageDrop API

No API keys, tokens, or authentication required.

---

## Examples

### Copilot Studio Agent Analysis

```
User: Analyze the IT Help Desk Triage Agent and create an architecture dashboard

Agent: [Reads all .mcs.yml files → Maps 7 actions, 14 topics → Traces routing logic]
       [Generates triage-agent-architecture.html]

       Would you like a shareable link? (active for 3 days)
       > Yes

       🔗 Your architecture dashboard is live for 3 days:
       https://cool-forest-x2k4.pagedrop.io
```

### Power Platform Solution ZIP

```
User: Provide an architecture file for ExpenseApprovalWorkflow.zip

Agent: [Extracts ZIP → Analyzes entities, flows, relationships]
       [Generates expense-approval-workflow-architecture.html]

       Would you like a shareable link? (active for 3 days)
       > No — just open it locally

       📂 File saved to: C:\path\to\expense-approval-workflow-architecture.html
```

### Visual Walkthrough

Here's what the end-to-end flow looks like when analyzing a Power Platform solution ZIP:

**1. Agent analyzes the solution**

The agent extracts the ZIP, reads all solution components, and begins generating the architecture dashboard.

![Step 1 — Agent analyzes the solution ZIP](assets/Arch-1-Initialize.png)

**2. Choose how to share**

Once the dashboard is generated, you're prompted to share via a temporary PageDrop link or open it locally.

![Step 2 — Share options for the architecture dashboard](assets/Arch-2-Prompt.png)

**3. Dashboard ready**

The shareable link is live and the dashboard summary shows what's included — architecture diagrams, ERD, data flows, components, and notes.

![Step 3 — Dashboard ready with shareable link](assets/Arch-3-Result.png)

<img width="1511" height="1170" alt="Screenshot 2026-03-11 193351" src="https://github.com/user-attachments/assets/9be0f1fb-0959-411e-b321-d8ba68e0e515" />


---

## Updating

```bash
# Copilot CLI
copilot plugin update power-agents-blueprint

# Claude Code
/plugin marketplace update power-agents-blueprint-marketplace
```

## Uninstalling

```bash
# Copilot CLI
copilot plugin uninstall power-agents-blueprint

# Claude Code
/plugin uninstall power-agents-blueprint@power-agents-blueprint-marketplace
```

---

## License

MIT
