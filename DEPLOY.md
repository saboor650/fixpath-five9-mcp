# Deploying five9-mcp (FixPath)

One Worker serves one client domain. If a Worker for the domain already exists and holds its Five9
secrets, an upgrade is a redeploy of the same Worker name.

## 1. Put the repo on GitHub (once)

```bash
cd five9-mcp
git remote add origin git@github.com:<your-org>/five9-mcp.git
git push -u origin main
```

Keep the repo **private**: `src/about.js` names the operator and the playbooks describe client designs.

## 2. Configure labels

In `wrangler.toml` (or the Cloudflare dashboard → Worker → Settings → Variables):

```toml
DOMAIN_LABEL = "Example Client Inc"
CLIENT_LABEL = "Example Client"
```

Secrets (`FIVE9_USERNAME`, `FIVE9_PASSWORD`, `MCP_AUTH_TOKEN`, optional REST keys) are unchanged on
a redeploy — do not re-enter them.

## 3. Test and deploy

```bash
npm test                # 47 tests, no Five9 access needed
npx wrangler deploy     # same Worker name = same URL, the connector keeps working
```

Open `https://<worker>.workers.dev/` — the landing page lists the tool count.

## 4. First-run verification (read-only → harmless writes)

Do these from the AI client, in order. Each one exercises a code path that unit tests cannot
prove against a live domain.

1. `about` — confirm it shows your DOMAIN_LABEL / CLIENT_LABEL and FixPath.
2. `list_ivr_modules` on an existing IVR script — inventory should match the designer.
3. `patch_ivr_script` with `dry_run: true` and one `rename_module` op — review the change list.
4. `manage_web_connector modify` — trigger dispositions, URL, and POST fields all round-trip.
   One Five9 quirk: a connector with POST fields but **no URL variables** cannot be modified
   (Five9 rejects the POST fields as unknown call variables); the tool says so and accepts
   `variables: {"session_id": "Call.session_id"}` in the same call as the workaround.
5. `modify_vcc_configuration` with a value already in place, e.g.
   `{"miscOptions": {"voicemailTimeout": 20}}` — then `get_vcc_configuration`.
6. `find_calls` with `hours: 24, ani: "<your test phone>"`.
7. `bulk_create_users` with a two-row CSV and `dry_run: true` (default).

Only after 4 and 5 succeed should the write versions be used for real changes.
