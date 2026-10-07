# GitHub Pages deployment

This repository contains the already-built static site. There is no npm install
or build step here.

## Enable GitHub Pages

After pushing the files, open **Settings → Pages → Build and deployment** in
GitHub and select:

- **Source:** Deploy from a branch
- **Branch:** `main`
- **Folder:** `/ (root)`

Save and wait for the Pages deployment to finish in the **Actions** tab.
The site should be available at:

- https://yaroxor.github.io/armaturen-pages-deploy/
- https://yaroxor.github.io/armaturen-pages-deploy/vkm9-pg/
- https://yaroxor.github.io/armaturen-pages-deploy/kp1/

The root `.nojekyll` file disables Jekyll processing, so GitHub Pages publishes
the `_app/` directory containing the JavaScript and CSS. Without it, the home
page may show HTML while the interactive chamber pages fail to load.
See [GitHub's publishing instructions](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site).

The HTML uses relative asset paths and computes the SvelteKit base URL from the
current location, so the repository URL prefix is already supported. Use the
directory URLs above, including the trailing slash.

Each HTML page also allows the exact hash of SvelteKit's accessibility announcer
style through `style-src-attr`. The original policy blocked this inline style.
Stylesheets still use `style-src 'self'`; the script bootstrap hashes are unchanged.

## Apply the fix from another machine

Copy `armaturen-pages-deploy-github-pages-fix.patch` from the machine where the fix was prepared to
your local machine (for example, with `scp` or your usual file transfer tool).
In your local clone, with a clean working tree:

```sh
git switch main
git pull --ff-only origin main
git apply --check /path/to/armaturen-pages-deploy-github-pages-fix.patch
git apply /path/to/armaturen-pages-deploy-github-pages-fix.patch
git diff
git status --short
git add .nojekyll README.md index.html kp1/index.html vkm9-pg/index.html
git commit -m "Fix GitHub Pages static asset publishing"
git push origin main
```

If you prefer review through a pull request, create a branch with
`git switch -c fix/github-pages` before applying the patch, then push that branch
and open a pull request into `main`. Publishing happens after merging to `main`.

## Updating the site

Build the application in its source repository and copy the complete generated
site into this deployment repository, including `_app/`, `index.html`,
`kp1/index.html`, and `vkm9-pg/index.html`. Keep the root `.nojekyll` file and
this README when replacing generated files. Commit and push the HTML and assets
together so their hashed filenames stay in sync.
When rebuilding, ensure the source build's CSP permits the announcer style too:
the HTML changes in this patch would otherwise be overwritten. If SvelteKit
changes that style, its SHA-256 allowance must be recalculated from the new style.

## Local preview

From the parent directory of this clone:

```sh
python3 -m http.server 8000
```

Open `http://localhost:8000/armaturen-pages-deploy/` to preview with the same URL
prefix as GitHub Pages. Check both links, refresh each chamber page, and verify
that the 3D canvas and controls work. Opening HTML directly with `file://` does
not provide a valid module-loading preview.
