# Aide · Vercel + Supabase backend

This deploys Aide's saving API on Vercel and stores its workspace in your existing Supabase database. It is a single private workspace, protected by a separate Aide saving key. It does not create teacher accounts or change the app's temporary `000000` entrance.

## Link your existing projects

1. Run `supabase/schema.sql` in your Supabase project's SQL Editor. It adds `aide_workspaces`, `aide_workspace_history`, and the `aide_save_workspace` function. It enables RLS and denies anonymous/authenticated direct access to these new objects. It does not change unrelated tables.
2. Put this folder's contents at the root of a Git repository. Import that repository into your Vercel project. Framework: **Other**. Build command: empty. Output directory: default/empty. No application dependency installation is needed. Node runtime: 22.x. The `api/workspace.js` function uses Vercel's standard Fetch handler.
3. Add Production environment variables in Vercel:

| Variable | Value |
| --- | --- |
| `SUPABASE_URL` | Your existing Supabase project URL, such as `https://PROJECT.supabase.co` |
| `SUPABASE_SECRET_KEY` | A server-only `sb_secret_…` key from Supabase Settings → API Keys |
| `AIDE_SYNC_KEY` | A new random saving key, at least 32 characters; use the guide's generator |
| `AIDE_ALLOWED_ORIGINS` | `https://aide-teacher-studio.ridoditama.chatgpt.site` |
| `AIDE_WORKSPACE_ID` | `aide` |

The existing integration's `SUPABASE_SERVICE_ROLE_KEY` can be used instead of `SUPABASE_SECRET_KEY` if your project only has legacy keys. Never use an anon/publishable key for this backend. Do not prefix server secrets with `NEXT_PUBLIC_`. Do not put them in Aide, Git, screenshots, or chat. Store them in Vercel's secret environment settings.

4. Deploy/redeploy after setting the variables. Changes do not affect an older deployment. Keep Vercel project protection settings compatible with cross-origin calls to the production API. If Vercel shows a platform sign-in response instead of Aide JSON, use a production deployment allowed to serve this API; do not put a Vercel bypass token into the browser. The saving key still protects every API read/write.
5. In Aide → Settings → Vercel & Supabase, enter your `https://PROJECT.vercel.app` production URL and **AIDE_SYNC_KEY**. Click **Connect & check**. This reads database status; it does not save or replace records.
6. For a new cloud workspace, click **Save to cloud**. For an existing cloud workspace, use **Load cloud copy** after exporting local data you want to keep. Then turn on autosave if desired. Reconnect after reloading: the saving key is held only in page memory.

If your existing Vercel project hosts another application, deploy this bundle as a separate backend project using the same Supabase database. Avoid replacing unrelated source files. If the Vercel dashboard already supplied Supabase environment variables, verify the names above and reuse the same project's values.

## Saving behavior

- API route: `/api/workspace`, GET/POST/OPTIONS. Every data request requires the saving key and a configured origin. Origins are exact URLs, without trailing slashes; multiple origins may be comma-separated. There is no arbitrary database, table, user, or URL selector.
- The database stores the full version-1 Aide workspace as JSONB. Existing exports/imports remain compatible. Local browser saving still happens first; a separate cloud status reports cloud success.
- Atomic database revision checks reject old-device updates with HTTP 409. Duplicate requests with the same UUID and payload return the original success. An acknowledgement lost in transit can be reconciled by Check cloud; an explicit retry reuses the same request UUID and snapshot.
- Autosave is off until opted in. Errors/conflicts pause it. It runs only while the tab is open. Loads replace local records only after confirmation. Unconfirmed requests are never replayed automatically.
- Supabase retains the latest 20 revisions in `aide_workspace_history`. History is not public and is accessible through the Supabase dashboard; this app version does not have a history restoration UI. Continue exporting independent backups.
- Workspace requests are capped at 2 MB. Export a backup if you reach the limit.
- One saving key grants access to one workspace. Share it only with intended collaborators. For separately isolated teacher accounts, add an account-based authorization design before expanding the audience.

## Checks

Run `node --test tests/handler.test.mjs` for access, validation, server-owned targeting, conflict, and sanitized-error checks. The Aide source includes cloud client workflow tests. The SQL and end-to-end save/load protocol were also run against a local Postgres runtime, including role restrictions, idempotency, conflicts, lost acknowledgements, and version retention. Your live Vercel deployment and Supabase database are not yet configured or tested by this package.

Official references: [Vercel Node Functions](https://vercel.com/docs/functions/runtimes/node-js), [Vercel environment settings](https://vercel.com/docs/environment-variables), [Supabase API keys](https://supabase.com/docs/guides/getting-started/api-keys), [Supabase function permissions](https://supabase.com/docs/guides/database/functions).
