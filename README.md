# Calorie Compass PWA

A lightweight Progressive Web App for estimating food calories/macros, exercise calories, and approximate daily energy balance. Designed for mobile Safari and GitHub Pages.

## Included

- Responsive web app with food and exercise logs
- Profile-based baseline energy and suggested macro targets
- AI estimate endpoint template using OpenAI API via a Cloudflare Worker
- Built-in rough heuristic estimates when no AI endpoint is configured
- Local browser storage, offline app shell, PWA manifest and icon
- No account or analytics by default

## Important limitation

**GitHub Pages is static hosting. Do not put an AI API key in `app.js`, HTML, or any public GitHub file.** The optional Cloudflare Worker keeps the provider key server-side. AI estimates cost money according to your provider plan and can be wrong. The built-in fallback is a basic keyword heuristic, not AI.

## 1. Publish the app on GitHub Pages

1. Create a new GitHub repository, for example `calorie-compass`.
2. Upload the contents of this project (the files inside this folder, not the enclosing ZIP).
3. In the repository, open **Settings → Pages**.
4. Under Build and deployment, choose **Deploy from a branch**, select `main` and `/ (root)`, then Save.
5. Wait for the published URL, usually `https://YOUR-USERNAME.github.io/calorie-compass/`.

Use HTTPS for service workers and installation. GitHub Pages provides HTTPS automatically.

## 2. Enable AI estimates (optional but recommended)

The app works without this step, using basic local heuristics. For AI estimates, deploy the included Worker.

1. Create a Cloudflare account and install Node.js plus Wrangler on your computer.
2. From the `worker` folder, authenticate Wrangler with `npx wrangler login`.
3. Set the provider key as a secret (do not commit it): `npx wrangler secret put OPENAI_API_KEY`.
4. Deploy from the `worker` folder: `npx wrangler deploy`.
5. Copy the deployed Worker base URL, e.g. `https://calorie-compass-estimator.YOUR-SUBDOMAIN.workers.dev`.
6. Open the GitHub Pages app → **AI connection & privacy** → paste the Worker base URL → **Save endpoint**.

The Worker expects an OpenAI API key with API access and uses `gpt-4o-mini` by default. Model availability and pricing depend on your provider account. To change the model, edit `MODEL` in `worker/wrangler.toml` and redeploy.

### Production hardening

The included Worker uses permissive CORS (`*`) so the static app can call it easily. For a public production deployment, change `Access-Control-Allow-Origin` to your exact GitHub Pages origin, add rate limiting/abuse protection, and consider a usage budget. CORS alone is not authentication and does not stop non-browser callers.

## 3. Add to iPhone/iPad Home Screen

1. Open the deployed HTTPS URL in **Safari**.
2. Tap **Share**.
3. Choose **Add to Home Screen**.
4. Tap **Add**.

On Mac, Safari supports adding web apps in supported macOS versions; installation options vary by OS version. The site can also be used directly in Safari without installing.

## Energy-balance calculation

- Baseline daily energy expenditure is estimated using the Mifflin–St Jeor equation multiplied by the selected activity factor.
- The daily balance shown is `food calories − baseline daily expenditure − logged exercise calories`.
- Because the activity factor already accounts for typical activity, the app warns users not to log routine activity twice. The baseline estimate is still only an approximation.
- Macro targets are generic starting points, not personalized clinical recommendations. Age under 18 is intentionally not supported by the UI.

## Privacy

Profile and logs are stored in localStorage in the current browser on the current device. They are not uploaded by the app except food/exercise descriptions and relevant fields sent to the configured AI Worker when a user explicitly asks for an estimate. The Worker sends the estimate request to the AI provider. Review provider privacy terms before use. Clearing browser data may remove saved entries.

## Files

- `index.html`, `styles.css`, `app.js`: user interface and app logic
- `manifest.webmanifest`, `sw.js`, `icons/icon.svg`: PWA setup
- `worker/worker.js`, `worker/wrangler.toml`: optional AI proxy
