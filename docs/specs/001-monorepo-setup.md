# Spec 001: Monorepo and tooling setup

## Goal

Create a pnpm workspace monorepo with separate apps (api, worker, web) and
shared packages (shared, db). TypeScript, ESLint, and Prettier are configured
once at the root and inherited by every package.

## Why this exists

- One repo keeps shared types in sync between the API and the worker.
- Splitting api and worker from day one enforces the rule that the API never
  runs agents itself, it only enqueues them.
- Shared tooling keeps code style and compiler settings consistent.

## Out of scope

No application logic, no database, no Docker, no CI. Those are later specs.

## Structure

```
agent-platform/
├── package.json              # root: dev tools + workspace scripts
├── pnpm-workspace.yaml       # lists the workspace folders
├── tsconfig.base.json        # shared compiler settings
├── .nvmrc                    # pinned Node version
├── eslint config, .prettierrc
├── apps/
│   ├── api/                  # package.json + tsconfig.json (+ src/)
│   ├── worker/               # package.json + tsconfig.json (+ src/)
│   └── web/                  # package.json + tsconfig.json
└── packages/
    ├── shared/               # package.json + tsconfig.json (+ src/index.ts)
    └── db/                   # package.json + tsconfig.json (+ src/index.ts)
```

Every app and package keeps its OWN package.json (different dependencies,
built and deployed separately). The root package.json holds only shared dev
tooling and scripts.

## Behavior

- Inputs: none (setup task, no application logic)
- Outputs: a repo where `pnpm install` at the root sets up everything
- Happy path:
  - `pnpm install` works on a fresh clone
  - `pnpm -r typecheck` and `pnpm lint` pass across all workspaces
  - `pnpm --filter api <script>` runs a script in one app only
  - `apps/api` can import a type from `@agent-platform/shared`

## Key mechanics

- **pnpm-workspace.yaml** lists which folders are workspace packages:
  `apps/*` and `packages/*`. It does not merge package.json files.
- **Package names** use the `@agent-platform/` scope, for example
  `@agent-platform/shared` and `@agent-platform/api`.
- **Workspace dependencies** are declared with `workspace:*`, so pnpm links
  the local folder instead of downloading from the npm registry.
- **tsconfig.base.json** holds compiler settings shared by everyone. Each
  package has its own tsconfig.json that `extends` the base and adds only
  what's specific to it (for example `web` needs JSX settings, `api` doesn't).
- **`pnpm -r`** runs a command in every workspace package, in dependency
  order, skipping packages without that script.

## Failure cases

- Different Node/pnpm versions across machines: pin them with the
  `packageManager` field and `.nvmrc`.
- A package imports another package it never declared: pnpm must fail
  loudly (strictness is a feature).
- Shared package not found by an app: workspace linking or the shared
  package's name/entry point is misconfigured.

## Files touched

- Root: package.json, pnpm-workspace.yaml, tsconfig.base.json, ESLint
  config, .prettierrc, .nvmrc
- Each of apps/api, apps/worker, apps/web, packages/shared, packages/db:
  package.json and tsconfig.json (plus a minimal src/index.ts where needed)
- No application logic

## How I'll verify it

- Delete node_modules, run `pnpm install`: it succeeds
- `pnpm
