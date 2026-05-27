# Contributing Docs to Knowledge Base

> Author: Your Name (Claude)

Hướng dẫn cho các bots/contributors thêm docs vào KB (`kb.superunderwear.org`).

---

## 1. Cấu trúc một project

Mỗi project cần 1 file `kb-manifest.yml` đặt ở **root của thư mục docs**:

```yaml
project: ten-project          # slug, dùng làm URL path
title: Tên Hiển Thị
description: "Mô tả ngắn"
icon: "🔥"
color:
  light: "#f59e0b"
  dark: "#fbbf24"

# Optional: exclude dirs không cần sync lên KB
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
        link: /ten-project/page-name
```

CI tự scan tất cả `kb-manifest.yml` trong mỗi repo → tạo project trên KB.

---

## 2. Thêm project mới

### Repo đã có trong `repos.yml`:
1. Tạo thư mục docs + các file `.md`
2. Thêm `kb-manifest.yml` vào root thư mục docs
3. Push → CI tự sync + build + deploy

### Repo mới chưa có:
1. Thêm tên repo vào `repos.yml` trong `Tiny-Hive/knowledge-base`
2. Tạo docs + `kb-manifest.yml` như trên
3. Push cả 2 repos

---

## 3. Viết Markdown

VitePress render Markdown thành HTML. Một số lưu ý:

### Tiêu đề
```markdown
# H1 — dùng cho title (1 lần duy nhất)
## H2 — sections chính
### H3 — sub-sections
```

### Code blocks
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

### Hình ảnh
```markdown
![Alt text](./path/to/image.png)
```

Hình ảnh trên KB hỗ trợ **click để zoom** (PhotoSwipe) — không cần config gì thêm.

---

## 4. Excalidraw Diagrams

KB hỗ trợ embed file `.excalidraw` trực tiếp. CI tự convert sang PNG khi build.

### Cách dùng:

1. **Tạo diagram** bằng [excalidraw.com](https://excalidraw.com) hoặc VS Code extension
2. **Save file** `.excalidraw` vào thư mục docs (ví dụ `diagrams/my-diagram.excalidraw`)
3. **Embed trong Markdown** — dùng đuôi `.excalidraw`, KHÔNG phải `.png`:

```markdown
![Architecture Diagram](./diagrams/my-diagram.excalidraw)
```

4. **Push** → CI tự động:
   - Convert `.excalidraw` → `.png` (cùng path, cùng tên)
   - Rewrite markdown refs từ `.excalidraw)` → `.png)`
   - Build + deploy

### Lưu ý:
- **KHÔNG cần commit file `.png`** — CI tạo tự động
- **KHÔNG sửa ref thành `.png`** trong markdown — để `.excalidraw`, CI rewrite
- Nếu cần update diagram: sửa file `.excalidraw`, push lại, CI re-convert
- File `.excalidraw` là JSON text — git diff friendly, mergeable

### Ví dụ cấu trúc:
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

## 5. Exclude directories

Nếu project có thư mục không muốn sync lên KB (build artifacts, test data, etc.), thêm `exclude` trong `kb-manifest.yml`:

```yaml
exclude:
  - node_modules
  - dist
  - reports
  - __pycache__
```

CI dùng `rsync --exclude` nên pattern match tên thư mục.

---

## 6. Access Control

### Policy types:
- `public` — ai cũng xem được
- `restricted` — chỉ users trong `allowed_emails` hoặc `allowed_groups`

### Groups:
Groups được define trong `groups.yml` ở repo `knowledge-base`:
```yaml
core-team:
  - user1@gmail.com
  - user2@gmail.com
```

Dùng group trong manifest:
```yaml
access:
  policy: restricted
  allowed_groups:
    - core-team
```

### Thêm user mới:
- Thêm email vào `allowed_emails` trong `kb-manifest.yml` của project
- Hoặc thêm vào group trong `groups.yml`

---

## 7. CI Pipeline

```
Push to source repo
  → repository_dispatch → knowledge-base CI
    → Pull: clone repo, find kb-manifest.yml, rsync docs
    → Build: generate config, convert excalidraw, vitepress build
    → Deploy: docker build + push GHCR, rollout restart K8s
```

### Trigger thủ công:
```bash
gh workflow run "Pull docs from source repos" \
  -R Tiny-Hive/knowledge-base \
  -f repo=all        # hoặc tên repo cụ thể
```

---

## 8. Troubleshooting

| Vấn đề | Nguyên nhân | Fix |
|---------|-------------|-----|
| Build fail "Could not resolve" | Markdown ref file không tồn tại | Check path, đảm bảo file có trong repo và không bị exclude |
| Excalidraw không convert | File `.excalidraw` bị exclude | Bỏ thư mục chứa nó khỏi `exclude` list |
| Docs không update | CI chưa trigger | Chạy dispatch thủ công hoặc check `repos.yml` |
| 403 trên KB | Email chưa có trong access list | Thêm vào `allowed_emails` hoặc group |
| Ảnh bị stretch khi zoom | Browser cache | Hard refresh (Ctrl+Shift+R) |
