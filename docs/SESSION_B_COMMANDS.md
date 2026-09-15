# Session B Commands and HW4 Checklist

This repository is the HW4 template. It contains the HW3 three-file app
already wired for a Worker, the full Context Scaffold, and the HW4 backend
files. If you are continuing in your own `mgt3745-hw3` repository (the
default), copy in `worker.js`, `wrangler.toml`, `schema.sql`, `package.json`,
`.gitignore`, `.devcontainer/`, and `context/TOOLS.md` and `context/STYLE.md`,
then follow the steps below. If you are starting fresh from this template,
paste your HW3 context files over the placeholders first.

## Session B, in Commands

Copy from here. Type nothing from the slides.

```bash
# Preflight
sudo apt-get install -y xdg-utils # installs xdg-utils which is needed for wrangler
npx wrangler --version          # if this fails: npm install
npx wrangler login --device              # approve in the browser tab; this token is a crossing

# Step 1: create the database
npx wrangler d1 create mgt3745-entries
#   paste the database_id it prints into wrangler.toml, replacing PASTE_ID_HERE
npx wrangler d1 execute mgt3745-entries --remote --file=schema.sql

# Step 3: deploy and hit it
npx wrangler deploy
#   prints https://mgt3745-hw4.<your-subdomain>.workers.dev
#   open <that url>/entries in a tab; expect []

curl -X POST https://mgt3745-hw4.<your-subdomain>.workers.dev/entries \
  -H 'content-type: application/json' \
  -d '{"text":"first crossing"}'
#   expect 201; reload the tab; expect one entry
```

If curl intimidates, paste this into the browser console on any page instead:

```js
fetch("https://mgt3745-hw4.<your-subdomain>.workers.dev/entries", {
  method: "POST",
  headers: { "content-type": "application/json" },
  body: JSON.stringify({ text: "first crossing" })
}).then(r => console.log(r.status));
```

## Step 4: Wire It to Your HW3 Page

Replace the two localStorage lines in `app.js`. Render is unchanged;
`textContent` still applies.

```js
const API = "https://mgt3745-hw4.<your-subdomain>.workers.dev";

async function load() {
  const res = await fetch(API + "/entries");
  if (!res.ok) { showError("could not load"); return []; }
  return res.json();
}

async function save(entry) {
  const res = await fetch(API + "/entries", {
    method: "POST",
    headers: { "content-type": "application/json" },
    body: JSON.stringify(entry)
  });
  if (!res.ok) showError("could not save");
}
```

`showError` is yours to write: put the message somewhere on the page a user
would see it. Do not throw in the console.

The demo: add an entry. DevTools, Application, Clear site data. Reload. The
entry is still there. That is your See It Work GIF.

## Common Failures

| Symptom | Cause | Fix |
|---|---|---|
| `wrangler deploy` complains about wrangler.toml | Broken TOML after pasting the id | Keep the quotes around the id. Change nothing else on that line. |
| Deployed Worker returns 500 on GET | Schema ran locally, not on Cloudflare | Rerun `d1 execute` with `--remote`. |
| Browser console: blocked by CORS policy | Worker missing the CORS headers, or the OPTIONS branch | Both are in the template `worker.js`. Compare yours line by line. |
| POST returns 400 "body must be JSON" | Missing `content-type` header or invalid JSON | Copy the fetch above exactly. |
| `PASTE_ID_HERE` still in wrangler.toml | Step 1 not finished | Run `d1 create` and paste the id. |

## Running Locally (Optional)

`npm run dev` starts the Worker on port 8787 with a local D1 emulator. We skip
it in Session B so the crossing is real; it is useful later for testing the
failure modes your verification table needs. Run the schema locally first
with `npx wrangler d1 execute mgt3745-entries --local --file=schema.sql`.

## Before HW4 Is Done

- `wrangler.toml` has a real id and no secrets.
- `.gitignore` excludes `node_modules/` and `.wrangler/`.
- `worker.js` has one validation rule of yours, traced to an EARS statement.
- `CORS` origin narrowed from `*` to your page's origin (Craft credit).
- TOOLS.md has at least four rows with crossing statements.
- STYLE.md has at least four tokens, one sentence each, and two refusals.
- Deployed URL is in the README and returns `[]` or entries, never an error.
- Final commit tagged `hw4`.
