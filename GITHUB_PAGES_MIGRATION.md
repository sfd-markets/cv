# GitHub Pages Migration

This site is configured as a **user/org GitHub Pages** site (`baseurl: ""`). Deploy it to `sfd-markets.github.io` so URLs are:

| URL | Page |
|-----|------|
| `https://sfd-markets.github.io/` | Landing |
| `https://sfd-markets.github.io/cv/` | Resume |
| `https://sfd-markets.github.io/portfolio/` | Projects listing |
| `https://sfd-markets.github.io/portfolio/vps-devops-lab/` | VPS project detail |

## Steps

1. **Create the user site repo** (if it does not exist):
   - Repository name must be `sfd-markets.github.io` under the `sfd-markets` org or account.

2. **Push this site** to the `main` branch of that repo:
   ```bash
   cd projects/cv
   git remote add pages git@github.com:sfd-markets/sfd-markets.github.io.git  # if needed
   git push pages main
   ```
   Or copy the repo contents into an existing `sfd-markets.github.io` checkout and push.

3. **Enable GitHub Pages**: Repo Settings → Pages → Build and deployment → Source: **Deploy from a branch** → Branch: `main`, folder: `/ (root)`.

4. **Retire the old project site** (`sfd-markets/cv`):
   - Archive the repo, or update its README to point visitors to `https://sfd-markets.github.io/cv/`.

5. **Verify** after the Pages build completes:
   - Landing, CV, portfolio listing, and VPS detail pages load.
   - Header tabs (Resume | Projects) appear on every page with correct active state.
   - Old URLs redirect via `jekyll-redirect-from`:
     - `/cv/portfolio/` → `/portfolio/`
     - `/cv/portfolio/oci-devops-lab/` → `/portfolio/vps-devops-lab/`
     - `/portfolio/oci-devops-lab/` → `/portfolio/vps-devops-lab/`

## Local preview

```bash
bundle exec jekyll serve
```

Open `http://localhost:4000/` — no `/cv` prefix in local URLs.
