# Contributing Docs to Knowledge Base

> Author: Your Name (Claude)

Guide for bots and contributors to add documentation to KB (`kb.superunderwear.org`).

---

## 1. Project Structure

Each project needs a `kb-manifest.yml` file at the **root of its docs directory**:

```yaml
project: my-project            # slug, used as URL path
title: My Project
description: "Short description"
icon: "🔥"
color:
  light: "#f59e0b"
  dark: "#fbbf24"

# Optional: exclude dirs from KB sync
exclude:
  - node_modules
  - .github

access:
  policy: restricted           # restricted | public
  allowed_groups:
    - core-team
  allowed_emails:
    - user@example.com

sidebar:
  - text: Section Name
    items:
      - text: Page Title
        link: /my-project/page-name
```

CI auto-scans all `kb-manifest.yml` files in each repo and creates projects on KB.

---

## 2. Adding a New Project

### Repo already listed in `repos.yml`:
1. Create a docs directory with `.md` files
2. Add `kb-manifest.yml` to the docs root
3. Push — CI auto-syncs, builds, and deploys

### New repo not yet listed:
1. Add the repo name to `repos.yml` in `Tiny-Hive/knowledge-base`
2. Create docs + `kb-manifest.yml` as above
3. Push both repos

---

## 3. Writing Markdown

VitePress renders Markdown to HTML. Key conventions:

### Headings
```markdown
# H1 — page title (use once)
## H2 — main sections
### H3 — sub-sections
```

### Code Blocks
````markdown
```yaml
key: value
```

```bash
kubectl get pods
```

```typescript
const x: number = 42
```
````

### Images
```markdown
![Alt text](./path/to/image.png)
```

Images on KB support **click-to-zoom** (PhotoSwipe) — no extra config needed.

---

## 4. Excalidraw Diagrams

KB supports embedding `.excalidraw` files directly. CI auto-converts them to PNG at build time.

### Usage:

1. **Create a diagram** using [excalidraw.com](https://excalidraw.com) or the VS Code extension
2. **Save the file** as `.excalidraw` in your docs directory (e.g. `diagrams/my-diagram.excalidraw`)
3. **Embed in Markdown** — use the `.excalidraw` extension, NOT `.png`:

```markdown
![Architecture Diagram](./diagrams/my-diagram.excalidraw)
```

4. **Push** — CI automatically:
   - Converts `.excalidraw` → `.png` (same path, same name)
   - Rewrites markdown refs from `.excalidraw)` → `.png)`
   - Builds and deploys

### Important:
- **Do NOT commit `.png` files** — CI generates them automatically
- **Do NOT change refs to `.png`** in markdown — keep `.excalidraw`, CI rewrites
- To update a diagram: edit the `.excalidraw` file, push, CI re-converts
- `.excalidraw` files are JSON text — git diff friendly, mergeable

### Example structure:
```
my-project/
├── kb-manifest.yml
├── index.md
├── architecture.md          ← ![](./diagrams/system.excalidraw)
└── diagrams/
    ├── system.excalidraw    ← source (commit this)
    └── system.png           ← auto-generated (DON'T commit)
```

---

## 5. Excluding Directories

If your project has directories that shouldn't be synced to KB (build artifacts, test data, etc.), add `exclude` to `kb-manifest.yml`:

```yaml
exclude:
  - node_modules
  - dist
  - reports
  - __pycache__
```

CI uses `rsync --exclude`, so patterns match directory names.

---

## 6. Access Control

### Policy types:
- `public` — visible to everyone
- `restricted` — only users in `allowed_emails` or `allowed_groups`

### Groups:
Groups are defined in `groups.yml` in the `knowledge-base` repo:
```yaml
core-team:
  - user1@gmail.com
  - user2@gmail.com
```

Reference groups in your manifest:
```yaml
access:
  policy: restricted
  allowed_groups:
    - core-team
```

### Adding a new user:
- Add their email to `allowed_emails` in your project's `kb-manifest.yml`
- Or add them to a group in `groups.yml`

---

## 7. CI Pipeline

```
Push to source repo
  → repository_dispatch → knowledge-base CI
    → Pull: clone repo, find kb-manifest.yml, rsync docs
    → Build: generate config, convert excalidraw, vitepress build
    → Deploy: docker build + push GHCR, rollout restart K8s
```

### Manual trigger:
```bash
gh workflow run "Pull docs from source repos" \
  -R Tiny-Hive/knowledge-base \
  -f repo=all        # or a specific repo name
```

---

## 8. Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| Build fail "Could not resolve" | Markdown refs a file that doesn't exist | Check path, ensure file exists and isn't excluded |
| Excalidraw not converting | `.excalidraw` file is in an excluded dir | Remove its directory from the `exclude` list |
| Docs not updating | CI hasn't triggered | Run manual dispatch or check `repos.yml` |
| 403 on KB | Email not in access list | Add to `allowed_emails` or a group |
| Image stretched on zoom | Browser cache | Hard refresh (Ctrl+Shift+R) |
