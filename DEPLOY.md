# Deploying on Vercel or Netlify (no local setup needed)

This version runs on Vercel **or** Netlify. Everything below is done in a web browser.

What changed from the Cloudflare version
- Database: Cloudflare D1 -> **Turso** (also SQLite, so all SQL is unchanged). Tables and seed data are created automatically on first load.
- Uploads: R2 binding -> **S3 API**. You can keep using your existing Cloudflare R2 bucket.
- Upload size limit is now 4 MB (Vercel/Netlify cap request bodies at about 4.5 MB).

## 1. Create the database (Turso, free tier)
1. Sign up at https://turso.tech and create a database (pick a region near India, e.g. Mumbai if offered).
2. Copy the **Database URL** (starts with `libsql://`) -> `TURSO_DATABASE_URL`.
3. Create a **token** for that database -> `TURSO_AUTH_TOKEN`.

## 2. Storage for uploaded images (reuse your R2 bucket)
1. Cloudflare dashboard -> R2 -> **Manage API Tokens** -> create a token with *Object Read & Write* on `1st-choice-uploads`.
2. You get an Access Key ID and Secret Access Key, plus your account's S3 endpoint
   (`https://<account-id>.r2.cloudflarestorage.com`).
3. Fill `S3_ENDPOINT`, `S3_BUCKET=1st-choice-uploads`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_REGION=auto`.
   Leave the bucket private; the website serves images through `/uploads/...`.

## 3A. Deploy on Vercel
1. Push this folder to GitHub (replace the contents of your `gill1s` repo).
2. vercel.com -> **Add New -> Project** -> import the repo.
3. Framework Preset: **Other**. Leave Build/Output settings at their defaults (`vercel.json` sets the build command).
4. Add the environment variables from `.env.example` (at minimum the Turso, admin and S3 ones).
5. Deploy, then add `1stchoiceimmigration.online` under Project -> Domains and update DNS as Vercel shows.

## 3B. Deploy on Netlify
1. Push to GitHub, then app.netlify.com -> **Add new site -> Import an existing project**.
2. Settings are read from `netlify.toml` (build `npm run build`, publish `public`).
3. Add the same environment variables under Site configuration -> Environment variables.
4. Deploy, then add your domain under Domain management.

## 4. First login
Open `/admin/login`. If no admin exists, one is created from `ADMIN_BOOTSTRAP_EMAIL` / `ADMIN_BOOTSTRAP_PASSWORD`
(password must be 12+ characters). There is no default password any more.

## Troubleshooting
- Page says "Server is not configured yet": a required environment variable is missing (the message names it).
- Generic "Something went wrong": check the function logs in Vercel/Netlify, or temporarily set `DEBUG=1`.
- Visit `/healthz` — it answers `ok` without touching the database, which separates hosting problems from database problems.
- Image upload errors: re-check the five `S3_*` variables and that the R2 token has write access.
