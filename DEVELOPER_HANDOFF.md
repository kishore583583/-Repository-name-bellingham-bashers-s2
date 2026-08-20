# Bellingham Bashers S2 — Developer Handoff

_Last verified: 2026-08-20_

This document is the operating handoff for a developer or Codex session that needs to maintain, upgrade, deploy, troubleshoot, or recover the Bellingham Bashers S2 application.

## 1. Source-of-truth / access model

Target operating model:

**GitHub (source + history) → Vercel (frontend/deployment entry) → Supabase (backend, database, Edge Functions)**

Do not share account passwords. Each developer should be invited to the three platforms using their own account and should create their own personal CLI/API tokens when needed.

---

## 2. GitHub

### Repository

- Owner: `kishore583583`
- Current private repository: `kishore583583/-Repository-name-bellingham-bashers-s2`
- Default branch: `main`
- Clone URL: `https://github.com/kishore583583/-Repository-name-bellingham-bashers-s2.git`
- Visibility: **Private**

> Note: the repository name currently contains the accidental `-Repository-name-` prefix. It can be renamed later to `bellingham-bashers-s2`, but coordinate the rename with Vercel/Git links first.

### Required developer access

Grant the developer **Admin** or equivalent repository permission if they are expected to own releases and settings. At minimum they need:

- Read/write code
- Create branches
- Open/merge pull requests
- Create tags/releases
- Read/write GitHub Actions if CI/CD is added
- Manage repository settings if they are a full maintainer

### Authentication

Preferred:

- GitHub account + repository collaborator access
- Codex/ChatGPT GitHub connector authorized for this private repository

For local CLI use, developer may use their own GitHub CLI login (`gh auth login`) or their own fine-grained PAT. **Never store a GitHub PAT in this repository.**

---

## 3. Vercel

### Production project

- Project name: `bellingham-bashers-s2`
- Project ID: `prj_CYruRlxFeaKZKxFjPn5N9wzhgKpv`
- Team ID: `team_ucIapL5XHNPPNqGkKCvOXhh3`
- Production domain: `https://bellingham-bashers-s2.vercel.app`
- Current Vercel Node runtime setting: `24.x`
- Latest verified deployment ID: `dpl_B4ZsZejqVxkJLzLRnYWxUL5QKrPt`

### Required developer access

Invite the developer to the Vercel team/project with enough permission to:

- View and create deployments
- Promote/redeploy/rollback production
- Modify build/project configuration
- Manage domains
- Manage environment variables
- Inspect runtime/build logs
- Manage Git integration

Keep the original owner account as an owner/admin for recovery.

### Authentication

Preferred:

- Developer's own Vercel account invited to the team/project
- `vercel login` for local CLI work

If automation requires `VERCEL_TOKEN`, create a token under the developer/service account and store it only in a secure secret store or CI environment. **Do not commit `VERCEL_TOKEN` to GitHub.**

Identifiers that are safe to store in configuration:

```text
VERCEL_ORG_ID=team_ucIapL5XHNPPNqGkKCvOXhh3
VERCEL_PROJECT_ID=prj_CYruRlxFeaKZKxFjPn5N9wzhgKpv
```

---

## 4. Supabase

### Project

- Project ref / ID: `sypbqpkvwzzvazcpjptu`
- Organization ID: `hydcyevqzhgewiqyowdu`
- Project name in Supabase: `qysnsx888b@privaterelay.appleid.com's Project`
- Region: `us-east-2`
- Project status at handoff: `ACTIVE_HEALTHY`
- Supabase API URL: `https://sypbqpkvwzzvazcpjptu.supabase.co`
- Database host: `db.sypbqpkvwzzvazcpjptu.supabase.co`
- PostgreSQL major version: `17`

### Safe client/runtime values

Use the modern publishable key for browser/client code:

```text
SUPABASE_URL=https://sypbqpkvwzzvazcpjptu.supabase.co
SUPABASE_PUBLISHABLE_KEY=sb_publishable_W1oJ9jMnblWmCrsZhKp0OQ_LNjdcYtH
```

Legacy anon key (still enabled at handoff; keep only for compatibility if existing code requires it):

```text
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InN5cGJxcGt2d3p6dmF6Y3BqcHR1Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODY5MDE3ODgsImV4cCI6MjEwMjQ3Nzc4OH0.PoO9gNq3XQux3H7eVaGILkILzo-tHIHWdXqC8__9uH0
```

These are publishable/client keys, not administrative credentials.

### Secrets that MUST NOT be placed in this document or Git

Do not commit or send in chat/plaintext:

- `SUPABASE_SERVICE_ROLE_KEY`
- Supabase database password / connection password
- Supabase personal access token
- JWT signing secrets
- Any OAuth provider client secrets
- SMTP secrets
- Webhook signing secrets

A developer with proper Supabase organization/project membership should create/use their **own** Supabase personal access token for CLI/API administration. Service-role credentials should only be stored in a trusted server environment or platform secret store when an application component genuinely requires them.

### Required developer access

Invite the developer to the Supabase organization/project with permission to manage:

- SQL/database schema and migrations
- Edge Functions
- Auth configuration
- Storage
- Logs
- API settings
- Project configuration
- Secrets/environment variables

For full maintenance responsibility, use the highest appropriate project/org role while keeping the original owner account active.

### Edge Functions currently present

Current versions verified at handoff:

| Function | Version | JWT verification |
|---|---:|---|
| `bashers-app` | 60 | disabled |
| `game-center` | 9 | disabled |
| `tournament-admin` | 12 | disabled |
| `tournament-score` | 1 | disabled |
| `bashers-access` | 2 | disabled |
| `roster-position` | 2 | disabled |
| `match-admin` | 7 | disabled |
| `scorer-live` | 4 | disabled |
| `support-board` | 2 | disabled |
| `bashers-revised-app` | 2 | disabled |
| `tournament-ops` | 1 | disabled |
| `waiver-center` | 6 | disabled |
| `youtube-live-lab` | 10 | disabled |

**Security warning:** JWT verification is currently disabled on these deployed functions. Do not blindly enable it in production because existing public flows may depend on unauthenticated function access. Treat this as a security-hardening item: inspect each function's custom authorization/role checks, test in a branch/staging environment, then enable JWT or stronger server-side authorization where appropriate.

---

## 5. Current application architecture

The production `bellingham-bashers-s2` Vercel project is currently a thin deployment/loader layer; the primary S2 application UI/logic is served from the Supabase Edge Function `bashers-app`.

Therefore, a Vercel deployment alone does not necessarily contain the full current application source. The migration goal is to move all recoverable Supabase Edge Function source and schema/migration assets into this GitHub repository so **GitHub becomes the true source of truth**.

Until that migration is complete, verify whether a proposed change belongs in Vercel or Supabase before editing/deploying.

---

## 6. Recommended repository layout

```text
/
├── README.md
├── DEVELOPER_HANDOFF.md
├── PROJECT_CONTEXT.md
├── ARCHITECTURE.md
├── CHANGELOG.md
├── .gitignore
├── supabase/
│   ├── config.toml
│   ├── migrations/
│   └── functions/
│       ├── bashers-app/
│       ├── game-center/
│       ├── tournament-admin/
│       ├── tournament-score/
│       ├── bashers-access/
│       ├── roster-position/
│       ├── match-admin/
│       ├── scorer-live/
│       ├── support-board/
│       ├── tournament-ops/
│       └── waiver-center/
└── web/ or app/             # if/when frontend source is separated from Edge Function HTML
```

Never add `.env`, `.env.local`, service-role keys, DB passwords, Vercel tokens, GitHub PATs, or Supabase personal tokens to source control.

Suggested `.gitignore` entries:

```text
.env
.env.*
!.env.example
.vercel
.supabase
node_modules
.DS_Store
```

---

## 7. Local developer authentication / bootstrap

A new maintainer should authenticate using their **own accounts**, not copied owner credentials.

### GitHub

```bash
gh auth login
git clone https://github.com/kishore583583/-Repository-name-bellingham-bashers-s2.git
cd -Repository-name-bellingham-bashers-s2
```

### Supabase CLI

```bash
supabase login
supabase link --project-ref sypbqpkvwzzvazcpjptu
```

For normal client development, use only the safe runtime values:

```bash
SUPABASE_URL=https://sypbqpkvwzzvazcpjptu.supabase.co
SUPABASE_PUBLISHABLE_KEY=sb_publishable_W1oJ9jMnblWmCrsZhKp0OQ_LNjdcYtH
```

### Vercel CLI

```bash
vercel login
vercel link
```

When prompted, link to project `bellingham-bashers-s2` under team `team_ucIapL5XHNPPNqGkKCvOXhh3`.

---

## 8. Deployment rules

### Before any production change

1. Pull latest `main`.
2. Create a branch, e.g. `fix/matches-page` or `feature/bracket-ui`.
3. Make one scoped change at a time.
4. Verify role-based functionality and navigation.
5. Open a PR.
6. Review diff before merging.
7. Tag meaningful production releases.
8. Deploy the exact merged commit/function source.
9. Record deployment IDs/function versions in the release notes.

### Supabase Edge Function deployment

Example:

```bash
supabase functions deploy bashers-app --project-ref sypbqpkvwzzvazcpjptu
```

Use the function's existing JWT setting unless the change explicitly includes tested authentication hardening.

### Vercel production deployment

After Git integration is established, prefer deploys tied directly to Git commits/PRs. For manual CLI production deployment:

```bash
vercel --prod
```

Always confirm the linked Vercel project before running a production command.

---

## 9. Version / release standard

Use semantic-style tags for recoverable releases:

```text
v2.0.0
v2.1.0
v2.1.1-matches-fix
```

For historical snapshots recovered from pre-Git deployments, use clearly labeled legacy tags/branches, for example:

```text
legacy/2026-08-18-pre-bracket
legacy/2026-08-19-known-good
legacy/2026-08-20-current
```

Do not fabricate historical commits. Only tag a historical version when the exact source or a verifiable deployment snapshot has been recovered.

---

## 10. Critical functional regression checklist

Before every release, explicitly verify that a change did not remove unrelated application capabilities. At minimum verify:

- Main navigation remains clickable and non-static
- Admin access works
- Captain access works
- Scorer access works
- Admin can score
- Scorer can see scoring controls
- Teams page works
- Rosters page works
- Admin/authorized Edit + Save/Cancel controls remain
- Season configuration remains available
- Matches setup remains available
- Court assignment remains available
- Byes/overrides remain available
- Match scoring works
- Tournament/bracket UI does not break Matches
- Home/live-match cards render
- Existing logos/team imagery remain intact

For production recovery, prefer a surgical page/function rollback rather than rebuilding unrelated areas.

---

## 11. Secret handoff checklist

The other developer needs **platform access**, not the owner's passwords.

Give them:

- [ ] GitHub repository invitation (Admin if full owner/developer responsibility)
- [ ] Vercel team/project invitation with production/deployment/env permission
- [ ] Supabase organization/project invitation with database/functions/settings permission
- [ ] Any external third-party service invitations used by the app

Developer creates their own:

- [ ] GitHub authentication/PAT if needed
- [ ] Vercel CLI/API token if needed
- [ ] Supabase personal access token if needed

Safe values already documented here:

- [x] GitHub repo/clone URL
- [x] Vercel project/team IDs
- [x] Supabase project ref/org ID/API URL
- [x] Supabase publishable key

Never exchange via this file:

- [ ] Supabase service-role key
- [ ] Database password
- [ ] Vercel personal token
- [ ] GitHub PAT
- [ ] Supabase personal access token

Use the platform's member/role system or a secure password/secret manager for any exceptional secret transfer.

---

## 12. First actions for the incoming developer

1. Confirm GitHub, Vercel and Supabase access.
2. Clone the private repository.
3. Authenticate GitHub/Vercel/Supabase CLIs with their own accounts.
4. Inventory all production Supabase Edge Function source into GitHub without changing behavior.
5. Export/commit database migrations/schema definitions (never production secrets/data dumps).
6. Link Vercel deployment to GitHub if not already linked.
7. Add `PROJECT_CONTEXT.md` containing permanent product/role requirements.
8. Establish a known-good release tag.
9. Perform future fixes through branches/PRs/tags.
10. Before auth hardening, audit why current Edge Functions run with `verify_jwt=false` and introduce protections incrementally with regression testing.

---

## 13. Ownership principle

The original owner should retain Owner/Admin access on GitHub, Vercel and Supabase even when another developer receives full day-to-day control. This guarantees recovery if credentials, integrations, or deployments are accidentally changed.
