# Hizway Rentals — GitHub + Cloudflare portable project

**Business:** Hizway Investment Ltd  
**Frontend:** HTML, CSS, JavaScript  
**Backend:** Cloudflare Workers (JavaScript)  
**Database:** Cloudflare D1 (SQLite)  
**Static assets:** Cloudflare Workers Static Assets

This repository is the Cloudflare edition of Hizway Rentals. GitHub stores and versions the source code; Cloudflare runs the application and database. **Do not deploy this project with GitHub Pages or Cloudflare Pages**: the API requires a Worker and a D1 binding.

## 1. Requirements

- A GitHub account and a Cloudflare account.
- Node.js LTS on your Windows 11 PC.
- A Cloudflare API token with permission to deploy Workers and manage D1.

## 2. Create the D1 database once

Open PowerShell in this project folder:

```powershell
npm install
npx wrangler login
npx wrangler d1 create hizway-rentals-db
```

Copy the `database_id` returned by Cloudflare. Open `wrangler.jsonc` and replace `REPLACE_WITH_DATABASE_ID` with that ID. Commit this updated config to GitHub; the database ID is an identifier, not a password.

## 3. Set the session secret

```powershell
npx wrangler secret put SESSION_SECRET
```

Enter a long, random secret when prompted. Do not commit this secret to GitHub or put it in `wrangler.jsonc`.

## 4. Create the database tables

```powershell
npx wrangler d1 migrations apply hizway-rentals-db --remote
```

## 5. Test locally

```powershell
npm run dev
```

Open the local URL shown by Wrangler. Register an account and test the application before publishing it.

## 6. Upload the project to GitHub

Create a new **private** GitHub repository, for example `hizway-rentals`. In PowerShell, from this project folder:

```powershell
git init
git add .
git commit -m "Prepare Hizway Rentals for Cloudflare"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/hizway-rentals.git
git push -u origin main
```

Replace `YOUR-USERNAME` with your GitHub username. Keep the repository private because it contains application source code. Never commit passwords, API tokens, `.dev.vars`, or production secrets.

## 7. Enable automatic deployment from GitHub

In Cloudflare, create an API token with the permissions needed to deploy Workers and access D1. In GitHub open:

**Repository → Settings → Secrets and variables → Actions → New repository secret**

Add these two repository secrets:

- `CLOUDFLARE_API_TOKEN` — your Cloudflare API token.
- `CLOUDFLARE_ACCOUNT_ID` — your Cloudflare account ID.

The included `.github/workflows/deploy.yml` applies D1 migrations and deploys the Worker whenever you push to `main`, or when you manually run the workflow from the Actions tab.

The first deployment requires the D1 database to exist and `wrangler.jsonc` to contain its real database ID. Set `SESSION_SECRET` directly on the Worker with the command in step 3 before using account login. Cloudflare keeps this secret on the Worker; it does not need to be a GitHub secret.

## 8. Your live application

After deployment, Wrangler/GitHub Actions will show the Worker URL, normally:

`https://hizway-rentals.<your-subdomain>.workers.dev`

You can attach a custom domain later from the Cloudflare Workers & Pages dashboard. Configure the domain on the Worker, not GitHub Pages.

## What is included

- Owner account registration and login
- Dashboard and basic rent totals
- Properties and units
- Tenants and leases
- Manual rent-payment recording
- Outstanding-rent follow-up list
- Expenses
- Maintenance requests
- Basic financial totals/report view
- Responsive dark-green/mint interface and Hizway logo
- D1 schema migration and GitHub Actions deployment workflow

## Important limits before real customers use it

This is a deployable conversion/starter, not a production-certified multi-landlord SaaS. The code still needs a full security and functional review, automated tests, role-based staff access, platform super-admin, tenant portal, record editing workflows, audit logs, reliable arrears/accounting rules, backups/restore testing, and privacy/legal review. It does not send SMS/WhatsApp reminders or process online payments. Do not store real tenant identity or financial records until the security review and backup plan are complete.

The original PHP/MySQL database cannot be imported directly into D1. D1 is SQLite-based; this project includes its own D1 migration in `migrations/0001_initial.sql`.
