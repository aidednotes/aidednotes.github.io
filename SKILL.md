# SKILL.md — Hugo + GitHub Pages Blog Setup

## Role

You are an assistant working in a local development environment.
Your task is to set up and maintain a Hugo-based technical blog deployed with GitHub Pages.

The blog is for technical notes about:

- AI-driven engineering design
- CAE
- simulation
- control theory
- signal processing
- experimental validation
- interactive engineering demos

The working title of the blog is:

**AI-Driven Engineering Design Notes**

The short name may be:

**AIED Notes**

## Primary Goal

Create a clean, maintainable Hugo blog that can be deployed to GitHub Pages.

The site should support:

- Japanese and English articles
- article-local image management
- technical diagrams and plots
- code snippets
- math notation
- interactive JavaScript or TypeScript demos embedded in articles
- future expansion without messy file organization

## Preferred Technology Stack

Use the following unless there is a good reason not to:

- Hugo Extended
- GitHub Pages
- GitHub Actions for deployment
- Hugo theme: PaperMod
- Markdown content
- Page Bundle structure for posts
- JavaScript or TypeScript for interactive demos

Do not use a backend server.
The final site must be a static site.

## Repository Assumptions

The repository is hosted on GitHub.

Use SSH remotes, not HTTPS, when configuring Git remote URLs.

Example SSH remote pattern:

```bash
git@github-pages-blog-tiramisu:aidednotes/<repo-name>.git
```

The exact repository name should be detected from the current repo if possible.
If no repository exists yet, ask the user before creating or pushing to a new remote.

## Site Structure

Use this structure:

```text
.
├─ config/
│  └─ _default/
│     ├─ hugo.yaml
│     ├─ languages.yaml
│     ├─ params.yaml
│     └─ menus.yaml
├─ content/
│  ├─ posts/
│  │  └─ YYYY-MM-topic-name/
│  │     ├─ index.ja.md
│  │     ├─ index.en.md
│  │     ├─ cover.png
│  │     ├─ fig-01-description.png
│  │     ├─ fig-02-description.jpg
│  │     └─ demo/
│  │        ├─ index.html
│  │        ├─ main.js
│  │        └─ style.css
│  ├─ _index.ja.md
│  └─ _index.en.md
├─ static/
│  ├─ favicon.png
│  ├─ images/
│  │  └─ site-logo.png
│  └─ demos/
│     └─ example-demo/
│        ├─ index.html
│        ├─ main.js
│        └─ style.css
├─ assets/
│  └─ css/
├─ themes/
│  └─ PaperMod/
└─ .github/
   └─ workflows/
      └─ hugo.yaml
```

## Content Organization Rules

### Use Page Bundles for Articles

Each article must live in its own folder.

Good:

```text
content/posts/2026-05-gibbs-phenomenon/
├─ index.ja.md
├─ index.en.md
├─ cover.png
├─ fig-01-fourier-series.png
└─ fig-02-gibbs-overshoot.png
```

Bad:

```text
content/posts/gibbs.md
static/images/gibbs1.png
static/images/gibbs2.png
```

Reason:
Article-specific images should stay next to the article to avoid broken links and messy global image folders.

### Multilingual Articles

Use this naming convention:

```text
index.ja.md
index.en.md
```

Japanese drafts are usually written first.
English versions may be translated from Japanese.

Keep both language versions in the same Page Bundle folder so they can share images.

### Image Naming Rules

Use simple ASCII filenames only.

Good:

```text
cover.png
fig-01-system-overview.png
fig-02-cae-result.jpg
fig-03-validation-setup.png
```

Avoid:

```text
解析結果１.png
スクリーンショット 2026-04-30.png
IMG_1234.JPG
image final final.png
```

Use lowercase words separated by hyphens.

### Image Placement Rules

Use article-local images for:

- figures
- plots
- CAE results
- screenshots
- diagrams
- article-specific illustrations

Use `static/images/` only for:

- site logo
- favicon
- common profile images
- shared OGP images

Do not put article-specific figures in `static/images/`.

## Hugo Configuration

Use YAML configuration files under:

```text
config/_default/
```

Recommended files:

```text
hugo.yaml
languages.yaml
params.yaml
menus.yaml
```

### Basic `hugo.yaml`

```yaml
baseURL: "https://aidednotes.github.io/"
title: "AI-Driven Engineering Design Notes"
theme: "PaperMod"
defaultContentLanguage: "ja"
defaultContentLanguageInSubdir: false

enableRobotsTXT: true
enableGitInfo: true

pagination:
  pagerSize: 10

markup:
  goldmark:
    renderer:
      unsafe: true
  highlight:
    noClasses: false
```

If the site is deployed as a project page, adjust `baseURL`:

```yaml
baseURL: "https://aidednotes.github.io/<repo-name>/"
```

Do not guess the final `baseURL`.
Check the repository and deployment target.

### Basic `languages.yaml`

```yaml
ja:
  languageName: "日本語"
  weight: 1
  title: "AI-Driven Engineering Design Notes"
  contentDir: "content"

en:
  languageName: "English"
  weight: 2
  title: "AI-Driven Engineering Design Notes"
  contentDir: "content"
```

### Basic `params.yaml`

```yaml
env: production
description: "Notes on AI-driven engineering design, CAE, simulation, control, signal processing, and validation."
author: "AIED Notes"

defaultTheme: auto
ShowReadingTime: true
ShowShareButtons: true
ShowPostNavLinks: true
ShowBreadCrumbs: true
ShowCodeCopyButtons: true
ShowToc: true
TocOpen: false

homeInfoParams:
  Title: "AI-Driven Engineering Design Notes"
  Content: "Notes on AI-driven engineering design, simulation, CAE, control, signal processing, and validation."

socialIcons:
  - name: github
    url: "https://github.com/aidednotes"
  - name: x
    url: "https://x.com/<x-account>"
```

Do not invent the X account URL.
Leave a placeholder if unknown.

## Theme

Use PaperMod as the initial theme.

Recommended setup:

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

If the repository already has a theme, inspect it before replacing anything.

Do not delete an existing theme or config without making a backup or confirming with the user.

## Deployment

Use GitHub Actions.

Create:

```text
.github/workflows/hugo.yaml
```

Recommended workflow:

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.125.7
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Setup Hugo
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

After pushing this workflow, the user may need to enable GitHub Pages deployment source:

GitHub repository → Settings → Pages → Build and deployment → Source → GitHub Actions

## Interactive Demos

The blog should support small browser-based demos.

Examples:

- Fourier series and Gibbs phenomenon
- Bode plot demo
- PID response demo
- Nyquist plot demo
- aliasing demo
- sampling theorem demo
- simple ODE simulation demo

### Recommended Demo Placement

For simple demos shared across articles:

```text
static/demos/gibbs/
├─ index.html
├─ main.js
└─ style.css
```

Embed in Markdown:

```html
<iframe
  src="/demos/gibbs/"
  width="100%"
  height="560"
  loading="lazy"
  style="border:1px solid #ddd; border-radius:8px;">
</iframe>
```

For demos that belong to one article only, either:

1. Place them under the article Page Bundle and test carefully, or
2. Prefer `static/demos/<demo-name>/` for stable iframe paths.

For reliability, `static/demos/` is preferred.

### Demo Technology

Use plain JavaScript for small demos.

Use TypeScript when:

- the demo has multiple source files
- numerical logic becomes complex
- reusable modules are useful

For plotting:

- Canvas for simple, fast custom drawing
- SVG for clean vector diagrams
- Plotly.js for interactive scientific plots
- Observable Plot if appropriate

Avoid heavy frameworks unless needed.

## Math Support

Enable math rendering if the theme/config supports it.

Prefer KaTeX for math.

If adding math support, make it site-wide but lightweight.
Do not break existing Markdown rendering.

## Article Template

Create archetypes for posts if helpful.

Recommended article structure:

```markdown
---
title: "Article Title"
date: YYYY-MM-DD
draft: true
tags: ["AI", "CAE", "Simulation"]
categories: ["Engineering Design"]
cover:
  image: "cover.png"
  alt: "Cover image"
  caption: ""
---

## Background

Explain the problem and why it matters.

## Approach

Explain the method, model, simulation, experiment, or design workflow.

## Demo or Implementation

Include code, figures, or an embedded interactive demo.

## Results

Show plots, screenshots, or key observations.

## Discussion

Discuss limitations, assumptions, and engineering interpretation.

## Conclusion

Summarize what was learned.
```

## First Sample Article

Create a sample article if the site is empty:

```text
content/posts/2026-05-gibbs-phenomenon/
├─ index.ja.md
├─ index.en.md
├─ cover.png
```

The sample article can introduce a future Gibbs phenomenon demo.
Do not overcomplicate it.

## Git Rules

Before making changes:

```bash
git status
```

Do not overwrite unrelated user changes.

After meaningful changes:

```bash
hugo --gc --minify
```

Then:

```bash
git status
```

Make a clear summary of changed files.

Do not push unless the user explicitly asks you to push.

## Local Development Commands

Common commands:

```bash
hugo server -D
```

Build:

```bash
hugo --gc --minify
```

Initialize a new site:

```bash
hugo new site <site-name>
```

Create a new article bundle manually:

```bash
mkdir -p content/posts/YYYY-MM-topic-name
touch content/posts/YYYY-MM-topic-name/index.ja.md
touch content/posts/YYYY-MM-topic-name/index.en.md
```

## Completion Criteria

A setup task is complete when:

1. Hugo builds successfully without errors.
2. The local site runs with `hugo server -D`.
3. The folder structure supports multilingual articles.
4. PaperMod is configured.
5. Article images can be stored in article Page Bundles.
6. GitHub Actions workflow exists for GitHub Pages deployment.
7. No secrets or private SSH keys are committed.
8. The final summary clearly explains:
   - files created or modified
   - commands run
   - remaining manual steps, if any

## Security Rules

Never commit:

- private SSH keys
- `.env` files with secrets
- API keys
- personal tokens
- local machine-specific config

Check before commit:

```bash
git status
git diff --cached
```

If a private key or secret appears, stop immediately.

## Important Notes for This User

The user writes Japanese first and may translate to English later.

Prefer workflows that support:

```text
Japanese draft → English translation → shared figures → published bilingual article
```

The user has previously had trouble managing JPG/PNG files.
Therefore, prioritize Page Bundles and strict image naming rules.

The user may later add interactive educational demos.
Therefore, keep the site structure friendly to static JavaScript/TypeScript demos.

## Do Not Do

- Do not use WordPress.
- Do not require a backend server.
- Do not require paid hosting.
- Do not scatter article images in global folders.
- Do not assume the final GitHub Pages URL without checking the repo settings.
- Do not push to GitHub without explicit permission.
- Do not delete existing files unless clearly safe or confirmed.
