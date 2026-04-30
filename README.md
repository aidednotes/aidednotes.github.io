# AIED Notes (Hugo + GitHub Pages)

Technical notes for AI-driven engineering design, CAE, simulation, control, signal processing, and experimental validation.

## Local development

This repo vendors:

- Hugo Extended: `bin/hugo` (not committed; ignored by `.gitignore`)
- Theme: `themes/PaperMod/` (snapshot extracted from upstream)

Commands:

```bash
./bin/hugo --gc --minify
./bin/hugo server -D
```

## GitHub Pages (GitHub Actions)

Workflow: `.github/workflows/hugo.yaml`

After pushing to `main`, enable:

Repository Settings -> Pages -> Build and deployment -> Source -> GitHub Actions

The workflow passes the Pages base URL to Hugo via `--baseURL`, so `config/_default/hugo.yaml` keeps `baseURL` as a local default.

## Content structure

Posts are Page Bundles under `content/posts/YYYY-MM-topic-name/`:

```text
content/posts/2026-05-gibbs-phenomenon/
├─ index.ja.md
└─ index.en.md
```

Put article-local images next to the `index.*.md` files (ASCII, lowercase, hyphenated filenames).

