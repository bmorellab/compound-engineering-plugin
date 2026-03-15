---
date: 2026-03-15
topic: ce-ideate-skill
---

# ce:ideate — Open-Ended Ideation Skill

## Problem Frame

The ce:brainstorm skill is reactive — the user brings an idea, and the skill helps refine it through collaborative dialogue. There is no workflow for the opposite direction: having the AI proactively generate ideas by deeply understanding the project and then filtering them through critical self-evaluation. Users currently achieve this through ad-hoc prompting (e.g., "come up with 100 ideas and give me your best 10"), but that approach has no codebase grounding, no structured output, no durable artifact, and no connection to the ce:* workflow pipeline.

## Requirements

- R1. ce:ideate is a standalone skill, separate from ce:brainstorm, with its own SKILL.md in `plugins/compound-engineering/skills/ce-ideate/`
- R2. Accepts an optional freeform argument that serves as a focus hint — can be a concept ("DX improvements"), a path ("plugins/compound-engineering/skills/"), a constraint ("low-complexity quick wins"), or empty for fully open ideation
- R3. Performs a deep codebase scan before generating ideas, grounding ideation in the actual project state rather than abstract speculation
- R4. Generates ~30 ideas using divergent thinking, designed to push past the "safe obvious" layer that LLMs default to
- R5. Self-critiques the full list, rejecting weak ideas with explicit reasoning — the adversarial filtering step is the core quality mechanism
- R6. Presents the top 5-7 surviving ideas with structured analysis: description, rationale, downsides, confidence score (0-100%), estimated complexity
- R7. Includes a brief rejection summary — one-line per rejected idea with the reason — so the user can see what was considered and why it was cut
- R8. Writes a durable ideation artifact to `docs/ideation/YYYY-MM-DD-<topic>-ideation.md` (or `YYYY-MM-DD-open-ideation.md` when no focus area). This compounds — rejected ideas prevent re-exploring dead ends, and un-acted-on ideas remain available for future sessions.
- R9. The default volume (~30 ideas, top 5-7 presented) can be overridden by the user's argument (e.g., "give me your top 3" or "go deep, 100 ideas")
- R10. Handoff options after presenting ideas: brainstorm a selected idea (feeds into ce:brainstorm), refine the ideation (dig deeper, re-evaluate, explore different angles), or end the session
- R11. Always routes to ce:brainstorm when the user wants to act on an idea — ideation output is never detailed enough to skip requirements refinement
- R12. Session completion: when ending, offer to commit the ideation doc to the current branch. If the user declines, leave the file uncommitted. Do not create branches or push — just the local commit.
- R13. Resume behavior: when ce:ideate is invoked, check `docs/ideation/` for recent ideation docs. If a relevant one exists, offer to continue from it (add new ideas, revisit rejected ones, act on un-explored ideas) or start fresh.
- R14. The ideation doc is written to disk before presenting ideas to the user — ensures the artifact survives even if the session is interrupted or the conversation context is lost.
- R15. When the user refines (digs deeper, re-evaluates), the ideation doc is updated in place with the revised analysis.
- R16. When the user picks an idea to brainstorm, the ideation doc is updated to mark that idea as "explored" with a reference to the resulting brainstorm session date, so future revisits show which ideas have been acted on.

## Success Criteria

- A user can invoke `/ce:ideate` with no arguments on any project and receive genuinely surprising, high-quality improvement ideas grounded in the actual codebase
- Ideas that survive the filter are meaningfully better than what the user would get from a naive "give me 10 ideas" prompt
- The ideation artifact persists and provides value when revisited weeks later
- The skill composes naturally with the existing pipeline: ideate → brainstorm → plan → work

## Scope Boundaries

- ce:ideate does NOT produce requirements, plans, or code — it produces ranked ideas
- ce:ideate does NOT modify ce:brainstorm's behavior — discovery of ce:ideate is handled through the skill description and catalog, not by altering other skills
- The skill does not do external research (competitive analysis, similar projects) in v1 — this could be a future enhancement but adds cost and latency without proven need
- No configurable depth modes in v1 — fixed volume with argument-based override is sufficient

## Key Decisions

- **Standalone skill, not a mode within ce:brainstorm**: The workflows are fundamentally different cognitive modes (proactive/divergent vs. reactive/convergent) with different phases, outputs, and success criteria. Combining them would make ce:brainstorm harder to maintain and blur its identity.
- **Durable artifact in docs/ideation/**: Discarding ideation results is anti-compounding. The file is cheap to write and provides value when revisiting un-acted-on ideas or avoiding re-exploration of rejected ones.
- **Always route to ce:brainstorm for follow-up**: At ideation depth, ideas are one-paragraph concepts — never detailed enough to skip requirements refinement.
- **Survivors + rejection summary output format**: Full transparency on what was considered without overwhelming with detailed analysis of rejected ideas.
- **Freeform optional argument**: A concept, a path, or nothing at all — the skill interprets whatever it gets as context. No artificial distinction between "focus area" and "target path."

## Outstanding Questions

### Deferred to Planning

- [Affects R3][Technical] Which agents should be dispatched for the codebase scan? repo-research-analyst is obvious, but should learnings-researcher or git-history-analyzer also run?
- [Affects R4][Needs research] What prompt structure within the skill produces the best divergent ideation? The user's proven prompts provide a starting pattern but may need adaptation for the skill context.
- [Affects R8][Technical] What frontmatter fields should the ideation artifact include? At minimum date and topic, but could include focus-area, idea-count, or project-name.
- [Affects R6][Technical] Should the structured analysis per idea include "suggested next steps" or "what this would unlock" beyond the current fields (description, rationale, downsides, confidence, complexity)?
- [Affects R2][Technical] How should the skill detect volume overrides in the freeform argument vs. focus-area hints? Simple heuristic or explicit parsing?

## Next Steps

→ `/ce:plan` for structured implementation planning
