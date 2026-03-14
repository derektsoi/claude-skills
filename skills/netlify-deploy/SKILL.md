---
name: netlify-deploy
description: Deploy HTML prototypes and static files to Netlify for instant shareable URLs. This skill should be used when the user says "deploy to Netlify", "host this prototype", "publish this HTML", "make this shareable", "give me a URL for this", or wants to put any static HTML/CSS/JS files on the web. Also trigger when the user asks to update an existing Netlify deployment, list their deployed sites, or manage Netlify projects. Even if they don't mention "Netlify" explicitly, trigger for any request to host or share HTML prototypes online.
---

# Netlify Deploy

Deploy static HTML prototypes to Netlify and get shareable URLs. Designed for PMs and non-engineers who need to quickly share mockups, prototypes, or static pages with stakeholders.

## Prerequisites Check

Before deploying, verify the user's Netlify setup. Run these checks in order and stop at the first failure.

### 1. Check Netlify CLI availability

```bash
npx netlify --version
```

If this fails, Node.js is required. Suggest: `brew install node` (macOS) or https://nodejs.org.

### 2. Check authentication

```bash
npx netlify status
```

**If already authenticated**: Confirm the logged-in identity and proceed to step 3.

**If NOT authenticated** (or the user is setting up for the first time): Walk them through account creation and login. This guidance matters because many users are PMs, not engineers — be explicit:

1. Go to https://app.netlify.com and sign up using **Google login with your company email** (e.g., name@company.com). This keeps accounts tied to the organization.
2. The free tier is sufficient — 500 sites, 100GB bandwidth/month.
3. After creating the account, run `npx netlify login` to authenticate the CLI. This opens a browser window to authorize.
4. Verify with `npx netlify status` — should show the logged-in identity.

Always mention the Google login / company email recommendation when guiding setup, even if the current machine is already authenticated — the user may be asking on behalf of a teammate or planning to set up on another machine.

### 3. Get the account slug

```bash
npx netlify api listAccountsForUser 2>&1 | grep '"slug"'
```

Save the account slug — needed for creating sites. Typically the user's name in kebab-case (e.g., `derek-tsoi`).

## Deployment Workflow

### Step 1: Discover existing sites

Always start by listing existing sites to avoid creating duplicates:

```bash
npx netlify sites:list --json 2>&1 | python3 -c "
import json, sys
sites = json.load(sys.stdin)
for s in sites:
    print(f\"{s['name']:40s} {s['ssl_url']:50s} Updated: {s['updated_at'][:10]}\")
" 2>/dev/null
```

Present the list to the user and ask:
- **Update an existing site?** → Use that site's ID for deployment
- **Create a new site?** → Proceed to Step 2

### Step 2: Create a new site (if needed)

Use a descriptive name following the convention `bns-{project-name}`:

```bash
npx netlify sites:create \
  --name <site-name> \
  --account-slug <account-slug> \
  --disable-linking \
  --manual
```

Important flags:
- `--disable-linking` prevents writing `.netlify/state.json` into the repo
- `--manual` skips CI setup (not needed for manual deploys)
- `--account-slug` is required — get it from the prerequisites check

Save the **site ID** from the output (a UUID like `ea91e400-cd35-4f00-ab7c-81b710956b28`). The site name alone does not work reliably for the `--site` flag in deploy commands — always use the site ID.

### Step 3: Prepare the deploy directory

Verify the directory contains deployable files:

```bash
ls <deploy-directory>/*.html
```

If no `index.html` exists in the root and the directory contains multiple HTML files, create one with links to all HTML files to improve the browsing experience:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Prototypes</title>
    <style>
        body { font-family: system-ui, sans-serif; max-width: 600px; margin: 2rem auto; padding: 0 1rem; }
        a { display: block; padding: 0.5rem 0; color: #0066cc; }
    </style>
</head>
<body>
    <h1>Prototypes</h1>
    <ul>
        <!-- One <li><a href="filename.html">Display Name</a></li> per file -->
    </ul>
</body>
</html>
```

### Step 4: Deploy

```bash
npx netlify deploy --prod --dir=<deploy-directory> --site=<site-id>
```

Key notes:
- Always use `--site=<site-id>` (the UUID), not the site name — name-based lookup is unreliable
- `--prod` deploys to the production URL
- Without `--prod`, creates a draft deploy with a unique preview URL (useful for testing before going live)
- Add a deploy note with `--message "Updated mockup v2"`

### Step 5: Return the URL

After successful deployment, provide the user with:
- **Live URL**: `https://<site-name>.netlify.app`
- **Admin URL**: `https://app.netlify.com/projects/<site-name>`

## Site Management

### List all sites
```bash
npx netlify sites:list --json
```

### Delete a site
```bash
npx netlify sites:delete <site-id>
```
Always confirm with the user before deleting — this is irreversible.

### Redeploy (update existing site)
Same command as Step 4 — run `netlify deploy --prod` with the same site ID. Each deploy creates a new version; previous versions are retained and can be rolled back from the Netlify dashboard.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `--site` with site name returns 404 | Use the site ID (UUID) instead of the name |
| Interactive prompt hangs in CLI | Pass all flags explicitly (`--account-slug`, `--name`, `--disable-linking`, `--manual`) |
| "Not logged in" error | Run `npx netlify login` |
| Site shows "Page not found" | Verify `index.html` exists in the deploy directory root |
| CDN shows stale content | Wait 1-2 minutes for propagation, or hard refresh |

## Naming Conventions

To keep sites organized across the team:
- Format: `bns-{project-name}` (e.g., `bns-trending-merchants`, `bns-auth-flow-mockup`)
- Keep names short and descriptive
- Avoid version numbers in names — use Netlify's built-in deploy history instead
