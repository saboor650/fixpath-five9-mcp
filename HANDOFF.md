# five9-mcp — handoff bundle (2026-09-22)

Source: https://github.com/saboor650/fixpath-five9-mcp (FixPath — www.fixpathit.com).
Snapshot of the FixPath edition as of 2026-09-22, including the patch_ivr_script prompt-id fix.
47/47 tests pass (`npm test`).

## Deploy in 5 steps (Cloudflare Workers, zero npm dependencies)
1. `npm install` (dev tooling only) → `npm test`.
2. Edit `wrangler.toml`: set `DOMAIN_LABEL`, `CLIENT_LABEL`, and the Five9 data-center base URLs
   if not US.
3. Secrets: `npx wrangler secret put FIVE9_USERNAME`, `FIVE9_PASSWORD`, `MCP_AUTH_TOKEN`
   (an admin+supervisor API user with SOAP web-services enabled; for the New Platform REST tools
   also `FIVE9_CLIENT_ID` / `FIVE9_CLIENT_SECRET` from Admin Console → API Access Control).
4. `npx wrangler deploy`.
5. Add the Worker URL as a custom MCP connector in Claude (header `Authorization: Bearer <MCP_AUTH_TOKEN>`).

## Things to know
- One Worker = one Five9 domain. Do not point a second client at the same Worker.
- `src/about.js` carries the operator/ground-rules text the AI reads first — edit it for the new
  operator and client.
- Known limits (no API path, UI only): campaign Outbound SMS Number, Digital Skill, Chat Profiles,
  IVR-schedule channel ticks. `set_agent_permissions` and `set_user_roles` REPLACE the whole role
  permission list on the Five9 side — the tools read-then-merge, but verify with `get_user_details`
  (on one domain the permission write reported ok and did not persist).
- Never `PUT /campaigns/v1/.../campaigns/{id}` — it silently drops skills.
- `DEPLOY.md` is FixPath's deploy note; the client-specific values in it are examples.
