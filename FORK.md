# Fork allocation ledger — frontend

This fork (`kirisame-meguru/frontend`) carries the **per-user-per-inbound traffic stats** feature directly
on **`main`** (`main` is the feature branch). See `../FORK-RESILIENCE.md` for the sync playbook.

## Allocation table

| Namespace | Symbol | File | Fork value | Upstream-conventional (PR-time) |
|-----------|--------|------|-----------|----------------------------------|
| dependency spec (feature-required) | `@remnawave/backend-contract` | `package.json` | `file:vendor/remnawave-backend-contract-2.8.35.tgz` | keep; remap to upstream-published |

The frontend allocates **no sequential identifiers** — no error codes, no proto fields, no migrations.
Everything it adds is a descriptive unique name:

- i18n keys: `base-node-form.per-inbound-user-usage` (anchored next to `traffic-tracking`),
  `user-usage-modal.widget.by-node`, `user-usage-modal.widget.by-inbound` (anchored next to
  `traffic-statistics`) in `public/locales/en/remnawave.json`
- query/mutation hooks: `useGetStatsUserPerInboundUsage`, `useSetInboundUsageTracking`

## 2.8.0 sync notes

Upstream 2.8.0 is a large UI jump (Mantine 8→9, react-table lib swap, `DEFAULT_DATE_RANGE` →
`getDefaultDateRange()`), yet the feature's TSX (user-usage-modal tabs, node-edit `trackInboundUserUsage`
toggle, config-profile-inbounds tracking switch) rebased and type-checks clean against it. The
`edit-node` form now follows upstream's **raw-bytes** `trafficLimitBytes` (dropped the fork's
`bytesToGbUtil` wrap). `backend-contract` (2.8.35) is vendored as a `file:` tarball; `fork-build.yml`
rebuilds it from the backend fork's `libs/contract` on every CI run (committed tgz is a local-dev
fallback).

A collision on any of these would be a **visible git textual conflict** (safe), so none needs a reserved
band. The one release-coupled change is the `@remnawave/backend-contract` dependency bump, which the
feature requires (it imports `GetStatsUserPerInboundUsageCommand` / `SetInboundUsageTrackingCommand`).

## For PR / version handling

- **Keep** the backend-contract dependency-spec bump; remap to the upstream-published version.
- The lockfile is isolated in a separate `chore:` commit; on rebase conflict take upstream then `npm install`.
- After any rebase that auto-merges `remnawave.json`, eyeball for an accidental duplicate i18n key.
