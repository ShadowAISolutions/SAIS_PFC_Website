# ​‌‌‌‌‌‌‌​​‌‌‌‌‌​ReadMe - SAIS_PFC_Website

A GitHub Pages deployment framework with automatic version polling, auto-refresh, and Google Apps Script (GAS) embedding support.

Last updated: `2026-02-19 02:50:56 PM EST`

**Live site:** [ShadowAISolutions.github.io/SAIS_PFC_Website](https://ShadowAISolutions.github.io/SAIS_PFC_Website)

## How It Works

### Auto-Refresh via Version Polling
Every hosted page polls a lightweight `.version.txt` file every 10 seconds. When a new version is deployed, the page detects the mismatch and auto-reloads — showing a green "Website Ready" splash with audio feedback.

### CI/CD Auto-Merge Flow
1. Push to a `claude/*` branch
2. GitHub Actions automatically merges into `main`, deploys to GitHub Pages, and cleans up the branch
3. No pull requests needed — the workflow handles everything

### GAS Embedding Architecture
Google Apps Script projects are embedded as iframes in GitHub Pages. The framework handles:

&emsp;Automatic GAS deployment via `doPost` when `.gs` files change<br>
&emsp;"Code Ready" blue splash on GAS updates (client-side polling)<br>
&emsp;Google Sign-In from the parent page (stable OAuth origin)

## Project Structure

```
SAIS_PFC_Website/
├── live-site-pages/             # Deployed to GitHub Pages
│   ├── index.html              # Live landing page
│   ├── index.version.txt       # Version file for auto-refresh
│   └── sounds/                 # Audio feedback files
├── live-site-templates/        # Template for new pages
├── .github/
│   ├── workflows/              # CI/CD pipeline
│   ├── ISSUE_TEMPLATE/         # Bug report & feature request forms
│   ├── PULL_REQUEST_TEMPLATE.md # PR checklist
│   └── FUNDING.yml             # Sponsor button config
├── repository-information/
│   ├── ARCHITECTURE.md         # System diagram (Mermaid)
│   ├── CHANGELOG.md            # Version history
│   ├── CODING-GUIDELINES.md    # Domain-specific coding knowledge
│   ├── GOVERNANCE.md           # Project governance
│   ├── IMPROVEMENTS.md         # Potential improvements
│   ├── STATUS.md               # Project status dashboard
│   ├── TODO.md                 # Actionable to-do items
│   ├── TOKEN-BUDGETS.md        # Token cost reference for CLAUDE.md
│   └── SUPPORT.md              # Getting help
├── CITATION.cff                # Citation metadata
├── CLAUDE.md                   # Developer instructions
├── CODE_OF_CONDUCT.md          # Community standards
├── CONTRIBUTING.md             # How to contribute
├── LICENSE                     # Proprietary license
└── SECURITY.md                 # Vulnerability reporting
```

## Documentation

> **Tip:** Links below navigate away from this page. **Ctrl + click** (or right-click → *Open in new tab*) to keep this ReadMe visible while you work.

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](repository-information/ARCHITECTURE.md) | Visual system diagram (Mermaid) |
| [CHANGELOG.md](repository-information/CHANGELOG.md) | Version history |
| [CLAUDE.md](CLAUDE.md) | Developer instructions and conventions |
| [IMPROVEMENTS.md](repository-information/IMPROVEMENTS.md) | Potential improvements to explore |
| [STATUS.md](repository-information/STATUS.md) | Current project status and versions |
| [TODO.md](repository-information/TODO.md) | Actionable planned items |

## Community

> **Tip:** Links below navigate away from this page. **Ctrl + click** (or right-click → *Open in new tab*) to keep this ReadMe visible while you work.

| Document | Description |
|----------|-------------|
| [Code of Conduct](CODE_OF_CONDUCT.md) | Community standards and expectations |
| [Contributing](CONTRIBUTING.md) | How to contribute to this project |
| [Security Policy](SECURITY.md) | How to report vulnerabilities |
| [Support](repository-information/SUPPORT.md) | Getting help |
| [Governance](repository-information/GOVERNANCE.md) | Project ownership and decision making |

Developed by: ShadowAISolutions

















