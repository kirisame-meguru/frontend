# Fork allocation ledger — frontend

This fork (`kirisame-meguru/frontend`) carries the **per-user-per-inbound traffic stats** feature on
branch `per-user-per-inbound-traffic-stats`. See `../FORK-RESILIENCE.md` for the sync playbook.

## Allocation table

| Namespace | Symbol | File | Fork value | Upstream-conventional (PR-time) |
|-----------|--------|------|-----------|----------------------------------|
| dependency spec (feature-required) | `@remnawave/backend-contract` | `package.json` | 2.8.0 | keep; remap to upstream-published |

The frontend allocates **no sequential identifiers** — no error codes, no proto fields, no migrations.
Everything it adds is a descriptive unique name:

- i18n keys: `base-node-form.per-inbound-user-usage`, `user-usage-modal.widget.by-node`,
  `user-usage-modal.widget.by-inbound` (`public/locales/en/remnawave.json`)
- query/mutation hooks: `useGetStatsUserPerInboundUsage`, `useSetInboundUsageTracking`

A collision on any of these would be a **visible git textual conflict** (safe), so none needs a reserved
band. The one release-coupled change is the `@remnawave/backend-contract` dependency bump, which the
feature requires (it imports `GetStatsUserPerInboundUsageCommand` / `SetInboundUsageTrackingCommand`).

## For PR / version handling

- **Keep** the backend-contract dependency-spec bump; remap to the upstream-published version.
- The lockfile is isolated in a separate `chore:` commit; on rebase conflict take upstream then `npm install`.
- After any rebase that auto-merges `remnawave.json`, eyeball for an accidental duplicate i18n key.
