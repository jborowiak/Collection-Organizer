---
name: step
description: Implement a numbered step from docs/03-mvp-plan.md. Use when the user asks to do, implement, or continue a specific MVP plan step by number (e.g. "/step 6" or "implement step 6").
---

# Implement an MVP plan step

Args: a step number, e.g. `6`. If no number is given, ask which step.

Do exactly this, in order:

1. Read `CLAUDE.md` and `docs/03-mvp-plan.md` in full.
2. Find the entry for the requested step number in `docs/03-mvp-plan.md`. Read every document it lists under **Docs** (e.g. sections of `docs/01-requirements.md` / `docs/02-architecture.md`) before writing any code.
3. Implement that step, following "Rules for every CLAUDE CODE step" in `CLAUDE.md`:
   - Only do what the step describes. If something in the documents is wrong or missing, fix the document and add a change-log entry instead of silently deviating.
   - Follow the repo conventions in `CLAUDE.md` (layering, EF Core/Npgsql/pgvector, OpenCvSharp4, central package management, commercial-use-licensed dependencies only).
   - Never ask for or commit secrets.
4. Finish by:
   - Checking every **Done when** item for the step.
   - Ticking the step in the plan's status table.
   - Making **one local commit** named `Step N: <title>` (do not push).
