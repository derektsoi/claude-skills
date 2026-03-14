# Netlify CLI Quick Reference

## Authentication
| Command | Purpose |
|---------|---------|
| `npx netlify login` | Authenticate via browser (Google login supported) |
| `npx netlify logout` | Clear stored credentials |
| `npx netlify status` | Show current auth status and account info |
| `npx netlify api listAccountsForUser` | Get account slug and capabilities |

## Site Management
| Command | Purpose |
|---------|---------|
| `npx netlify sites:list --json` | List all sites with metadata |
| `npx netlify sites:create --name NAME --account-slug SLUG --disable-linking --manual` | Create a new site |
| `npx netlify sites:delete SITE_ID` | Delete a site (irreversible) |

## Deployment
| Command | Purpose |
|---------|---------|
| `npx netlify deploy --prod --dir=PATH --site=SITE_ID` | Deploy to production |
| `npx netlify deploy --dir=PATH --site=SITE_ID` | Create a draft/preview deploy |
| `npx netlify deploy --prod --dir=PATH --site=SITE_ID --message "note"` | Deploy with a log message |

## Important Notes
- Always use site ID (UUID) for `--site`, not the site name
- `npx` ensures the latest CLI version without global install
- All interactive prompts must be bypassed with explicit flags for CLI automation
- Free tier: 500 sites, 100GB bandwidth/month, unlimited deploys
