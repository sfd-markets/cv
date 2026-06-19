# GitHub Pages Migration

This site is configured as a **user/org GitHub Pages** site (`baseurl: ""`). Deploy it to `sfd-markets.github.io` so URLs are:

| URL | Page |
|-----|------|
| `https://sfd-markets.github.io/` | Landing |
| `https://sfd-markets.github.io/cv/` | Resume |
| `https://sfd-markets.github.io/portfolio/` | Projects listing |
| `https://sfd-markets.github.io/portfolio/vps-devops-lab/` | VPS project detail |

## Why one org site repo is required

GitHub currently publishes from **separate project repos**:

| Repo | URL path | Problem |
|------|----------|---------|
| `sfd-markets/cv` | `/cv/` | Project-site prefix stacks with page permalinks (`/cv/cv/`) |
| `sfd-markets/portfolio` | `/portfolio/` | Old placeholder content, not this Jekyll site |
| *(none)* | `/` | Root returns 404 |

A single `sfd-markets.github.io` repo with `baseurl: ""` serves `/`, `/cv/`, and `/portfolio/` from one site — no `/cv/cv/` double path.

## Steps

### 1. Create the org site repo

1. On GitHub under the `sfd-markets` org: **New repository**
2. Name: `sfd-markets.github.io` (exact name required)
3. Visibility: Public (required for free GitHub Pages)

### 2. Push this site to the org repo

```bash
cd projects/cv
git remote add pages git@github.com:sfd-markets/sfd-markets.github.io.git  # if needed
git push pages main
```

Or clone `sfd-markets.github.io` fresh, copy Jekyll files from `projects/cv` (not `_site/`, `vendor/`), commit, and push.

### 3. Enable Pages on the org repo

`sfd-markets.github.io` → **Settings → Pages**:

- Source: **Deploy from a branch**
- Branch: `main`, folder: `/ (root)`
- Wait 1–3 minutes for the build

### 4. Disable conflicting project repos (required)

While `sfd-markets/cv` and `sfd-markets/portfolio` publish Pages, they **reserve** `/cv/` and `/portfolio/` and block the org site from serving pages at those paths.

For each repo: **Settings → Pages → set source to None** (disable Pages).

| Repo | Action | Why |
|------|--------|-----|
| `sfd-markets/cv` | Disable Pages | Frees `/cv/` for org site resume |
| `sfd-markets/portfolio` | Disable Pages | Frees `/portfolio/` for VPS listing |

**Order:** push org site first, then disable project Pages.

Update each retired repo's README to point visitors to:

- Resume: `https://sfd-markets.github.io/cv/`
- Projects: `https://sfd-markets.github.io/portfolio/`

Optional: archive the repos after disabling Pages.

### 5. Verify after deploy

| Check | URL | Pass criteria |
|-------|-----|---------------|
| Landing | `https://sfd-markets.github.io/` | Landing content, styled |
| Resume | `https://sfd-markets.github.io/cv/` | Resume (not landing), not `/cv/cv/` |
| Projects | `https://sfd-markets.github.io/portfolio/` | VPS listing (not old placeholder) |
| VPS detail | `https://sfd-markets.github.io/portfolio/vps-devops-lab/` | VPS architecture summary |
| CSS | `https://sfd-markets.github.io/assets/css/style.css` | 200 OK |
| Nav tabs | All pages | Resume / Projects tabs with active state |
| Old URL | `https://sfd-markets.github.io/cv/cv/` | Redirects to `/cv/` |

Old URLs redirect via `jekyll-redirect-from`:

- `/cv/cv/` → `/cv/`
- `/cv/portfolio/` → `/portfolio/`
- `/cv/portfolio/oci-devops-lab/` → `/portfolio/vps-devops-lab/`
- `/portfolio/oci-devops-lab/` → `/portfolio/vps-devops-lab/`

## Local preview

```bash
bundle exec jekyll serve
```

| Local URL | Page |
|-----------|------|
| `http://localhost:4000/` | Landing |
| `http://localhost:4000/cv/` | Resume |
| `http://localhost:4000/portfolio/` | Projects listing |
| `http://localhost:4000/portfolio/vps-devops-lab/` | VPS detail |
