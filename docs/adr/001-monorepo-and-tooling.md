# ADR 001: Monorepo and tooling

**Status:** Accepted
**Date:** 2026-10-12

## Context

The platform has two backend processes, the API and the worker, plus a
dashboard. The API creates agent runs and the worker executes them, so both
must agree exactly on what a run looks like (its fields and statuses). If they
lived in separate repos, those types would be copied by hand and drift apart.
In one repo, both import them from `packages/shared`, and a type change lands
in a single commit.

Separate folders for `api` and `worker` also enforce a design rule: the API
only enqueues runs and never executes agents itself.

## Options considered

- **Separate repos:** clean boundaries, but shared types must be duplicated or
  published as a package, which is heavy for a project this size.
- **npm workspaces:** no new tool to learn, but looser about undeclared
  imports, slower installs, and less convenient for running one app.
- **Yarn Berry:** strong workspace support, but Plug'n'Play can cause tooling
  friction, and its docs are easy to confuse with Yarn v1.
- **pnpm workspaces:** strict about undeclared dependencies, fast and
  disk-efficient installs, `--filter` to run a single app, and builds that run
  in dependency order.

## Decision

A monorepo, managed with pnpm workspaces.

These are two separate decisions. The monorepo is for shared types and one
place for tooling. pnpm is for strictness (a package can only import what it
declares), dependency-ordered `pnpm -r` commands, and `--filter`.

Layout: `apps/` (api, worker, web) and `packages/` (shared, db). Every package
keeps its own `package.json`. Internal dependencies use `workspace:*`, which
links the local folder instead of downloading from the registry. TypeScript
settings live in `tsconfig.base.json`, and each package extends it.

## Tradeoffs and consequences

- **`shared` exposes raw TypeScript source, not built JavaScript.** This saves
  a build step: an edit in `shared` shows up in `api` immediately. The cost is
  that anything consuming `shared` must be able to compile TypeScript. Today
  only `tsc --noEmit` runs, so nothing executes yet. Before production,
  `shared` has to be compiled from TypeScript to JavaScript, either bundled
  into the apps' output (tsup/esbuild), built to `dist/`, or handled with
  `pnpm deploy`, because a Docker image that copies only `apps/api` will not
  contain `packages/shared`. Decision deferred to the deployment phase.

- **`shared` has only an `exports` field, with no `main` or `types`.** Modern
  tools (including our `moduleResolution: "Bundler"` setting) read `exports`.
  Older tools using the legacy `"node"` resolution ignore it, look for `main`
  and `types`, and fail with "cannot find module". `exports` also blocks deep
  imports into the package's internals, which keeps its boundary clean.

- **Follow-ups:** add `.gitattributes` for consistent line endings before CI
  runs on Linux, and review the deprecated-ESLint notice at install time.
