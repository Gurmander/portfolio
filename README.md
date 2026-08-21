# Gurmander Singh Maan Portfolio

This repository contains a personal portfolio and technical blog built with [Hugo](https://gohugo.io/). It presents profile information, selected software and machine learning projects, a resume, and long-form technical writing.

The site is generated from Hugo templates, Markdown content, and YAML data. [Pagefind](https://pagefind.app/) adds static, client-side search after Hugo builds the HTML.

## Requirements

- Hugo Extended
- Node.js and npm

Install the JavaScript dependency used for search:

```bash
npm install
```

## Local development

Start Hugo's development server:

```bash
npm run dev
```

Open `http://localhost:1313/`. Hugo watches content, layouts, data, CSS, and static files and rebuilds the site when they change.

This runs `hugo server` and is the fastest option while editing the site. Pagefind cannot index Hugo's in-memory development output, so search is not available in this mode. Use the indexed preview workflow below when testing search.

## Production build

Build the site first, then generate the search index:

```bash
hugo --minify --baseURL https://gurmander.github.io/portfolio/
npx pagefind --site public
```

Hugo writes the generated website to `public/`. Pagefind then scans those generated HTML files and writes its search index and browser assets to `public/pagefind/`.

The order matters: Pagefind must run after Hugo because it indexes Hugo's output. Run both commands after changing content or templates and before deploying. The deployment platform must publish the final `public/` directory. GitHub Actions supplies the repository's actual Pages URL automatically, so the hard-coded URL above is only needed for a manual production build.

## Preview with Pagefind

To test the production build and search locally:

```bash
npm run preview
```

Open the URL printed by Pagefind, normally `http://localhost:1414/`. This command builds Hugo with a localhost URL, generates the Pagefind index, and serves both together.

Pagefind is used because this is a static website with no search backend. It creates a compact browser-side index from the generated pages. The search route is defined by `content/search.md`, while `layouts/search/single.html` loads the Pagefind UI and initializes it.

## Pages and content

### Home and About

The homepage and About section are rendered by `layouts/index.html`.

- Edit the name, tagline, social links, email, and resume URL in `data/site.yaml`.
- Add or replace profile photos in `static/images/profile_images/`.
- Update the About text or homepage section order in `layouts/index.html`.

The About navigation link points to the `#about` section on the homepage; it is not a separate content page.

### Projects

Project information is stored in `data/projects.yaml`. The homepage displays a compact project list, while `layouts/projects/list.html` renders the full project details at `/projects/`. The `content/projects/_index.md` file defines that section's title and description.

Add a project by adding another YAML entry to `data/projects.yaml` using the existing fields for title, achievement, description, technologies, and external links.

### Blog

Blog posts live under `content/blog/<year>/` as Markdown files. Each post uses YAML front matter for its title, date, description, categories, tags, and draft status.

Create a draft from the included archetype:

```bash
hugo new content blog/2026/my-post.md
```

Edit the generated file and set `draft: false` when it is ready to publish. The blog section metadata is in `content/blog/_index.md`; list and article rendering are handled by `layouts/_default/list.html` and `layouts/_default/single.html`.

### Static files

Files under `static/` are copied to the root of the generated site:

- `static/images/` becomes `/images/`.
- `static/resume/` becomes `/resume/`.

For example, `static/resume/Gurmander_Singh_Maan.pdf` is linked as `/resume/Gurmander_Singh_Maan.pdf`.

## Project structure

| Path | Purpose |
| --- | --- |
| `archetypes/` | Default front matter for newly created content |
| `assets/css/` | CSS processed by Hugo's asset pipeline |
| `content/` | Markdown pages and blog posts |
| `data/` | Profile and project data used by templates |
| `layouts/` | Hugo page templates and shared partials |
| `static/` | Images, resume, and other files copied unchanged |
| `hugo.toml` | Hugo site, taxonomy, and Markdown configuration |
| `package.json` | Pagefind development dependency |
| `public/` | Generated Hugo site and Pagefind index; not committed |

## Generated and local files

The `.gitignore` excludes reproducible or machine-specific files, including:

- `public/` and Hugo-generated resources
- `node_modules/`
- Hugo's build lock and statistics files
- local `.env` files, while allowing `.env.example`
- operating-system and editor metadata

Keep `package-lock.json` committed so Pagefind installs reproducibly.
