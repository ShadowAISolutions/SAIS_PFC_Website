# Project Architecture

```mermaid
graph TB
    subgraph "Repository: SAIS_PFC_Website"
        direction TB

        subgraph "Developer Workflow"
            DEV["Developer / Claude Code"]
            CB["claude/* branch"]
            DEV -->|"push"| CB
        end

        subgraph "CI/CD — auto-merge-claude.yml"
            direction TB
            TRIGGER{"Push to claude/*?"}
            CB --> TRIGGER
            TRIGGER -->|Yes| MERGE["Merge into main"]
            MERGE --> DIFF["Check git diff"]
            DIFF -->|"live-site-pages/ changed"| PAGES_FLAG["pages-changed = true"]
            DIFF -->|".gs changed"| GAS_DEPLOY["Deploy GAS via curl POST\n(no GAS projects yet)"]
            GAS_DEPLOY --> DELETE_BR
            MERGE --> DELETE_BR["Delete claude/* branch"]
            PAGES_FLAG --> DEPLOY_PAGES
            TRIGGER -->|"Direct push to main"| DEPLOY_PAGES
        end

        subgraph "GitHub Pages Deployment"
            DEPLOY_PAGES["Deploy live-site-pages/ to\nGitHub Pages"]
            LIVE["Live Site\nShadowAISolutions.github.io/SAIS_PFC_Website"]
            DEPLOY_PAGES --> LIVE
        end

        subgraph "live-site-pages/ — Hosted Content"
            direction LR
            INDEX["index.html\n(build-version: 01.01w)"]
            VERTXT["index.version.txt\n(01.01w)"]
            SND1["sounds/Website_Ready_Voice_1.mp3"]
            SND2["sounds/Code_Ready_Voice_1.mp3"]
        end

        subgraph "Auto-Refresh Loop (Client-Side)"
            direction TB
            BROWSER["Browser loads index.html"]
            POLL["Poll index.version.txt\nevery 10s"]
            COMPARE{"Remote version\n≠ loaded version?"}
            RELOAD["Set web-pending-sound\nReload page"]
            SPLASH["Show green 'Website Ready'\nsplash + play sound"]
            BROWSER --> POLL
            POLL --> COMPARE
            COMPARE -->|Yes| RELOAD
            RELOAD --> SPLASH
            COMPARE -->|No| POLL
        end

        subgraph "Template Files"
            TPL["AutoUpdateOnlyHtmlTemplate.html\n(build-version: 01.00w — never bumped)"]
            TPL_VER["AutoUpdateOnlyHtmlTemplate.version.txt"]
        end

        subgraph "Project Config"
            CLAUDE_MD["CLAUDE.md\n(project instructions)"]
            SETTINGS[".claude/settings.json\n(git * auto-allowed)"]
        end
    end

    TPL -.->|"copy to create\nnew pages"| INDEX
    LIVE -.->|"serves"| BROWSER

    style DEV fill:#4a90d9,color:#fff
    style LIVE fill:#66bb6a,color:#fff
    style SPLASH fill:#1b5e20,color:#fff
    style TPL fill:#ffa726,color:#000
    style CLAUDE_MD fill:#ce93d8,color:#000
```

Developed by: ShadowAISolutions

















