# 5612 split: ODD delegation agents and built-in overrides for Claude Code

Address the review on PR #5612 (decode2, issuecomment-6320889858): split into two PRs, fix the default-model drift on ODD rows with `inherit` semantics, label, and document `general-purpose.md` as a capability expansion.
PR A reuses #5612 (branch `fix/claude-odd-and-builtin-agent-models`, force-push with lease); PR B is new (`feat/claude-builtin-agent-overrides`). Both branch from `upstream/main` independently.

## Specs

- S1 — Reviewer item 2: "**Split en 2 PRs**: (a) ODD delegation agents (#5610), (b) overrides de built-ins (#5611) — comportamientos independientes con riesgo independiente. 580+/21f también falla el gate cognitivo."
- S2 — Reviewer item 4: "**Default-model drift**: main siembra los rows `odd-*` con Sonnet, así que usuarios que nunca abrieron el picker recibirían agentes ODD pineados a Sonnet en vez de heredar el modelo de la sesión — considerá semántica `inherit` para los ODD rows también, espejando lo que #5611 hace para built-ins."
  - S2a: an ODD row without an explicit pick renders `model: inherit` and no effort; it never falls back to `default`/Sonnet.
  - S2b: presets stop seeding `odd-*`; the picker shows `[inherit]` and offers `inherit` on ODD rows (clears the pick).
  - S2c (user decision L2): a persisted map that exactly equals a legacy seeded preset (preset + `odd-*`=sonnet/default effort) is treated as inherit for the ODD rows; real Custom picks keep applying.
- S3 — Reviewer item 3: "**Label `type:feature`**."
- S4 — Reviewer item 5: "Documentar explícitamente que `general-purpose.md` (que conserva todas las tools) es una expansión de capacidades gated por pick explícito."
- S5 — Keep: "Bien elegido no usar `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` — así los picks de `review-*`/`jd-*` siguen aplicando."

## Tasks

Slices (L5: CI `pr-size-policy` caps additions+deletions at 400, tests included):

- [x] T1 — S1, S2a, S2c, S5: A1 agents + routing + installer inherit (375 lines). Branch `fix/claude-odd-and-builtin-agent-models` (#5612). Route: inline. Commit: 727cc1d7
- [x] T1b — S2b, S2c: A2 picker ODD rows inherit (168 lines), stacked on A1. Branch `feat/claude-odd-inherit-picker`. Route: inline. Commit: 55abbd74
- [x] T1c — S1: A3 telemetry for managed ODD agents (61 lines), from main. Branch `feat/claude-odd-agent-telemetry`. Route: inline. Commit: ae7b0b25
- [x] T2 — S1, S4, S5: B1 opt-in Explore/general-purpose overrides + capability-expansion docs (270 lines), from main. Branch `feat/claude-builtin-agent-overrides`. Route: inline. Commit: 4bbeee1f (c6685e68 when stacked on A2)
- [x] T2b — S1: B2 picker built-in rows (96 lines), stacked on A2 + B1. Branch `feat/claude-builtin-picker-rows`. Route: inline. Commit: 4fa775b5
- [x] T3 — S1, S3: push all, retitle/rebody #5612 as A1, open A2/A3/B1/B2, ask maintainer for labels, reply on review. Route: inline. Commit: n/a

## Log

- L1 (user, verbatim): "https://github.com/Gentleman-Programming/gentle-ai/pull/5612#issuecomment-6320889858 o revisor comento isso no pr oque quer dizer?" then "GO" after the offer to "preparar a divisão em dois branches e a mudança para `inherit` nas linhas ODD".
- L2 (user decision): legacy seed → "Tratar seed legado como inherit (Recomendado)"; delivery → "Reusar #5612 + PR novo (Recomendado)".
- L3: item 1 (issue approval gate `status:needs-review` on #5610/#5611) is maintainer-owned; not actionable here.
- L4: differential sync (base vs PR A binaries, scratch HOME): no-state -> `model: inherit`; custom picks -> haiku/opus+high/sonnet. The seeded-preset case first rendered sonnet because sync drops the `orchestrator` row before install (`internal/cli/sync.go:2297`); seed matching now ignores `orchestrator`, pinned by the "as sync passes it" test case. Other diffs: CLAUDE.md delegation line, ownership ledger, digests/timestamps only.
- L5 (user decision): PR A was 602 lines vs the 400-line CI gate; user chose "Fatiar em PRs ≤400 (Recomendado)" over size:exception. B also exceeded 400, so it was split the same way.
- L6: pre-existing/environmental failures, identical on upstream/main: `internal/shellinstaller` (e2e shared-recovery), `internal/components/agentguidance` codex case (mise codex wrapper exits 127; passes without the wrapper on PATH).
- L7: viniciosrab has no triage on upstream; `type:*` labels must be added by a maintainer.
- L8: PRs: A1 #5612 (force-pushed by the user via /tmp/pr5612/push-a1.sh; harness blocks agent force-push), A2 #5660 draft, A3 #5658, B1 #5659, B2 #5661 draft. Reply posted: issuecomment-6326247315. Pending maintainer: type:feature labels, status:approved on #5610/#5611; then rebase drafts.
- L9 (user): "https://github.com/Gentleman-Programming/gentle-ai/pull/5612 conflit". Rebased A1 onto upstream/main eecceea8b as 13fc0acd3 (local branch `rebase/5612`); the only conflict was the generated `orchestrator-module-baseline.json`, regenerated with `-update` (only the claude-code row differs from main). Tests: reviewassets, model, assets green; cli/agentguidance failures are environmental and identical on upstream/main (mise codex wrapper exit 127; `TestEngramPathGuidanceDefault` because mise GOBIN is off PATH). Agent force-push blocked again; push script `/tmp/pr5612/push-a1-rebase.sh` for the user.
