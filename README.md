# Agent Platform

A multi-tenant platform for running AI agents reliably: durable background execution, human approval for risky actions, and a full trace of every step an agent takes.

## Why this exists

Running an AI agent in production is harder than calling an LLM. Runs can take minutes, servers crash halfway through, provider APIs time out, and an agent that sends emails or issues refunds needs a human in the loop. When something goes wrong, teams need to see exactly what the agent did, why, and what it cost.

Agent Platform handles that layer. Teams define agents and connect tools, and the platform runs them as resumable background jobs with approval gates, per-tenant isolation, and an audit trail.

## Planned features

- Agent runs as resumable background jobs that survive worker crashes
- Tool calling with idempotency, so a retry never repeats a side effect
- Human-in-the-loop approvals for sensitive actions
- Live run viewer with a step-by-step trace, latency, and token cost
- Document search (RAG) over each tenant's uploaded files
- Multi-tenancy with role-based access control and usage limits

## Tech stack

Express + TypeScript, PostgreSQL (pgvector), Redis + BullMQ, Next.js, Docker, GitHub Actions, AWS (Terraform)

## Status

Early development. Currently building Phase 1: monorepo, auth, and tenancy.

## Docs

Architecture decisions, concept notes, and failure-mode analysis live in [`/docs`](./docs).
