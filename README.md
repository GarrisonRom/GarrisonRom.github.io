# GarrisonRom Tech Site

A complete personal technical blog site featuring homepage, articles, documentation center, and admin panel.

## File Structure

```
garrisonrom-site/
├── index.html              # Homepage (Chinese/English bilingual)
├── style.css               # Global styles (GitHub dark theme)
├── papers/                 # Publication PDFs
│   ├── ACL2026_4655_Revealing_Procedural_Reas.pdf
│   ├── ICML2026_9079_The_Geometry_of_Narrow_Fi.pdf
│   ├── KDD2026_3421_Mining_Point_of_No_Return.pdf
│   └── SymPareto_A_Dynamic_GainDegradation_Co-Evolution_Framework_for_Pareto-Optimal_Task_Scheduling_in_Autonomous_Intelligent_Systems.pdf
├── posts/                  # Blog posts
│   ├── post.html           # Blog template (Markdown + KaTeX rendering)
│   └── md/                 # Markdown source files
│       ├── note-01.md      # Cross-Domain Adaptation Notes 01-09
│       ├── note-02.md
│       ├── note-03.md
│       ├── note-04.md
│       ├── note-05.md
│       ├── note-06.md
│       ├── note-07.md
│       ├── note-08.md
│       └── note-09.md
├── docs/                   # Documentation center
│   ├── index.html          # Docs entry
│   ├── projects.html       # Project docs
│   ├── papers.html         # Paper archive
│   ├── tech-spec.html      # Technical specification (ContinualEdge)
│   ├── research-direction.html  # Research directions
│   └── tech-stack.html     # Technology stack
└── admin/                  # Admin panel
    ├── login.html          # Login page
    └── dashboard.html      # Content management dashboard
```

## Usage

### 1. Local Preview
Open `index.html` directly in browser. All pages are static HTML, no server required.

### 2. Deploy to GitHub Pages
1. Create GitHub repository (e.g., `GarrisonRom.github.io`)
2. Push files to repository root
3. Enable GitHub Pages (Settings → Pages → Source: main branch)
4. Access `https://garrisonrom.github.io`

### 3. Admin Panel
- Visit `admin/login.html`
- Default username: `garrison`
- Default password: `scut2026` (recommended to change)
- After login, you can edit: study notes, research directions, tech stack, project status, paper tracking, site configuration
- All data stored in browser localStorage, supports export/import JSON backup

**Note**: Current authentication is frontend-only simulation, suitable for personal local use. For production deployment, add backend authentication.

## Features

- **Static**: No build tools, no dependencies, ready to run
- **Bilingual**: Chinese/English toggle (JavaScript-controlled CSS visibility)
- **Responsive**: Mobile to desktop (2.4" to desktop)
- **Dark Theme**: GitHub-style color scheme, eye-friendly and professional
- **Math Support**: KaTeX rendering for mathematical formulas in blog posts
- **Markdown Support**: Markdown to HTML conversion for blog content

## Customization Guide

### Modify Personal Info
Edit `admin/dashboard.html` or login to admin panel to modify site settings.

### Add New Articles
1. Add Markdown file in `posts/md/`
2. Update `posts/post.html` to include the new post
3. Add link in `index.html` #posts section

### Change Password
Edit `ADMIN_PASS` constant in `admin/login.html`.

## Color Reference

| Token | Value | Usage |
|-------|-------|-------|
| `--bg` | `#0d1117` | Main background |
| `--bg-secondary` | `#161b22` | Card background |
| `--border` | `#30363d` | Border |
| `--text` | `#c9d1d9` | Main text |
| `--text-heading` | `#f0f6fc` | Headings |
| `--blue` | `#58a6ff` | Accent/links |
| `--green` | `#3fb950` | Success/Accepted |
| `--orange` | `#f0883e` | Warning/Submitted |
| `--red` | `#f85149` | Error/Rejected |
| `--purple` | `#a371f7` | Highlight/Motto |

---
Built by Xavier Luo (Jiaxin Luo).
