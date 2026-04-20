# GitHub Dependabot & Git Dependencies - Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Dependabot Architecture](#architecture)
3. [Core Concepts](#concepts)
4. [Integration Procedure](#procedure)
5. [Configuration Deep Dive](#configuration)
6. [Pipeline & Workflow](#pipeline)
7. [Git PRs & Dependabot](#git-prs)
8. [Security & Best Practices](#security)
9. [Troubleshooting](#troubleshooting)

---

## Introduction<a name="introduction"></a>

### What is GitHub Dependabot?

GitHub Dependabot is an automated dependency management system that:
- **Monitors** your project dependencies for updates and vulnerabilities
- **Creates PRs** automatically when new versions are available
- **Manages** version updates across multiple package managers
- **Integrates** with GitHub Security Advisory database
- **Runs** checks and tests on proposed dependency changes

### Key Features
- ✅ Automated vulnerability alerts
- ✅ Automatic PR creation for updates
- ✅ Support for 13+ package managers
- ✅ Customizable update schedules
- ✅ Auto-merge capabilities
- ✅ Grouped dependencies
- ✅ Version strategies (semver, auto, etc.)

---

## Dependabot Architecture<a name="architecture"></a>

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Dependabot                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  1. DEPENDENCY SCANNER & PARSER                      │   │
│  │  ├─ Reads lock files (package-lock.json, etc)        │   │
│  │  ├─ Parses manifests (requirements.txt, etc)         │   │
│  │  ├─ Identifies current versions                      │   │
│  │  └─ Tracks all dependencies (direct & transitive)   │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  2. VERSION CHECKER & REGISTRY QUERY                │   │
│  │  ├─ Queries npm, PyPI, Maven Central, etc           │   │
│  │  ├─ Identifies new versions                         │   │
│  │  ├─ Checks against allowed version ranges           │   │
│  │  ├─ Analyzes semantic versioning                    │   │
│  │  └─ Filters based on pre-release settings           │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  3. VULNERABILITY DETECTOR                           │   │
│  │  ├─ Checks GitHub Advisory Database                 │   │
│  │  ├─ Queries NVD (National Vulnerability Database)   │   │
│  │  ├─ Scans for known CVEs                            │   │
│  │  ├─ Identifies critical/high severity fixes         │   │
│  │  └─ Prioritizes security updates                    │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  4. UPDATE ELIGIBILITY ANALYZER                      │   │
│  │  ├─ Checks ignore-list configuration                │   │
│  │  ├─ Validates against version requirements          │   │
│  │  ├─ Analyzes compatibility constraints              │   │
│  │  ├─ Schedules based on frequency config             │   │
│  │  └─ Respects grouped update policies                │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  5. PR GENERATION ENGINE                             │   │
│  │  ├─ Creates new branches                            │   │
│  │  ├─ Updates manifest & lock files                   │   │
│  │  ├─ Generates commit messages                       │   │
│  │  ├─ Creates detailed PR descriptions               │   │
│  │  ├─ Adds labels & milestones                        │   │
│  │  └─ Triggers CI/CD pipelines                        │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  6. CHANGE VALIDATOR                                 │   │
│  │  ├─ Runs automated tests                            │   │
│  │  ├─ Checks build status                             │   │
│  │  ├─ Analyzes linting & code quality                 │   │
│  │  ├─ Verifies dependency conflicts                   │   │
│  │  └─ Generates confidence scores                     │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                    │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  7. MERGE ORCHESTRATOR                               │   │
│  │  ├─ Auto-merge if enabled & criteria met            │   │
│  │  ├─ Updates dependency PR references                │   │
│  │  ├─ Handles merge conflicts                         │   │
│  │  ├─ Manages PR state lifecycle                      │   │
│  │  └─ Tracks completion                               │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

```
REPOSITORY          GITHUB SERVERS          EXTERNAL REGISTRIES
     │                    │                          │
     │─ Push hook ────────→│                         │
     │                     │──── Query npm ─────────→│
     │                     │←─── Return versions ─←──│
     │                     │                         │
     │                     │──── Query PyPI ────────→│
     │                     │←─── Return versions ─←──│
     │                     │                         │
     │                     │──── Query Advisory DB ──│
     │                     │←─── CVE info ──────────│
     │                     │                         │
     │←─── PR Created ─────│                         │
     │                     │                         │
     │─ CI/CD Triggered ───→│                        │
     │← Test Results ←──────│                         │
     │                     │                         │
     │← Update Status ─────│                         │
```

### Package Manager Support

Dependabot supports **13+ package managers**:

| Language | Package Manager | Lock File |
|----------|-----------------|-----------|
| Python | pip | requirements.txt, setup.py, Pipfile |
| Python | pipenv | Pipfile, Pipfile.lock |
| Python | Poetry | pyproject.toml, poetry.lock |
| JavaScript | npm | package.json, package-lock.json |
| JavaScript | Yarn | yarn.lock |
| JavaScript | pnpm | pnpm-lock.yaml |
| Java | Maven | pom.xml |
| Java | Gradle | build.gradle, gradle.lock |
| Ruby | Bundler | Gemfile, Gemfile.lock |
| PHP | Composer | composer.json, composer.lock |
| Go | Go modules | go.mod, go.sum |
| Rust | Cargo | Cargo.toml, Cargo.lock |
| .NET | NuGet | .csproj, packages.config |
| Elixir | Mix | mix.exs, mix.lock |

---

## Core Concepts<a name="concepts"></a>

### 1. Dependency Graph Scope

```
YOUR PROJECT
├─ Direct Dependencies (explicitly listed)
│  ├─ react@18.2.0
│  ├─ fastapi@0.109.0
│  └─ express@4.18.2
│
└─ Transitive Dependencies (nested)
   ├─ From react:
   │  ├─ react-dom
   │  └─ scheduler
   ├─ From fastapi:
   │  ├─ starlette
   │  ├─ pydantic
   │  └─ uvicorn
   └─ Multiple-level deep nesting...
```

Dependabot tracks **all** dependencies and can update both direct and transitive.

### 2. Semantic Versioning (SemVer)

```
Version: MAJOR.MINOR.PATCH-PRERELEASE+METADATA
Example: 1.2.3-alpha.1+build.123

MAJOR   = Breaking changes (1.0.0 → 2.0.0)
MINOR   = New features, backward compatible (1.2.0 → 1.3.0)
PATCH   = Bug fixes (1.2.0 → 1.2.1)
```

VCS Constraints:
```
^1.2.3   = >=1.2.3, <2.0.0  (caret - minor updates allowed)
~1.2.3   = >=1.2.3, <1.3.0  (tilde - patch updates only)
1.2.3    = Exact version only
1.2.x    = >=1.2.0, <1.3.0
>=1.2.3  = Any version above
```

### 3. Version Strategies

**Strategy: auto** (Dependabot's default)
- Automatically increments version constraints based on release type
- For ^1.2.3 + minor release → becomes ^1.3.0
- For ^1.2.3 + major release → becomes ^2.0.0

**Strategy: increase-semver-minor**
- For ^1.2.3 → ^1.3.0 (allows minor bumps)
- For ~1.2.3 → ~1.3.0

**Strategy: increase-semver-major**
- For 1.2.3 → 2.0.0 (allows major bumps)
- More conservative, mainly for major releases

**Strategy: increase-semver-patch**
- For 1.2.3 → 1.2.4 (patch only)
- Most conservative approach

### 4. Dependency Groups

Allows grouping related dependencies:

```yaml
groups:
  python-testing:
    patterns:
      - "pytest*"
      - "mock*"
  
  frontend-ui:
    patterns:
      - "react*"
      - "@react*"
      - "framer*"
```

All members of a group are updated in a single PR.

### 5. Vulnerability Alert Types

**Type: Vulnerability Alert**
- CVE detected in current version
- Critical/High severity
- Fixed in newer version
- Dependabot auto-creates PR immediately

**Type: Update Alert**
- New version available
- No known vulnerabilities
- Scheduled update based on config

---

## Integration Procedure<a name="procedure"></a>

### Step 1: Enable Dependabot (UI Method)

1. Navigate to your GitHub repository
2. Go to **Settings → Code security and analysis**
3. Scroll to **Dependabot**
4. Click **Enable** for:
   - ✅ Dependabot alerts
   - ✅ Dependabot security updates
   - ✅ Dependabot version updates (⚠️ Preview)

### Step 2: Create Configuration File

Create `.github/dependabot.yml`:

```yaml
version: 2

updates:
  # Python Backend Dependencies
  - package-manager: "pip"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "03:00"
    open-pull-requests-limit: 10
    pull-request-branch-name:
      separator: "/"
    labels: ["python", "dependencies"]
    reviewers: ["@your-username"]
    allow:
      - dependency-type: "production"
    
  # Node.js Backend Dependencies
  - package-manager: "npm"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "03:30"
    open-pull-requests-limit: 5
    labels: ["javascript", "backend", "dependencies"]
    allow:
      - dependency-type: "production"
    
  # Node.js Frontend Dependencies
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "04:00"
    open-pull-requests-limit: 5
    labels: ["javascript", "frontend", "dependencies"]
    reviewers: ["@frontend-team"]
```

### Step 3: Configure PR Grouping

```yaml
updates:
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    schedule:
      interval: "weekly"
    groups:
      development:
        dependency-types:
          - "dev"
        patterns:
          - "vite*"
          - "tailwindcss*"
          - "postcss*"
          - "autoprefixer*"
      
      ui-libraries:
        dependency-types:
          - "production"
        patterns:
          - "react*"
          - "@react*"
          - "framer*"
          - "three*"
          - "@react-three*"
      
      utilities:
        dependency-types:
          - "production"
        patterns:
          - "@supabase*"
          - "lucide*"
          - "recharts*"
```

### Step 4: Set Ignore Rules

```yaml
updates:
  - package-manager: "pip"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
    ignore:
      - dependency-name: "deprecated-package"
        versions: ["*"]
      - dependency-name: "major-breaking-release"
        versions: [">=3.0.0"]
      - dependency-name: "fastapi"
        versions: [">=0.110.0"]  # Avoid specific versions
```

### Step 5: Enable Auto-Merge (Optional, for Patch Updates)

In `.github/dependabot.yml`:

```yaml
updates:
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    # ... other config ...
    auto-merge:
      enabled: true  # ⚠️ Review carefully
```

Then, configure Repository Settings:
- Go to **Settings → General**
- Enable **Automatically delete head branches**
- Enable **Require status checks to pass before merging**
- Select auto-merge approval rules

---

## Configuration Deep Dive<a name="configuration"></a>

### Complete Dependabot Configuration Schema

```yaml
version: 2

# Global settings
registries:  # Optional: Custom registry configs
  python-custom:
    type: "python-index"
    url: "https://pypi.example.com"
    username: "user"
    password: "${{ secrets.CUSTOM_REGISTRY_PASSWORD }}"

updates:
  - package-manager: "pip"  # or npm, yarn, java, etc
    
    # Location
    directory: "/soc2-analyzer/backend"
    
    # Execution Schedule
    schedule:
      interval: "weekly"  # daily, weekly, monthly
      day: "monday"  # mon-sun
      time: "03:00"  # HH:MM UTC
      timezone: "America/Los_Angeles"  # Optional
    
    # PR Limits
    open-pull-requests-limit: 10  # Max open PRs at once
    pull-requests-limit: 100  # Max total (includes closed)
    
    # Version Strategy
    version-strategy: "auto"  # auto, increase-semver-minor, etc
    
    # Branch Configuration
    pull-request-branch-name:
      separator: "/"
      prefix: "dependabot"
    target-branch: "main"  # Override default branch
    
    # Rebase & Commit
    rebase-strategy: "disabled"  # auto, disabled
    commit-message:
      prefix: "chore(deps):"
      prefix-scope: "scope-name"
      include: "scope"
    
    # Filtering & Ignoring
    allow:
      - dependency-type: "production"  # or all, indirect, direct
      - dependency-type: "development"
    
    ignore:
      - dependency-name: "package-name"
        versions: ["*"]  # Ignore all versions
      - dependency-name: "other-package"
        versions: ["2.*", ">=3.0.0"]  # Ignore specific patterns
    
    # Vulnerability Handling
    security-advisories-limit: 10  # Max vulns to auto-fix
    
    # Labels & Assignment
    labels: ["dependencies", "python"]
    reviewers: ["@user1", "team/backend"]
    assignees: ["@user1"]
    milestone: "Q1 2024"
    
    # Grouping
    groups:
      production-major:
        dependency-types: ["production"]
        update-types: ["major"]
      
      test-dependencies:
        dependency-types: ["development"]
        patterns: ["pytest*", "mock*"]
    
    # Auto-Merge Configuration
    auto-merge:
      enabled: true
    
    # Custom Registries
    registries:
      - "python-custom"
```

### Package Manager Specific Configs

**Python (pip):**
```yaml
- package-manager: "pip"
  directory: "/"
  requirements-update-strategy: "auto"  # auto, widen-ranges, increase-versions
  requirements-files: ["requirements.txt", "requirements-dev.txt"]
```

**Node.js (npm):**
```yaml
- package-manager: "npm"
  directory: "/"
  npm-shrinkwrap: true  # Consider npm-shrinkwrap.json
```

**Yarn:**
```yaml
- package-manager: "yarn"
  directory: "/"
  yarn-workspaces: true  # Handle monorepos
```

---

## Pipeline & Workflow<a name="pipeline"></a>

### Complete PR Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│  1. SCHEDULED EXECUTION                                         │
│  ├─ Dependabot wakes up at configured time                      │
│  ├─ Scans repository for all dependencies                       │
│  └─ Queries package registries for new versions                 │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  2. VERSION ANALYSIS                                            │
│  ├─ Compares current vs latest versions                         │
│  ├─ Checks update strategy rules                                │
│  ├─ Filters against ignore list                                 │
│  ├─ Groups dependencies if configured                           │
│  └─ Prioritizes security vulnerabilities                        │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  3. PR CREATION DECISION                                        │
│  ├─ Check: Are current open PRs < limit?                        │
│  ├─ Check: Is this a new update?                                │
│  ├─ Check: Does it pass ignore rules?                           │
│  ├─ Decision: Create PR or skip                                 │
│  └─ If multiple updates: Group them                             │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  4. BRANCH & FILE UPDATES                                       │
│  ├─ Create new branch: dependabot/npm_and_yarn/...             │
│  ├─ Update package.json/requirements.txt                        │
│  ├─ Update package-lock.json/poetry.lock                        │
│  ├─ Generate semantic commit message                            │
│  └─ Push to remote                                              │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  5. PR GENERATION                                               │
│  ├─ Create Pull Request with:                                   │
│  │  ├─ Title: "chore(deps): update package-name..."            │
│  │  ├─ Description: Release notes, changelog links              │
│  │  ├─ Labels: frontend, dependencies, etc                      │
│  │  ├─ Reviewers: code-owners, specified users                  │
│  │  └─ Assignees: maintainers                                   │
│  └─ Link to dependent PRs                                       │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  6. CI/CD PIPELINE TRIGGER                                      │
│  ├─ GitHub Actions runs on every commit to PR                   │
│  ├─ Steps usually include:                                      │
│  │  ├─ Install dependencies                                     │
│  │  ├─ Run unit tests                                           │
│  │  ├─ Lint code                                                │
│  │  ├─ Build application                                        │
│  │  ├─ Security scanning                                        │
│  │  └─ Code coverage checks                                     │
│  └─ Report back status to PR                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  7. PR REVIEW & MERGE DECISION                                  │
│  ├─ Check statuses:                                             │
│  │  ├─ All CI checks passed? ✓                                  │
│  │  ├─ Manual reviews approved? ✓                               │
│  │  ├─ Merge conflicts? ✗                                       │
│  │  └─ Branch protection rules satisfied? ✓                     │
│  ├─ If all green and auto-merge enabled:                        │
│  │  └─ Merge PR automatically                                   │
│  └─ Otherwise: Wait for manual review                           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  8. MERGE & CLEANUP                                             │
│  ├─ Merge PR to main/target branch                              │
│  ├─ Delete feature branch                                       │
│  ├─ Trigger post-merge CI/CD                                    │
│  └─ Close connected issues/dependencies                         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│  9. MONITORING & UPDATES                                        │
│  ├─ Track merge success                                         │
│  ├─ Monitor for new related updates                             │
│  ├─ Update dependent PRs                                        │
│  └─ Prepare next scheduled run                                  │
└─────────────────────────────────────────────────────────────────┘
```

### GitHub Actions Integration

Your existing workflow can be enhanced:

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, master, develop]
  pull_request:
    branches: [main, master]

jobs:
  # ===== EXISTING TEST JOB =====
  build-and-test:
    name: "Build & Test"
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Setup environments
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      # Install dependencies
      - name: Install Python dependencies
        working-directory: soc2-analyzer/backend
        run: pip install -r requirements.txt
      
      - name: Install Node dependencies (Backend)
        working-directory: soc2-analyzer/backend
        run: npm ci
      
      - name: Install Node dependencies (Frontend)
        working-directory: soc2-analyzer/frontend
        run: npm ci
      
      # Run tests
      - name: Run Python tests
        working-directory: soc2-analyzer/backend
        run: pytest tests/ -v
      
      - name: Run Frontend tests
        working-directory: soc2-analyzer/frontend
        run: npm run test
      
      - name: Build Backend
        working-directory: soc2-analyzer/backend
        run: npm run build
      
      - name: Build Frontend
        working-directory: soc2-analyzer/frontend
        run: npm run build

  # ===== NEW: DEPENDENCY VALIDATION =====
  dependency-check:
    name: "Dependency Security Check"
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      
      # Additional security scanning for dependabot PRs
      - name: Check for vulnerable dependencies
        run: |
          if [[ "${{ github.head_ref }}" == "dependabot/"* ]]; then
            echo "Running dependency vulnerability check..."
            # Install snyk or similar tool
            npm install -g snyk
            snyk test --severity-threshold=high
          fi

  # ===== OPTIONAL: AUTO-APPROVAL FOR SAFE UPDATES =====
  approve-dependabot:
    name: "Auto-Approve Safe Updates"
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request' && github.actor == 'dependabot[bot]'
    permissions:
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
      
      - name: Approve & Enable Auto-Merge
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            
            // Only auto-approve patch updates & security fixes
            if (pr.title.includes('patch') || pr.title.includes('security')) {
              await github.rest.pulls.createReview({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: pr.number,
                event: 'APPROVE',
                body: 'Auto-approved: safe dependency update ✅'
              });
              
              // Enable auto-merge for squash strategy
              await github.rest.pulls.enableAutoMerge({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: pr.number,
                merge_method: 'squash'
              });
            }
```

---

## Git PRs & Dependabot<a name="git-prs"></a>

### PR Creation Process

**Standard Dependabot PR:**
```
Title: chore(deps): bump react from 18.2.0 to 18.3.0

Body:
Bumps [react](https://github.com/facebook/react) from 18.2.0 to 18.3.0.

Release notes:
Sourced from [react](https://github.com/facebook/react/releases).

## v18.3.0

### New Features
- New useOptimistic hook
- New experimental useFormState hook

[Changelog](...)
[Compare](...)

---
Dependabot will resolve any conflicts with this PR as long as you
don't alter it yourself. You can also trigger a rebase manually by
commenting `@dependabot rebase`.

---
Dependabot commands and options
- `@dependabot rebase` will rebase this PR
- `@dependabot recreate` will recreate this PR
- `@dependabot merge` will merge this PR (if tests pass)
- `@dependabot squash and merge` will squash and merge this PR
- `@dependabot cancel merge` will cancel a previously requested merge
- `@dependabot reopen` will reopen this PR if it is closed
- `@dependabot close` will close this PR
- `@dependabot show <dependency-name> dependency updates` will show the
  dependency updates for the shown dependency
- `@dependabot ignore this major version` will close this PR and stop
  Dependabot creating any more for this major version (unless you
  reopen the PR or upgrade to it using `@dependabot upgrade`)
- `@dependabot ignore <dependency-name> major version` will close this
  PR and stop Dependabot creating any more for this dependency's major
  version (unless you reopen the PR or upgrade to it yourself)
```

### Dependabot PR Commands

| Command | Effect |
|---------|--------|
| `@dependabot rebase` | Rebase on conflicts |
| `@dependabot recreate` | Delete & recreate PR |
| `@dependabot merge` | Merge if tests pass |
| `@dependabot squash and merge` | Squash commits & merge |
| `@dependabot close` | Close PR |
| `@dependabot reopen` | Reopen closed PR |
| `@dependabot ignore <pkg> major version` | Stop major version updates |
| `@dependabot ignore this major version` | Ignore this specific major |

### Handling Merge Conflicts

When dependencies conflict:

1. **Automatic Resolution (if rebase-strategy: auto)**
   ```yaml
   rebase-strategy: "auto"
   ```
   Dependabot automatically rebases when conflicts detected.

2. **Manual Resolution**
   ```bash
   # Comment on the PR
   @dependabot rebase
   ```

3. **Recreate PR**
   ```
   @dependabot recreate
   ```

### PR Status Checks

Dependabot PRs show multiple status checks:

```
Branches:
✓ All checks have passed (3)
├─ Build & Test - Passed
├─ Lint Check - Passed
└─ Security Scan - Passed

Required approvals:
⏳ Awaiting review from code-owners
```

### Linked PRs

When dependencies depend on each other:

```
PR #123: bump react from 18.2.0 to 18.3.0
PR #124: bump @types/react from 18.2.0 to 18.3.0

These updates are linked because @types/react depends on react.
```

---

## Security & Best Practices<a name="security"></a>

### 1. Vulnerability Management

**Critical Security Update Flow:**
```
┌─────────────────────────┐
│ CVE Discovered          │
│ (e.g., XSS in React)    │
└────────────┬────────────┘
             ↓
┌─────────────────────────────────────────┐
│ GitHub Advisory Database Updated        │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ Dependabot Detects in your Project       │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ Automatic PR Created (Security Alert)    │
│ - Labels: security, critical             │
│ - Auto-assigns reviewers                 │
│ - No merge delays                        │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ Team Reviews & Tests PR                  │
│ - Run full test suite                    │
│ - Check for breaking changes             │
│ - Verify compatibility                   │
└────────────┬────────────────────────────┘
             ↓
┌─────────────────────────────────────────┐
│ Merge & Deploy Security Patch            │
└─────────────────────────────────────────┘
```

### 2. Best Practices

**DO:**
- ✅ Enable both Alerts AND Version Updates
- ✅ Use weekly schedules for routine updates
- ✅ Group related dependencies
- ✅ Auto-merge patch versions
- ✅ Require tests to pass before merge
- ✅ Review major/minor updates manually
- ✅ Monitor update trends
- ✅ Keep lock files committed

**DON'T:**
- ❌ Ignore security vulnerabilities
- ❌ Set too aggressive schedules
- ❌ Auto-merge major versions
- ❌ Merge without testing
- ❌ Ignore flaky CI pipelines
- ❌ Keep outdated dependencies
- ❌ Block legitimate updates indefinitely

### 3. Risk Management

**Version Update Risk Assessment:**

```
┌──────────────────────────────────────────────────────┐
│                    RISK MATRIX                       │
├──────────────────────────────────────────────────────┤
│                                                      │
│  MAJOR UPDATE                                        │
│  ┌─────────────────────────────────────────┐        │
│  │ ⚠️ HIGH RISK                            │        │
│  │ • Breaking changes common               │        │
│  │ • Extensive testing needed              │        │
│  │ • May require code changes              │        │
│  │ • Manual review required                │        │
│  │ → Auto-merge: DISABLED                  │        │
│  └─────────────────────────────────────────┘        │
│                                                      │
│  MINOR UPDATE                                        │
│  ┌─────────────────────────────────────────┐        │
│  │ ⚠️ MEDIUM RISK                          │        │
│  │ • New features added                     │        │
│  │ • Backward compatible                    │        │
│  │ • Most apps work fine                    │        │
│  │ • Recommended: Manual review             │        │
│  │ → Auto-merge: OPTIONAL                  │        │
│  └─────────────────────────────────────────┘        │
│                                                      │
│  PATCH UPDATE                                        │
│  ┌─────────────────────────────────────────┐        │
│  │ ✅ LOW RISK                             │        │
│  │ • Bug fixes only                         │        │
│  │ • Fully backward compatible              │        │
│  │ • No new features                        │        │
│  │ • Tests usually sufficient               │        │
│  │ → Auto-merge: RECOMMENDED                │        │
│  └─────────────────────────────────────────┘        │
│                                                      │
│  SECURITY PATCH                                      │
│  ┌─────────────────────────────────────────┐        │
│  │ 🔴 CRITICAL!                            │        │
│  │ • CVE vulnerability fix                  │        │
│  │ • Must be reviewed ASAP                  │        │
│  │ • Deploy IMMEDIATELY after testing       │        │
│  │ • Auto-merge: OK for trusted packages    │        │
│  │ → Auto-merge: HIGHLY RECOMMENDED         │        │
│  └─────────────────────────────────────────┘        │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### 4. Custom Security Policies

```yaml
# .github/dependabot.yml with security focus
version: 2

updates:
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    
    schedule:
      interval: "weekly"
      day: "monday"
    
    # Separate production from dev
    groups:
      security-patches:
        dependency-types: ["production"]
        update-types: ["patch", "security"]
      
      development:
        dependency-types: ["development"]
        update-types: ["patch", "minor"]
      
      major-updates:
        dependency-types: ["production"]
        update-types: ["major"]
    
    auto-merge:
      enabled: true
      rules:
        - dependency-type: "production"
          update-type: ["patch", "security"]
        # Require manual review for major versions
```

---

## Troubleshooting<a name="troubleshooting"></a>

### Common Issues & Solutions

#### Issue 1: Dependabot PRs Not Being Created

**Symptoms:** No PRs created even though dependencies are outdated

**Root Causes & Solutions:**

1. **Dependabot not enabled**
   ```
   Settings → Code security → Enable Dependabot
   ```

2. **.github/dependabot.yml syntax error**
   ```yaml
   # ❌ Wrong indentation
   updates:
     - package-manager: "npm"
     directory: "/path"
   
   # ✅ Correct indentation
   updates:
     - package-manager: "npm"
       directory: "/path"
   ```

3. **Open PR limit reached**
   ```yaml
   open-pull-requests-limit: 10  # Increase this
   ```

4. **All updates in ignore list**
   ```yaml
   ignore:
     - dependency-name: "*"  # ❌ Ignores everything!
   ```

5. **Schedule hasn't run yet**
   - Dependabot runs on UTC schedule
   - May take up to 24 hours for first run
   - Can't manually trigger, must wait

**Debug Steps:**
```bash
# Check if config file is valid
# GitHub will show syntax errors in Settings → Code security

# Check repository permissions
# Dependabot needs:
# - Read access to all branches
# - Write access to create branches/PRs
```

#### Issue 2: PR Fails CI/CD Tests

**Symptoms:** Dependabot PR created but tests fail

**Diagnosis:**

1. **Build fails**
   ```bash
   npm install  # Check if new version breaks deps
   npm run build  # Check for compilation errors
   ```

2. **Test failures**
   - Review test output
   - May be pre-existing failures (check main branch)
   - May be breaking change in new version

3. **Type errors (TypeScript)**
   - New version may have breaking types
   - Update import statements
   - Run `npm run type-check`

**Solutions:**

```yaml
# Temporarily ignore problematic packages
ignore:
  - dependency-name: "problematic-package"
    versions: [">=2.0.0"]

# Or require manual review
groups:
  manual-review:
    dependency-types: ["production"]
    patterns: ["*"]
```

#### Issue 3: Merge Conflicts in Lock Files

**Symptoms:** PR shows merge conflict in package-lock.json

**Solutions:**

1. **Auto-rebase**
   ```yaml
   rebase-strategy: "auto"
   ```

2. **Manual rebase**
   ```
   Comment on PR: @dependabot rebase
   ```

3. **Recreate PR**
   ```
   Comment on PR: @dependabot recreate
   ```

#### Issue 4: Too Many PRs Created

**Symptoms:** Overwhelmed with Dependabot PRs

**Solutions:**

```yaml
# Limit number of open PRs
open-pull-requests-limit: 3

# Group related dependencies
groups:
  dependencies:
    dependency-types: ["production"]
    patterns: ["*"]

# Reduce check frequency
schedule:
  interval: "monthly"  # Instead of weekly
```

#### Issue 5: Specific Package Keeps Failing

**Symptoms:** `package-name` PR always fails tests

**Investigation:**

```bash
# Test locally
# 1. Create test branch
git checkout -b test/package-upgrade
npm install package-name@new-version
npm run test

# 2. Check changelog for breaking changes
# Visit github.com/owner/repo/releases

# 3. Check if it's a known issue
# Search issues in their repo
```

**Resolution:**

```yaml
# Option 1: Ignore major versions
ignore:
  - dependency-name: "package-name"
    versions: [">=2.0.0"]

# Option 2: Require manual updates
groups:
  manual-only:
    patterns: ["package-name"]
    # Dependabot creates PR but doesn't auto-merge
```

### Monitoring Dependabot Health

**Check Dashboard:**
1. Go to repo **Settings → Code security → Dependabot**
2. View:
   - Alerts (vulnerabilities found)
   - Updates (recent/pending updates)
   - Configuration status

**View Public Insights:**
```
https://github.com/owner/repo/network/dependencies
```

**Check PR Activity:**
```
https://github.com/owner/repo/pulls?q=is:pr+author:dependabot
```

---

## Implementation Summary

### Quick Start for Your Project

1. **Create `.github/dependabot.yml`:**
```yaml
version: 2

updates:
  - package-manager: "pip"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "03:00"
    labels: ["python", "dependencies"]
    
  - package-manager: "npm"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "03:30"
    labels: ["javascript", "backend"]
    
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "04:00"
    labels: ["javascript", "frontend"]
```

2. **Enable in GitHub UI:**
   - Settings → Code security → Enable Dependabot

3. **Create GitHub Actions workflow:**
   - Copy enhanced CI/CD pipeline above
   - Ensure tests run on Dependabot PRs

4. **Monitor & Adjust:**
   - Review PRs weekly
   - Adjust schedule/grouping based on volume

---

## References

- [GitHub Dependabot Documentation](https://docs.github.com/en/code-security/dependabot)
- [Configuring Dependabot](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/about-dependabot-version-updates)
- [Dependabot Commands](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/managing-pull-requests-for-dependency-updates)
- [GitHub Advisory Database](https://github.com/advisories)
