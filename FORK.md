# Fork allocation ledger — frontend

This fork (`kirisame-meguru/frontend`) carries the **per-user-per-inbound traffic stats** feature directly
on **`main`** (`main` is the feature branch). See `../FORK-RESILIENCE.md` for the sync playbook.

## Allocation table

| Namespace | Symbol | File | Fork value | Upstream-conventional (PR-time) |
|-----------|--------|------|-----------|----------------------------------|
| dependency spec (feature-required) | `@remnawave/backend-contract` | `package.json` | `file:vendor/remnawave-backend-contract-3.4.3.tgz` | keep; remap to upstream-published |

The frontend allocates **no sequential identifiers** — no error codes, no proto fields, no migrations.
Everything it adds is a descriptive unique name:

- i18n keys: `user-usage-modal.widget.by-node`, `user-usage-modal.widget.by-inbound` (in the
  `user-usage-modal.widget` block) in `public/locales/en/remnawave.json` — the only locale file the
  fork touches
- query hook: `useGetStatsUserPerInboundUsage`

## The panel does not switch tracking on

Per-user-per-inbound tracking is enabled inside the xray config itself
(`$.inbounds[].trackTrafficPerUser`), edited through the config profile editor. The frontend only
*displays* the collected usage (the per-inbound tab of the user-usage modal). The two switches that
used to control it — node `trackInboundUserUsage` and config-profile-inbound `trackUserUsage`, the
latter via `useSetInboundUsageTracking` — are gone, along with the
`base-node-form.per-inbound-user-usage` locale key.

## 3.3.2 sync notes

Upstream 3.3.2 moved the user-usage modal to `src/shared/_modals/users/user-usage-modal/user-usage.modal.tsx`
(was `src/widgets/dashboard/users/user-usage-modal/user-usage-modal.widget.tsx`) and rebuilt it around
`NiceModal.create` + `CompoundDrawerShared`; the fork's by-node/by-inbound `Tabs` split was re-applied on
top of that shape. The stats route param moved from `uuid: string` to `userId: number`, and the commands
now export `RequestParamSchema`/`RequestParam` instead of `RequestSchema`/`Request` — the fork's
`useGetStatsUserPerInboundUsage` follows, and the backend fork's `GetStatsUserPerInboundUsageCommand`
(contract 3.4.3, route constant `GET_INBOUNDS_BY_ID`) mirrors it. Upstream also dropped the
`user-usage-modal.widget.traffic-statistics` key and deleted
`config-profile-inbounds.drawer.widget.tsx`. `backend-contract` (3.4.3) is referenced as a `file:`
tarball that `fork-build.yml` rebuilds from the backend fork's `libs/contract` on every CI run — no tgz
is committed anymore, so a local dev build must `npm pack` it first.

A collision on any of these would be a **visible git textual conflict** (safe), so none needs a reserved
band. The one release-coupled change is the `@remnawave/backend-contract` dependency bump, which the
feature requires (it imports `GetStatsUserPerInboundUsageCommand`).

## For PR / version handling

- **Keep** the backend-contract dependency-spec bump; remap to the upstream-published version.
- The lockfile is isolated in a separate `chore:` commit; on rebase conflict take upstream then `npm install`.
- After any rebase that auto-merges `remnawave.json`, eyeball for an accidental duplicate i18n key.
