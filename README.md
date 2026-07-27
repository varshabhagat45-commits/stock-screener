# Long-Term Stock Screener — deployment guide

This turns the screener into a real, always-on website (like Investilo) where the
"Analyze a stock" tab actually calls Claude live, from any browser, with no Claude.ai
session needed. It has two parts:

- `index.html` — the screener + Analyze UI (unchanged from the version you already have)
- `api/analyze.js` — a small serverless function that holds your Anthropic API key
  **on the server** and calls Claude on the page's behalf. The key never reaches the browser.

You need two accounts, both free to create:
1. An **Anthropic API key** — separate from your claude.ai login. Analysis calls here are
   billed pay-as-you-go on that key (not part of a Claude.ai subscription).
2. A **Vercel** account (or Netlify/Render — steps below are for Vercel, the simplest path).

---

## 1. Get an Anthropic API key

1. Go to https://console.anthropic.com and sign in (or create an account).
2. Add billing (Settings → Billing) — pay-as-you-go, no separate subscription needed.
3. Settings → API Keys → Create Key. Copy it — you won't see it again after this screen.

Keep this key private. Never paste it into `index.html` or any file that reaches the browser —
that's the whole reason `api/analyze.js` exists.

## 2. Deploy to Vercel (free tier is enough for personal use)

**Easiest path — no command line:**
1. Go to https://vercel.com and sign up (GitHub login is simplest).
2. Create a new GitHub repo, upload these three files/folders to it:
   `index.html`, `api/analyze.js`, `package.json`.
3. In Vercel: **Add New → Project → Import** your new repo.
4. Before clicking Deploy, open **Environment Variables** and add:
   - Name: `ANTHROPIC_API_KEY`
   - Value: *(paste the key from step 1)*
5. Click **Deploy**. Vercel gives you a live URL like `your-project.vercel.app` —
   that's your website. Open it, go to the Analyze tab, and it should call Claude directly.

**If you prefer the command line instead:**
```bash
npm install -g vercel
cd stock-screener-app
vercel login
vercel                      # first deploy, follow the prompts
vercel env add ANTHROPIC_API_KEY production   # paste your key when asked
vercel --prod                # redeploy with the env var applied
```

## 3. Using it afterward

- The screener table works exactly as before — no backend needed for that part.
- The Analyze tab now calls `/api/analyze` on your own domain, which calls Claude
  with your key server-side and returns the write-up.
- Each analysis run costs a small amount on your Anthropic billing — the deep-dive
  frameworks (Operating/NBFC/Value/Thesis) use more tokens (and cost more) than
  the Quick Take.
- To update the screener's auto-loaded shortlist or the framework prompts later,
  edit `index.html` (the stock list) or `api/analyze.js` (the prompts), then
  redeploy (`vercel --prod`, or just push to GitHub if you connected that way —
  Vercel redeploys automatically on push).

## Troubleshooting

- **"Server is missing ANTHROPIC_API_KEY"** — the environment variable isn't set
  for this deployment. Re-check step 4 above, or Vercel dashboard → your project →
  Settings → Environment Variables.
- **401/403 from Anthropic** — the key is invalid, revoked, or billing isn't set up
  on the Anthropic account. Recheck console.anthropic.com.
- **Works locally but not after redeploy** — env vars set via `vercel env add` only
  apply to new deployments; run `vercel --prod` again after adding one.
