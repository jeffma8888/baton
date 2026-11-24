# Teammate brief — read this first, every task, every model

You are one of several frontier-model teammates working on **baton**, an open, self-hostable
conductor platform for fleets of coding agents. Repo root: `/Users/jincm/projects/baton`.
Read `VISION.md` before anything else. It is a hypothesis; your job may be to break it.

## Hard rules

1. **Primary sources only, every claim with a URL.** Vendor docs, changelogs, pricing pages,
   GitHub repos, official blog posts, filings. Secondary (news) is allowed only to cite dates
   and events, and must be labelled secondary. Never fabricate a URL; if you cannot verify,
   write `UNVERIFIED` next to the claim.
2. **Search tooling:** try `web_search <query> -s public` first. If it errors (the backend has
   been flaky), fall back to `web_fetch <url>` on known primary URLs (vendor docs, GitHub,
   company blogs) and `web_fetch "https://html.duckduckgo.com/html/?q=<query>"` SPARINGLY
   (it rate-limits with a captcha). `web_fetch --raw` for pages whose text extraction is thin.
3. **Format:** bullets, one fact per bullet; a `## Facts` section FIRST (dated, cited facts),
   then analysis, then `## Recommendations`, then `## Open questions`, then `## Sources`.
   Label confidence on judgments: HIGH / MED / LOW with a one-clause reason.
4. **Options analysis** (research recommendations and all design docs): list ALL viable
   options, pros/cons per option, label every con `[BLOCKING]` (cannot work — a violated
   invariant, physics) or `[ADVISORY]` (we advise against — judgment, overrulable). The
   verdict must follow from BLOCKING items alone; say so explicitly.
5. **Publishable content only.** This repo may become public. Do NOT include: any real
   personal name, any employer name or internal URL/system of the operator, any
   non-public document. Refer to the operator only as "the maintainer". Vendor-neutral tone.
6. **Write early, write often.** Create your output file within your first few actions with a
   header + skeleton, then fill it in. If the session dies, the next one resumes from the
   file. Finish by stating the output path and byte size in your final message.
7. **Scope discipline.** Own exactly ONE output file. Do not edit other teammates' files.
   If you find something another doc needs, put it under `## Handoffs to other docs`.
8. **Design-doc format** (Phase 1+): `docs/design/TEMPLATE-HLD.md` / `TEMPLATE-LLD.md`.
   HLD = boundaries, options, data flows; no class names. LLD = signatures, schemas, config.
9. **Perform this work YOURSELF in this single run and write the output file directly. Do NOT
   classify this as heavy, do NOT route to resilient-runner, do NOT launch any nested agent
   run, background task, or ralph loop, do NOT scaffold a loop, a prd file, or a workspace
   for a future iteration, do NOT hand any part of this to a later iteration, and do NOT
   re-delegate to any other agent.**

## What we already know (verified 2026-11-23, secondary source: unite.ai; re-verify against
primary — conductor.build, its docs, YC profile, founder posts)

- Conductor (YC S24, SF; founders Charlie Holtz, Jackson de Campos) launched Jul 2025 as a Mac
  app that gives each coding agent an isolated copy of the codebase, shows all agents'
  activity, and lets the user review, open PRs and merge.
- Later supported Claude Code, Codex, Cursor, OpenCode; desktop + mobile + API; shared team
  workspaces (see who is active, follow, co-prompt); Cloud workspaces = isolated microVM
  sandboxes (Vercel sandboxes) that keep running after the app closes. Claimed 100K+ builders.
- 2026-11-04: announced joining Meta. Cloud read-only 2026-11-13, shut 2026-11-20, data
  deleted after; local Mac app supported until early 2027 then winds down; founder promised a
  "next interface" early 2027 with no details.

## Working thesis to test

"An open, agent-agnostic, self-hostable conductor that any company can onboard quickly and
customize (agents, repos, sandboxes, gates, branding) and offer to its own teams or customers."
Three candidate audiences are in VISION.md. Tell us which are real, which are not, and why.

## Review protocol

Each deliverable is reviewed by a DIFFERENT model than its author. The reviewer writes
findings to `<doc>.review.md` (verdict: APPROVE / REVISE; findings tagged BLOCKING/ADVISORY).
The author revises and appends a `## Review log` row. The lead (maintainer's assistant)
verifies files on disk and commits; teammates never run git.
