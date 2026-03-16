# rs-station.github.io-preview

This repository hosts live preview deployments of pull requests from [rs-station.github.io](https://github.com/rs-station/rs-station.github.io). 

Every time a pull request is opened or updated, a GitHub Actions workflow automatically builds and deploys a preview version of the site here, at a unique URL tied to the PR number.

---

## How it works

### Overview

The main site repository contains a [GitHub Actions workflow](https://github.com/rs-station/rs-station.github.io/blob/main/.github/workflows/build_only.yml) that that builds and deploys previews when PRs are opened or updated.

### The deploy workflow

When a pull request is opened or updated against the `main` branch of the main repository, a workflow runs that:

1. **Checks out the PR branch** using `actions/checkout`.
2. **Sets up Ruby** using `ruby/setup-ruby`, which installs the correct Ruby version and runs `bundle install` automatically via the `bundler-cache: true` option.
3. **Builds the Jekyll site** using `bundle exec jekyll build`, passing a `--baseurl` flag so that all internal links and asset paths are correct for the preview subdirectory. The baseurl is set to `/pr-previews/pr-{number}`, where `{number}` is the PR number.
4. **Adds a `.nojekyll` file** to the built output. This tells GitHub Pages not to try to process the branch as a Jekyll site itself, which would otherwise cause it to ignore folders starting with underscores.
5. **Deploys to this repository** using `peaceiris/actions-gh-pages`, authenticated via a deploy key (see below). It pushes the contents of the `_site/` build output into a subfolder called `pr-previews/pr-{number}` on this repository's `gh-pages` branch. The `keep_files: true` option ensures that deploying one PR's preview does not overwrite previews from other open PRs.
6. **Posts a comment on the PR** with the live preview URL, using `actions/github-script`.

### The cleanup workflow

When a pull request is closed (whether merged or not), a second workflow runs that:

1. Checks out this repository's `gh-pages` branch directly, using the deploy key for authentication.
2. Deletes the `pr-previews/pr-{number}` subfolder.
3. Commits and pushes the deletion, so stale previews do not accumulate indefinitely.

### Authentication: deploy key

The workflows authenticate to this repository using an **SSH deploy key**.

The private half of the deploy key is stored as a secret (`PREVIEW_DEPLOY_KEY`) in the main repository. The public half is registered as a deploy key in this repository's settings with write access enabled.

### Why a separate repository?

The main site uses **GitHub Actions** as its Pages deployment source (rather than deploying directly from a branch). This mode is designed for a single canonical deployment via the official `actions/deploy-pages` action, and does not support deploying multiple simultaneous previews to subfolders.

By using a separate repository — this one — with Pages configured to deploy from the `gh-pages` **branch**, we get full control over the folder structure and can maintain as many simultaneous PR previews as needed without interfering with the main site's deployment pipeline.

---

## URL structure

Preview URLs follow this pattern:

```
https://{org}.github.io/{this-repo}/pr-previews/pr-{number}/
```

The comment posted automatically on each PR will include the exact URL.

---

## Repository settings

For this setup to work, this repository must have:

- **GitHub Pages enabled**, with source set to **"Deploy from a branch"** → `gh-pages` branch → `/ (root)`.
- The **public deploy key** registered under Settings → Deploy keys, with write access enabled.


**Old previews not being cleaned up**: Check the cleanup workflow run in the main repository's Actions tab. If the `gh-pages` branch checkout step fails, it is likely an authentication issue with the deploy key.
