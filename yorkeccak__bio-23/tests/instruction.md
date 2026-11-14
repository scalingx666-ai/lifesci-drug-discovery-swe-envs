# Issue

**Title:** Upgrade Next.js to 16.4.5 and React to 19.3.1

## Description

## Why
Patches GHSA-xqc2-xf4c-x3h3 (critical, unauthenticated RCE in React Server Components). Next.js vendors its own copy of the RSC runtime, so the fix requires Next.js 16.4.1+ (or 15.5.28+); this moves to the latest release.

## Versions
| Package | Before | After |
|---|---|---|
| next | 15.4.8 | 16.4.5 |
| react / react-dom | 19.1.2 | 19.3.1 |
| eslint-config-next | 15.4.8 | 16.4.5 |
| @types/react / @types/react-dom | 19.1.10 | 19.3.0 |
| jspdf | 3.0.3 | 4.2.1 (critical advisories, no 3.x fix) |
| drizzle-orm | 0.44.7 | 0.45.4 (high advisory) |
| zod | 4.0.17 | 4.6.7 (high advisory) |

## Audit (`pnpm audit`)
| | critical | high | moderate | low |
|---|---|---|---|---|
| Before | 10 | 101 | 86 | 15 |
| After | 0 | 2 | 3 | 5 |

Transitive findings are fixed with `pnpm.overrides`. Most entries float a package to the latest release within the major version it already uses (e.g. `axios@^1.0.0 -> ^1.21.0`). Three are scoped to a parent because no fix exists within the major:
- `@daytonaio/sdk>tar` -> 7.x (the SDK only calls `tar.create`, which works the same in 7.x)
- `chromium-bidi>zod` -> 4.x (only used for WebDriver BiDi; puppeteer uses CDP by default)
- `get-uri>basic-ftp` -> 6.x (same `Client` API)

Remaining:
- `extract-zip` (high, 2 advisories): no patched version exists. It comes in through `puppeteer` -> `@puppeteer/browsers`, which uses it to download browsers at install time. Only puppeteer 25 drops it, and puppeteer 25 needs Node >= 22.12 (the Dockerfile uses Node 20), so that upgrade is left for a separate change.
- Moderates in `esbuild` (via drizzle-kit, dev only), `minimatch` 10 (via typescript-eslint, dev only) and `uuid` 9 (via the AWS SDK in the Daytona SDK) need a new major version.

## Code changes
- `eslint.config.mjs`: `next lint` was removed in Next 16, so the lint script is now `eslint .`, using the native flat config from `eslint-config-next` 16. The React Compiler rules added in `eslint-plugin-react-hooks` v7 (`set-state-in-effect`, `refs`, `static-components`, `immutability`, `purity`, `preserve-manual-memoization`) flag existing code, so they are set to warn rather than fail lint.
- `tsconfig.json`: the settings `next build` rewrites itself (`jsx: react-jsx`, include `.next/dev/types`).
- `middleware.ts` is unchanged. It is deprecated in favour of `proxy.ts` but still works (the build reports it as `Proxy (Middleware)`).

## Verification
- Clean `pnpm install`
- `tsc --noEmit`: passes
- `pnpm lint`: 0 errors (28 warnings, all from the newly added rules or `no-img-element`)
- `next build` (Turbopack): passes, all 29 routes
- `next start` smoke test: `/`, `/reports` and `/api/env-status` return 200
- Lockfile resolves next 16.4.5, react 19.3.1, react-dom 19.3.1

## Task

Modify the repository so that the issue described above is resolved. The repository is checked out at base commit `df85451102b3`. Edit files in place; the verifier captures your changes via `git diff` and scores them against an oracle patch using SWE-RL-style diff-similarity reward.