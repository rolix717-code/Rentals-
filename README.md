# HIZWAY RENTALS

**Hizway Investment Ltd** · Rental and property management workspace

A deployable Cloudflare Workers application with a responsive browser interface, Cloudflare D1 database, account registration/login, and GitHub Actions deployment workflow.

## Included in this build

- Owner account registration and sign-in (PBKDF2 password hashing; signed, HTTP-only session cookie).
- Dashboard with rent collection, active rent, occupancy, and unpaid-rent indicators.
- Property, unit, tenant, lease, payment, expense, and maintenance registers.
- Rent follow-up list and basic financial summary/print view.
- Responsive interface using the supplied Hizway brand logo.
- D1 migration and GitHub Actions workflow for Cloudflare Workers.

## Technology

- Frontend: HTML, CSS, vanilla JavaScript
- Backend: Cloudflare Workers (ES modules)
- Database: Cloudflare D1 (SQLite)
- Static assets: Workers Static Assets
- Deployment: Wrangler CLI or GitHub Actions

> This is a functional project foundation, not a certified accounting or property-management SaaS. SMS/WhatsApp delivery, online payment collection, staff roles, audit trails, full edit workflows, and production security review are not included yet. Test with sample data before entering real tenant or financial information.

## Requirements

- Node.js 20 or newer (Node.js 22 LTS recommended)
- npm
- Cloudflare account with Workers and D1 access
- GitHub account if using automatic deployment

## 1. Install dependencies

Open a terminal in this project directory:

```bash
npm install
npx wrangler login
```

## 2. Create the D1 database

Run this once:

```bash
npx wrangler d1 create hizway-rentals-db
```

Cloudflare prints a `database_id`. Open `wrangler.jsonc` and replace `REPLACE_WITH_DATABASE_ID` with that exact ID. Do not change the database name or binding unless you also update the commands and workflow.

## 3. Configure the session secret

Generate a long random secret (at least 32 characters) and store it directly on the Worker:

```bash
npx wrangler secret put SESSION_SECRET
```

Do not put this secret in GitHub source code, `wrangler.jsonc`, or a committed `.dev.vars` file. For local development, create a local `.dev.vars` file containing:

```text
SESSION_SECRET=replace-with-a-random-secret-of-at-least-32-characters
```

`.dev.vars` is ignored by Git.

## 4. Apply the database migration

For the remote production database:

```bash
npm run db:migrate
```

For a local development database:

```bash
npm run db:migrate:local
```

## 5. Run and check locally

```bash
npm run check
npm run dev
```

Open the local address printed by Wrangler. Register an owner account and test the application with sample properties, units, tenants, leases, payments, expenses, and maintenance records.

## 6. Deploy manually

After the database ID and production `SESSION_SECRET` are configured:

```bash
npm run deploy
```

The Worker URL is shown by Wrangler after deployment. You can attach a custom domain to the Worker from the Cloudflare dashboard. This project uses Workers, not GitHub Pages.

## 7. Upload to GitHub

Create a repository (private is recommended), then from this project folder:

```bash
git init
git add .
git commit -m "Set up Hizway Rentals"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/hizway-rentals.git
git push -u origin main
```

Replace `YOUR-USERNAME` with your GitHub username. Never commit `.dev.vars`, passwords, or API tokens.

## 8. Enable automatic GitHub deployment

In GitHub, open **Repository → Settings → Secrets and variables → Actions** and add:

- `CLOUDFLARE_API_TOKEN` — API token permitted to deploy the Worker and access the D1 database.
- `CLOUDFLARE_ACCOUNT_ID` — your Cloudflare account ID.

The workflow in `.github/workflows/deploy.yml` applies migrations and deploys on pushes to `main` and on manual workflow dispatch. The D1 database must already exist and `wrangler.jsonc` must contain its real ID. Set `SESSION_SECRET` on the Worker before users sign in.

## Project layout

```text
├── .github/workflows/deploy.yml  # GitHub Actions deployment
├── migrations/                  # D1 SQL migrations
├── public/                      # Browser app and brand assets
│   ├── assets/app.css
│   ├── assets/app.js
│   ├── assets/hizway-logo.png
│   └── index.html
├── src/index.js                 # Worker API and session handling
├── package.json
└── wrangler.jsonc                # Worker, assets, and D1 binding
```

## Important next development milestones

1. Complete create/read/update/delete workflows and validation for every register.
2. Strengthen tenant/lease/payment relationship checks and add audit logs.
3. Implement accurate month-by-month rent allocation, partial payments, arrears, and lease expiry rules.
4. Add receipt generation and downloadable PDF/CSV reports.
5. Add scheduled reminders through a selected SMS/WhatsApp provider.
6. Add staff accounts/permissions, backup/restore procedures, and a security/privacy review.

The existing schema is a fresh D1 schema. It does not import an existing MySQL/PHP database automatically.
