# my-wiki — Platform Engineering Portfolio

A Markdown-first professional portfolio and technical knowledge base built with MkDocs Material, hosted on GitHub Pages.

## Publish
The site is live at https://gtamang001.github.io/my-wiki/.

Every push to `main` runs `.github/workflows/deploy.yml`, which builds with `mkdocs build --strict` and deploys to GitHub Pages (Settings → Pages → Source: GitHub Actions). You can also run it manually from the **Actions** tab.

## Local preview
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```
Visit `http://127.0.0.1:8000/`. Check `mkdocs build --strict` before committing.

## Editorial standards
- All examples must be synthetic, sanitized, or independently recreated.
- Never publish employer code, credentials, private endpoints, internal diagrams, or logs.
- Distinguish **production experience**, **evaluation/POC**, and **learning lab**.
- Case studies describe approaches and lessons, not confidential internal deployments.
- Verify all commands and manifests in an isolated test cluster before using them elsewhere.
- Replace placeholder profile links and review all public claims before publishing.

## Site structure
```
docs/
├── index.md, about.md, projects.md, contact.md   # Portfolio
├── case-studies/      # Problem → approach → decisions → lessons
├── troubleshooting/   # Runbooks: symptoms → checks → fix → prevention
├── labs/              # Reproducible hands-on exercises
├── notes/<subject>/   # Reference & study notes (kubernetes/, networking/, aws/ …)
├── blog/posts/        # Dated learning log & vlog posts
└── tags.md            # Auto-generated tag index
templates/             # Starting points for new pages (not published)
examples/              # Manifests referenced by labs
```

## Adding content
1. **Copy a template** from `templates/` into the right folder:

    | Writing about… | Template | Destination |
    | --- | --- | --- |
    | An issue you fixed | `troubleshooting.md` | `docs/troubleshooting/<component>-<symptom>.md` |
    | A project or initiative | `case-study.md` | `docs/case-studies/<topic>.md` |
    | A hands-on experiment | `lab.md` | `docs/labs/<topic>.md` |
    | Study / reference material | `note.md` | `docs/notes/<subject>/<topic>.md` |
    | A dated update or vlog | `blog-post.md` | `docs/blog/posts/YYYY-MM-DD-<slug>.md` |

2. **Fill in the front matter**: a one-sentence `description` and `tags`. Reuse existing tags so the tag index stays tidy.
3. **Add the page to `nav`** in `mkdocs.yml` and to its section's `index.md` table. Blog posts are picked up automatically.
4. **Preview** with `mkdocs serve`, then run `mkdocs build --strict`.
5. **Commit and push** to `main`; the site redeploys automatically.

## Conventions
- File names are lowercase kebab-case; URLs follow the file path, so avoid renaming published pages. If you must, add an entry to `redirects` in `mkdocs.yml`.
- Blog categories are limited to: Learning Log, Vlog, Troubleshooting, Exam Prep (see `categories_allowed`).
- Put durable knowledge from a blog post into `notes/` or `troubleshooting/` and link to it. The blog is the timeline; the knowledge base is the reference.
- "Last updated" dates come from git history automatically.
