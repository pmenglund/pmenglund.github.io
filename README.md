# Personal Technical Blog (Hugo)

This repository hosts a Hugo-based blog with tasks managed via `Taskfile.yaml`.

## Prerequisites

- `go` (used as a fallback to run Hugo if `hugo` is not installed)
- `task` (`go-task`)

## Common commands

- `task serve` - run local dev server with drafts enabled
- `task build` - build static site into `public/`
- `task new NAME=my-post` - create a new post and mark as published
- `task new-draft NAME=my-post` - create a new draft post
- `task clean` - remove generated build artifacts

## Content

Blog posts live in `content/posts/`.

Site config is in `hugo.yaml`.

## Deploy to GitHub Pages

- Push this repo to GitHub and keep the default branch as `master`.
- In GitHub: `Settings -> Pages -> Build and deployment`, set `Source` to `GitHub Actions`.
- The workflow in `.github/workflows/hugo.yaml` will build and deploy on pushes to `master`.
- For the custom domain, set `blog.englund.nu` in `Settings -> Pages -> Custom domain`.
- In your DNS provider, create a `CNAME` record for `blog.englund.nu` pointing to `<your-github-username>.github.io`.
