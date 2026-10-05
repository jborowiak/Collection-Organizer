# CLAUDE.md

## Project

Sugar Bag Collection Organizer — a personal collection cataloguing app with a "check if I already have it" duplicate-detection feature based on photo matching (DINOv2 embeddings + SIFT geometric verification + fuzzy text), built as a Blazor WASM PWA over an ASP.NET Core Minimal API, deployed to Azure Container Apps + Static Web Apps with a Neon PostgreSQL (pgvector) database.

The documents in `docs/` are the source of truth:
- `docs/01-requirements.md` — functional/non-functional requirements
- `docs/02-architecture.md` — architecture, algorithm design, data model, deployment, ADRs
- `docs/03-mvp-plan.md` — the step-by-step implementation plan (this is the plan you are executing)

## Conventions

- C# / .NET 10, ASP.NET Core **Minimal APIs**.
- EF Core with **Npgsql** + **pgvector** (`Pgvector.EntityFrameworkCore`) for the `vector(384)` column. No HNSW index (architecture §8.1) — brute-force cosine search via SQL.
- **Modular monolith**: `Domain ← Application ← Infrastructure/Vision/Worker ← Api`; `UI ← Web`. Don't add new cross-layer references that invert this.
- ML/vision behind interfaces: `IImageEmbedder`, `IGeometricVerifier`, `ITextExtractor`. Implementations live in `SugarBags.Vision` / `SugarBags.Infrastructure`.
- **OpenCvSharp4** for image work (not Emgu CV). Prefer OpenCvSharp over ImageSharp.
- All libraries and models used must allow **commercial use** (NFR-12) — check the license before adding a dependency.
- Central package management: add new package versions to `Directory.Packages.props`, reference without a version in individual `.csproj` files.

## Rules for every CLAUDE CODE step in `docs/03-mvp-plan.md`

1. Read the step, the document sections it lists under **Docs**, and this file before starting.
2. Only do what the step describes. If something in the documents is wrong or missing, fix the document and add a change-log entry instead of silently deviating.
3. Finish by checking every **Done when** item, ticking the step in the plan's status table, and making **one local commit** named `Step N: <title>`.
4. **Never** ask for secrets (passwords, connection strings, client secrets) in the chat and never commit them. Secrets go into .NET user-secrets locally, Container Apps secrets in Azure, and GitHub Actions secrets in CI — the user types them in themselves.
5. Claude Code commits locally; the user pushes (`git push`) when they are happy with the step, or explicitly asks Claude Code to push.

## Common commands

```bash
# Build everything
dotnet build

# Run all tests
dotnet test

# Run the API (https://localhost:7200)
dotnet run --project src/SugarBags.Api

# Run the Web app (https://localhost:7100)
dotnet run --project src/SugarBags.Web

# Run the evaluation tool (once testdata/ exists — see step 3)
dotnet run --project tools/SugarBags.Eval -- --data testdata
```

Local PostgreSQL + Azurite via Docker, and EF Core migration commands, are documented once step 10 adds `docker-compose.yml`.
