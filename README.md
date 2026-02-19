# ​‌‌‌‌‌‌‌​​‌‌‌‌‌​Auto Update HTML Template

A GitHub Pages deployment framework with automatic version polling, auto-refresh, and Google Apps Script (GAS) embedding support.

Last updated: `2026-02-19 01:40:08 PM EST`

You are currently using the **autoupdatehtmltemplate** developed by **ShadowAISolutions**<br>
Update your code and claude will update the live site link here

## Copy This Repository

1. Click the `⧉` button below to copy this template's URL:

```
https://github.com/ShadowAISolutions/autoupdatehtmltemplate
```

   > <sub>**Tip:** Link below navigates away from this page. `Shift + click` (or `Right-click` → `Open link in new window`) to keep this ReadMe visible while you work.</sub>

2. Go to [**GitHub Importer**](https://github.com/new/import) and paste what you just copied into the `The URL for your source repository *` field
3. Fill in the `Repository name *` field with a descriptive name of your choice
4. Click the green `Begin import` button

## Initialize This Template

> **Important:** The links in steps 1 and 2 below point to the settings of **whichever repo you are viewing this page from**. Make sure you are using the links below while on `YOUR OWN COPY` of the repository, not on the original template repo — otherwise the links will lead to a 404 page.

> <sub>**Tip:** Links below navigate away from this page. `Right-click` → `Open link in new window` to keep this ReadMe visible while you work.</sub>

### 1. Enable GitHub Pages

Go to your repository's [**Pages settings**](../../settings/pages) and configure:

- **Source**: Select **GitHub Actions** (not "Deploy from a branch")

  This allows the included workflow to deploy your `live-site-pages/` directory automatically.

### 2. Configure the `github-pages` Environment

Go to your repository's [**Environments settings**](../../settings/environments), click into the `github-pages` environment, and:

- Select the dropdown next to the **Deployment branches and tags** heading and choose **No restriction**

### 3. Run Claude Code and Type `initialize`

> The initialization process takes approximately **~5 to 8 minutes** from when you send `initialize` to when Claude has finished all its actions.

Open the repo with Claude Code and type **`initialize`** as your first prompt. Claude will automatically:

&emsp;Detect your new repo name and org<br>
&emsp;Update all references throughout the codebase<br>
&emsp;Replace the placeholder text above with your live site link<br>
&emsp;Commit and push — triggering the workflow to deploy to GitHub Pages

Your site will be live at `https://<your-org>.github.io/<your-repo>/`

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
autoupdatehtmltemplate/
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

> <sub>**Tip:** Links below navigate away from this page. `Right-click` → `Open link in new window` to keep this ReadMe visible while you work.</sub>

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](repository-information/ARCHITECTURE.md) | Visual system diagram (Mermaid) |
| [CHANGELOG.md](repository-information/CHANGELOG.md) | Version history |
| [CLAUDE.md](CLAUDE.md) | Developer instructions and conventions |
| [IMPROVEMENTS.md](repository-information/IMPROVEMENTS.md) | Potential improvements to explore |
| [STATUS.md](repository-information/STATUS.md) | Current project status and versions |
| [TODO.md](repository-information/TODO.md) | Actionable planned items |

## Community

> <sub>**Tip:** Links below navigate away from this page. `Right-click` → `Open link in new window` to keep this ReadMe visible while you work.</sub>

| Document | Description |
|----------|-------------|
| [Code of Conduct](CODE_OF_CONDUCT.md) | Community standards and expectations |
| [Contributing](CONTRIBUTING.md) | How to contribute to this project |
| [Security Policy](SECURITY.md) | How to report vulnerabilities |
| [Support](repository-information/SUPPORT.md) | Getting help |
| [Governance](repository-information/GOVERNANCE.md) | Project ownership and decision making |

Developed by: ShadowAISolutions

















