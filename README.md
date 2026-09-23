# 🧠 Nyrobrain Agent Skills (`Anson-Nyrobrain-Skills`)

<img width="868" height="540" alt="Screenshot 2026-05-04 at 8 34 16 PM" src="https://github.com/user-attachments/assets/f25188f6-8d08-4818-9c10-e3e5a9fbe46c" />
<img width="1371" height="523" alt="Screenshot 2026-09-23 at 5 33 43 PM" src="https://github.com/user-attachments/assets/51d6cb5b-0e5f-4c00-9e5d-187cdaa9c7b8" />



Welcome to **Nyrobrain Skills**—a high-performance, data-driven AI agent skill layer built for automated quantitative research, strategy optimization, and systematic execution. 

This repository leverages the native agent skill surface, moving away from legacy manual documentation in favor of dynamic, execution-ready system capabilities that plug directly into your local AI agent (Claude Code, OpenClaw, Codex).

---

## 🚀 Architectural Overview

Nyrobrain operates on a modular skill-based framework. Instead of copying text instructions manually, these skills graft specialized behaviors straight into your agent's runtime directory (`~/.agents/skills/`), providing isolated context management and robust execution scaffolding.

### Core Skill Surfaces Available:
* **`/strategy`** – Quantitative alpha research, backtesting parameters, and mathematical signals.
* **`/portfolio`** – Risk management limits, dynamic asset allocation, and delta hedging rules.
* **`/execution`** – Order routing logic, liquidity analysis, and slippage mitigation pipelines.
* **`/deploy`** – Live pipeline production, Docker/VM orchestration, and agentic heartbeat tracking.
* **`/guide`** – Interactive onboarding, pipeline visualization, and architecture maps.

---

## 📦 Installation

Install the entire Nyrobrain skill suite directly into your local agent environment using a single command:

```bash
npx skills add AnsonBQ/Anson-Nyrobrain-Skills
```

### Upgrading Skills
To pull the latest alpha research strategies and performance adjustments from the main branch, simply re-run the installer or target a specific updates check:
```bash
npx skills update AnsonBQ/Anson-Nyrobrain-Skills
```

---

## ⚙️ Workflow & Pipeline Management

While legacy workflows relied heavily on static setup files inside `docs/pipelines/`, this repository strictly prioritizes dynamic pipeline generation.

### Creating a New Quant Pipeline
To spin up a new strategy structure with clean boundaries, ask your agent:
> *"Use `/make-pipeline` to construct a new alpha research loop for [Asset Name]."*

The agent will leverage Nyrobrain's core runtime logic to establish proper workspace scoping and project isolation boundaries automatically.

---

## 🗂️ Repository Structure

```text
Anson-Nyrobrain-Skills/
├── skills/                     # Active agent skills directory
│   ├── strategy/               # Alpha generation & backtesting logic
│   │   └── SKILL.md
│   ├── portfolio/              # Risk management & asset sizing
│   │   └── SKILL.md
│   ├── execution/              # Order execution & API endpoints
│   │   └── SKILL.md
│   └── deploy/                 # Production pipeline controllers
│       └── SKILL.md
└── README.md                   # Project documentation
```

---

## 🛡️ Workspace & Safety Constraints

All Nyrobrain skills operate under rigorous runtime constraints to prevent memory leaks and context pollution during prolonged backtests:
* **Context Isolation:** Every research process runs under scoped project workspaces.
* **Execution Heartbeats:** Active pipelines maintain a persistent progress heartbeat (`PROGRESS.md`) to handle unexpected disconnects safely.

---

## 🤝 Contributing & Local Development

If you are developing custom patches or expanding the trading surfaces:
1. Fork the repository.
2. Build reusable modules under the `skills/` subdirectory rather than creating standalone text instructions.
3. Validate your triggers locally before opening a Pull Request.
