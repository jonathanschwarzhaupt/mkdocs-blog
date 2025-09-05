# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

**Development server:**
```bash
mkdocs serve
```
Starts local development server at http://127.0.0.1:8000/ with live reload

**Build site:**
```bash
mkdocs build
```
Builds static site to `site/` directory

**Deploy:**
```bash
mkdocs gh-deploy
```
Builds and deploys to GitHub Pages

**Environment setup:**
```bash
mise install
```
Installs Python 3.13 and uv via mise tool manager

**Install dependencies:**
```bash
uv sync
```
Installs project dependencies via uv package manager

## Project Architecture

This is a personal blog built with **MkDocs Material** for Jonathan Schwarzhaupt, focusing on data engineering, cloud, and development topics.

### Key Structure:
- **docs/**: All content and assets
  - **blog/**: Blog posts using MkDocs blog plugin
    - **posts/**: Individual blog post markdown files
    - **images/**: Blog-specific images
  - **assets/**: Site-wide assets (profile photo, favicon)
  - **stylesheets/**: Custom CSS overrides
- **overrides/**: Custom template overrides (home.html)
- **mkdocs.yml**: Main site configuration
- **pyproject.toml**: Python project dependencies

### Content Management:
- Blog posts are markdown files in `docs/blog/posts/`
- Each post has frontmatter with date, tags, and categories
- Supported categories: Cloud, Python, Tools, Data Engineering, DevOps, Personal Project
- Images should be placed in `docs/blog/images/` and referenced relatively
- Tags are managed via the tags plugin and displayed on `blog/tags.md`

### Theme Configuration:
- Uses Material for MkDocs with "sage" color scheme
- Custom logo: FontAwesome portable toilets icon
- Navigation includes instant loading, tabs, and table of contents features
- Social links configured for LinkedIn and GitHub
- Code highlighting with line numbers and copy buttons

### Development Environment:
- Uses **mise** for Python and uv tool management
- Python 3.13 required
- **uv** for fast dependency management
- Setup scripts handle devcontainer initialization
- Auto-creates virtual environment via mise

### Content Guidelines:
- Blog posts focus on data engineering, cloud technologies, and development tools
- "Home Plumbing" section refers to personal data platform projects
- Maintains informal, learning-focused tone
- Code snippets use syntax highlighting with multiple language support