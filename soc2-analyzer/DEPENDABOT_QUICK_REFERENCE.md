# Dependabot Quick Reference & Management Guide

## 📋 File Locations

```
.github/
├── dependabot.yml              # Dependabot configuration
└── workflows/
    ├── compliance-scan.yml     # (existing) Your compliance scans
    └── dependabot-integration.yml  # (new) Dependabot automation
```

---

## 🚀 Quick Start

### 1. Enable Dependabot (One-time)
```
GitHub.com → Repository Settings → Code security and analysis
→ Enable "Dependabot alerts" ✓
→ Enable "Dependabot security updates" ✓
```

### 2. Configuration Checklist
- ✅ `.github/dependabot.yml` created
- ✅ Schedule set (Monday 3:00 AM UTC)
- ✅ Labels configured
- ✅ Open PR limits set
- ✅ Ignore rules reviewed
- ✅ Grouping strategy defined

### 3. Validate CI/CD Pipeline
- ✅ Run test workflow on Dependabot PR
- ✅ Verify tests pass with new dependencies
- ✅ Check build succeeds

### 4. Monitor & Adjust
- 📊 Review PRs weekly
- 🔧 Adjust schedule/groups as needed
- 🔐 Watch for security vulnerabilities

---

## 📊 Expected Behavior

### First Run (24-48 hours after enabling)
- Dependabot scans repository
- Creates PRs for available updates
- You'll see 5-15 PRs depending on staleness

### Subsequent Runs (Weekly)
- Creates 1-3 PRs per week normally
- Security fixes get immediate attention
- Patch updates usually batch together

### Timeline Example
```
Monday 3:00 AM UTC:
├─ Run dependency scan
├─ Find 2 patch updates
├─ Find 1 minor update
└─ Create 3 PRs (based on grouping)

Tuesday Morning:
├─ Your team reviews PRs
├─ Tests run automatically
└─ Approve/merge as needed

Next Monday 3:00 AM UTC:
└─ Repeat
```

---

## 🎯 Dependabot PR Management Commands

### Comment on PR to Trigger Actions

| Command | Effect |
|---------|--------|
| `@dependabot rebase` | Rebase PR on conflicts |
| `@dependabot recreate` | Delete and recreate entire PR |
| `@dependabot merge` | Merge if all checks pass |
| `@dependabot squash and merge` | Squash commits and merge |
| `@dependabot close` | Close the PR |
| `@dependabot reopen` | Reopen closed PR |
| `@dependabot ignore this major version` | Skip this major release |
| `@dependabot ignore <package> major version` | Skip major for that package |

### Examples
```bash
# If merge conflict occurs
@dependabot rebase

# If tests pass and ready
@dependabot squash and merge

# If package is problematic
@dependabot ignore this major version
```

---

## 🔍 Monitoring & Diagnostics

### Check Dependabot Status
1. **In GitHub UI:**
   - Repository → Settings → Code security → Dependabot section
   - Shows: Alerts, Updates, Configuration status

2. **View All Dependabot PRs:**
   ```
   https://github.com/YOUR_REPO/pulls?q=author:dependabot
   ```

3. **Check Dependency Graph:**
   ```
   https://github.com/YOUR_REPO/network/dependencies
   ```

### Troubleshooting

**❌ PRs not being created?**
```yaml
# Check:
1. Is Dependabot enabled? (Settings → Code security)
2. Is .github/dependabot.yml valid? (No YAML syntax errors)
3. Are all dependencies in ignore list?
4. Have 24+ hours passed since enabling?

# To debug:
# - Check GitHub Settings for error messages
# - Validate YAML at https://www.yamllint.com
# - Check if schedule window has occurred
```

**❌ PR fails CI/CD tests?**
```bash
# Check:
1. Test on main branch first (is it a pre-existing failure?)
2. Run npm install / pip install locally
3. Review changelog for breaking changes
4. Check if types need updating

# Solution:
# If legitimate incompatibility, ignore the version:
ignore:
  - dependency-name: "problem-package"
    versions: [">=2.0.0"]
```

**❌ Too many PRs created?**
```yaml
# Solutions:
1. Reduce frequency:
   schedule:
     interval: "monthly"  # Instead of weekly

2. Reduce open PR limit:
   open-pull-requests-limit: 3  # Instead of 10

3. Group all dependencies:
   groups:
     all-deps:
       patterns: ["*"]  # Update all together
```

---

## 🔐 Security Vulnerability Response

### Urgent: Critical CVE in Current Dependencies

**Immediate Actions:**
1. GitHub will alert you (Settings → Security advisories)
2. Dependabot auto-creates PR with fix
3. **Merge security PR IMMEDIATELY after tests pass**
4. Deploy to production ASAP

**Response Time SLA:**
- Critical (CVSS 9.0+): Fix within 24 hours
- High (CVSS 7.0-8.9): Fix within 1 week
- Medium (CVSS 4.0-6.9): Fix within 2 weeks
- Low: Review quarterly

---

## 📈 Performance & Customization

### Recommended Schedules by Team Size

| Team Size | Recommendation |
|-----------|-----------------|
| Solo Dev | Monthly updates |
| 2-3 Devs | Weekly patch, Monthly minor/major |
| 5+ Devs | Daily patch, Weekly minor, Monthly major |

### Recommended Grouping Strategy

```yaml
groups:
  # Patch security fixes only (auto-merge after testing)
  security-patches:
    dependency-types: ["production"]
    update-types: ["security"]
  
  # Patch bug fixes (auto-merge after testing)
  patch-updates:
    dependency-types: ["production"]
    update-types: ["patch"]
  
  # Minor new features (manual review recommended)
  minor-updates:
    dependency-types: ["production"]
    update-types: ["minor"]
  
  # Major breaking changes (always manual review)
  major-updates:
    dependency-types: ["production"]
    update-types: ["major"]
  
  # Development dependencies (batch together)
  dev-updates:
    dependency-types: ["development"]
```

### Auto-Merge Best Practices

```yaml
# ✅ SAFE TO AUTO-MERGE:
auto-merge: true  # For patch updates only

# ❌ DANGEROUS TO AUTO-MERGE:
# - Major version updates
# - Minor updates with unknown breaking changes
# - Updates from untrusted sources
# - Updates without comprehensive test coverage
```

---

## 📝 Configuration Templates

### Minimal Setup (Starting Point)
```yaml
version: 2
updates:
  - package-manager: "pip"
    directory: "/soc2-analyzer/backend"
    schedule:
      interval: "weekly"
    labels: ["python", "dependencies"]
  
  - package-manager: "npm"
    directory: "/soc2-analyzer/frontend"
    schedule:
      interval: "weekly"
    labels: ["javascript", "dependencies"]
```

### Advanced Setup (Production Ready)
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
    reviewers: ["@backend-team"]
    
    groups:
      patches:
        update-types: ["patch"]
      minor:
        update-types: ["minor"]
      major:
        update-types: ["major"]
    
    allow:
      - dependency-type: "production"
    
    ignore:
      - dependency-name: "deprecated-package"
        versions: ["*"]
    
    auto-merge:
      enabled: true  # Only after validating CI/CD
```

---

## 🎓 Learning Resources

### Key Concepts
- **Semver:** MAJOR.MINOR.PATCH-PRERELEASE
- **Update Types:** patch (bugfix), minor (feature), major (breaking)
- **Dependency Scope:** production (app) vs development (testing/build)
- **Lock Files:** Exact versions from install, always commit

### Common Patterns

**Match multiple patterns:**
```yaml
patterns:
  - "react*"
  - "@react*"
  - "@types/react*"
```

**Ignore multiple packages:**
```yaml
ignore:
  - dependency-name: "package1"
  - dependency-name: "package2"
  - dependency-name: "package3"
```

**Timezone examples:**
```yaml
timezone: "America/New_York"      # EST/EDT
timezone: "Europe/London"          # GMT/BST
timezone: "Asia/Tokyo"             # JST
timezone: "Australia/Sydney"       # AEDT/AEST
# Full list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
```

---

## 🔗 External Resources

- 📖 [Official Dependabot Docs](https://docs.github.com/en/code-security/dependabot)
- 🔐 [GitHub Advisory Database](https://github.com/advisories)
- 📊 [Semantic Versioning](https://semver.org/)
- 🐍 [PyPI Package Index](https://pypi.org)
- 📦 [NPM Registry](https://www.npmjs.com)

---

## ✅ Implementation Checklist

- [ ] Enable Dependabot in GitHub Settings
- [ ] Copy `.github/dependabot.yml` to repository
- [ ] Copy `.github/workflows/dependabot-integration.yml` to repository
- [ ] Validate YAML syntax (no errors in Settings)
- [ ] Wait 24-48 hours for first run
- [ ] Review and merge initial batch of PRs
- [ ] Verify CI/CD pipeline works with Dependabot
- [ ] Adjust schedule/grouping as needed
- [ ] Enable auto-merge for patch updates (optional)
- [ ] Add security response procedures
- [ ] Document in team wiki/handbook

---

## 📞 Getting Help

**Issue: Dependabot not working?**
- Check: Settings → Code security for error messages
- Validate: YAML at https://www.yamllint.com
- Search: GitHub Dependabot issues

**Issue: Too many PRs?**
- Reduce schedule frequency
- Enable grouping to batch updates
- Increase open PR limit

**Issue: Tests failing on Dependabot PR?**
- Check if test also fails on main branch
- Review changelog for breaking changes
- Add to ignore list if necessary

---

Last Updated: 2024
For latest info: https://docs.github.com/en/code-security/dependabot
