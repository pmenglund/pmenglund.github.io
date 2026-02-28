# Blog Publishing Instructions

These instructions apply to this repository.

## Before Publishing A New Post

1. Re-read the target post file in `content/posts/` before editing if there has been any pause between turns.
2. Validate frontmatter:
   - `title` is set
   - `slug` is set and stable
   - `draft: false`
   - `date` is not in the future at publish time (future dates are excluded from the live site)
3. Validate content accuracy:
   - commands and paths are correct
   - links resolve and point to the intended repo/page
4. Run build:
   - `task build`
5. Verify the post is included in generated output:
   - `rg -n "<slug>|<title snippet>" public/index.html public/posts/index.html -S`
   - optionally verify the rendered page path exists under `public/YYYY/MM/DD/<slug>/index.html`
6. Review git state:
   - `git status --short`
   - ensure only intended files are staged

## Commit And Push

1. Stage and commit the intended files with a clear message.
2. Push to the active branch (usually `master`): `git push origin master`
3. After push, confirm GitHub Pages deployment completes before expecting the post on `https://blog.englund.nu/`.

## Review Readiness

- Required generators:
  - `None.`

- Required validation:
  - `task build`

- Notes:
  - For post visibility checks, use the slug or title snippet in `public/index.html` and `public/posts/index.html`.
