# my-wiki — Platform Engineering Portfolio

A Markdown-first professional portfolio and technical knowledge base built with MkDocs Material, hosted on GitHub Pages.

## Publish
1. Create a **public** GitHub repository named `my-wiki` with default branch `main`.
2. Copy this repository's contents to it and commit/push.
3. Replace `YOUR_GITHUB_USERNAME` in `mkdocs.yml` with your actual GitHub username.
4. Go to **Settings → Pages → Build and deployment → Source: GitHub Actions**.
5. Open **Actions**, allow the workflow to complete. Your site will be at `https://YOUR_GITHUB_USERNAME.github.io/my-wiki/`.

**Important:** Update `mkdocs.yml` placeholders *before* your first deployment; do not publish a placeholder URL.

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

## Content
Documentation is in `docs/`. Navigation is in `mkdocs.yml`. Add new `.md` pages and link them in `nav`.
