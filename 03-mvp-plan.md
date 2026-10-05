# Sugar Bag Collection Organizer — MVP Implementation Plan

| | |
|---|---|
| **Document** | MVP implementation plan (step by step) |
| **Version** | 1.1 (repository confirmed public) |
| **Related** | `01-requirements.md` (v0.3), `02-architecture.md` (v0.5) |
| **Location in repo** | `docs/03-mvp-plan.md` |

---

## How to use this plan

Each step is one unit of work. There are two types:

- **CLAUDE CODE** — run it in Claude Code from the repository root with a prompt such as:
  > Read `CLAUDE.md` and `docs/03-mvp-plan.md`, then implement step 6.
- **EXTERNAL** — things Claude Code cannot do for you (cloud portals, accounts, your phone, photographing bags). The step lists exactly what to click or type. You can still ask Claude Code *"Walk me through step 19"* or *"Give me the exact commands for step 23"* if you get stuck.

**Rules for every CLAUDE CODE step** (also written into `CLAUDE.md` in step 2):
1. Read the step, the document sections it lists under **Docs**, and `CLAUDE.md` before starting.
2. Only do what the step describes. If something in the documents is wrong or missing, fix the document and add a change-log entry instead of silently deviating.
3. Finish by checking every **Done when** item, ticking the step in the status table below, and making **one local commit** named `Step N: <title>`.
4. **Never** ask for secrets (passwords, connection strings, client secrets) in the chat and never commit them. Secrets go into .NET user-secrets locally, Container Apps secrets in Azure, and GitHub Actions secrets in CI — you type them in yourself.

**Git:** Claude Code commits locally; you push (`git push`) when you are happy with the step — or tell Claude Code to push.

---

## Status

| # | Step | Type | Done |
|---|---|---|---|
| **A** | **Setup** | | |
| 1 | Prepare your machine and the repository | EXTERNAL | ☑ |
| 2 | Scaffold the solution and `CLAUDE.md` | CLAUDE CODE | ☐ |
| **B** | **Algorithm spike (go/no-go)** | | |
| 3 | Photograph the test set | EXTERNAL | ☐ |
| 4 | Get the DINOv2 model (ONNX) | CLAUDE CODE | ☐ |
| 5 | Vision core: normalization, corner detection, embeddings | CLAUDE CODE | ☐ |
| 6 | Evaluation tool v1: retrieval accuracy | CLAUDE CODE | ☐ |
| 7 | Geometric verification, score fusion, tiers + evaluation v2 | CLAUDE CODE | ☐ |
| 8 | Go/no-go review of the algorithm | EXTERNAL | ☐ |
| **C** | **Backend** | | |
| 9 | Create the Neon database and verify extensions | EXTERNAL | ☐ |
| 10 | Local development environment (Docker) | CLAUDE CODE | ☐ |
| 11 | Domain, persistence and file storage | CLAUDE CODE | ☐ |
| 12 | Catalog API | CLAUDE CODE | ☐ |
| 13 | Photo indexing pipeline | CLAUDE CODE | ☐ |
| 14 | Duplicate check API | CLAUDE CODE | ☐ |
| **D** | **Frontend** | | |
| 15 | Blazor PWA shell | CLAUDE CODE | ☐ |
| 16 | Collection browsing UI | CLAUDE CODE | ☐ |
| 17 | Add item, photo capture and duplicate check UI | CLAUDE CODE | ☐ |
| **E** | **Authentication** | | |
| 18 | Azure account, budget alert and resource group | EXTERNAL | ☐ |
| 19 | Entra External ID tenant, app registrations, user flow | EXTERNAL | ☐ |
| 20 | Google sign-in: OAuth client and federation | EXTERNAL | ☐ |
| 21 | Integrate authentication in API and UI | CLAUDE CODE | ☐ |
| **F** | **Deployment** | | |
| 22 | Infrastructure as code (Bicep) and backup job | CLAUDE CODE | ☐ |
| 23 | Deploy the Azure infrastructure | EXTERNAL | ☐ |
| 24 | Dockerfile and GitHub Actions (CI/CD) | CLAUDE CODE | ☐ |
| 25 | Connect GitHub to Azure | EXTERNAL | ☐ |
| 26 | First release and acceptance test | EXTERNAL | ☐ |

---

## Values you will collect

Keep a private note (a password manager is ideal). **Non-secret** values may be committed in config files; **secret** values must never be committed or pasted into Claude Code.

| Value | From step | Secret? | Where it ends up |
|---|---|---|---|
| Neon pooled connection string | 9 | **Yes** | Container Apps secret (step 23); backup job secret |
| Azure subscription ID, region, resource group name | 18 | No | Bicep parameters, GitHub secrets (step 25) |
| External ID tenant ID and tenant subdomain | 19 | No | `appsettings*.json` (API and Web) |
| SPA (web) app client ID | 19 | No | Web `appsettings*.json` |
| API app client ID and scope `api://<api-client-id>/access_as_user` | 19 | No | API and Web config |
| Google OAuth client ID | 20 | No | Entered in Entra only |
| Google OAuth client secret | 20 | **Yes** | Entered in Entra only |
| API URL and Static Web App URL | 23 | No | Config, External ID redirect URI |
| Static Web Apps deployment token | 25 | **Yes** | GitHub secret |
| GitHub deploy identity client ID | 25 | No (but treat as sensitive) | GitHub secret |

---

## Decisions made in this plan

These implementation choices are not yet in the architecture document. Step 22 records them there.

| Decision | Reason |
|---|---|
| Local development uses Docker (PostgreSQL with pgvector, Azurite storage emulator); **Neon is used only for the deployed app** | Experiments and migrations never touch real data; no internet needed for development |
| Database migrations are applied by the API at startup; the Container App is limited to **max 1 replica** | Simplest safe option for a single-user MVP |
| Secrets live in **Container Apps secrets**, not Key Vault | Saving variant from the cost analysis; one less service |
| Storage access from the API uses the Container App's **managed identity** (no storage keys in the app) | Fewer secrets |
| Nightly `pg_dump` writes to an **Azure Files share** in the same storage account (mounted into the backup job) instead of a blob container | The standard PostgreSQL image has no Blob upload tool; a mounted share needs no custom image |
| Fixed local ports: Web `https://localhost:7100`, API `https://localhost:7200` | The login redirect URI registered in step 19 must match exactly |

---

# Phase A — Setup

## Step 1 — Prepare your machine and the repository

**Type:** EXTERNAL · **Depends on:** — · **Time:** ~1 hour

**Goal:** all tools installed, the repository cloned, the three documents in `docs/`.

1. **Install** (accept defaults unless noted):
   - **.NET 10 SDK** — from `dotnet.microsoft.com/download`. Check: `dotnet --version` prints `10.x`.
   - **Git** — check: `git --version`.
   - **Docker Desktop** — needed from step 10 (local database) and for tests. Start it once and let it finish setup.
   - **Claude Code** — follow the official installation instructions at `docs.claude.com` (Claude Code section). Check: `claude --version`.
   - **Azure CLI** — needed from step 18. Check: `az --version`.
   - *Optional:* **Python 3.11+** — only if step 4 needs to export the model itself.
   - *Optional editor:* Visual Studio 2022/2026 or VS Code with the C# Dev Kit.
2. **Repository visibility: confirmed PUBLIC.** This decides the container image setup (public ghcr.io image, architecture §8.2). Because the repository is public, remember: **never commit secrets, personal photos or the `testdata/` folder** (it is git-ignored in step 2), and anything in the repository is readable by everyone.
3. **Clone the repository:**
   ```bash
   git clone https://github.com/jborowiak/Collection-Organizer.git
   cd Collection-Organizer
   ```
4. **Add the documents:** create a folder `docs` and copy `01-requirements.md`, `02-architecture.md` and `03-mvp-plan.md` into it.
5. **Commit and push:**
   ```bash
   git add docs
   git commit -m "Add requirements, architecture and MVP plan"
   git push
   ```
6. Start Claude Code in the repository root: `claude`.

**Done when:** all tool checks print a version and `docs/` with the three files is on GitHub.

---

## Step 2 — Scaffold the solution and `CLAUDE.md`

**Type:** CLAUDE CODE · **Depends on:** 1 · **Docs:** architecture §4.3, §9 (ADRs), this plan's "How to use" and "Decisions" sections

**Goal:** an empty but buildable solution with the agreed structure, and a `CLAUDE.md` that makes every later session start with the right context.

**Tasks**
1. Create `SugarBags.sln` with the projects from architecture §4.3: `SugarBags.Api` (ASP.NET Core Minimal API), `SugarBags.Application`, `SugarBags.Domain`, `SugarBags.Infrastructure`, `SugarBags.Vision`, `SugarBags.Worker` (class library; runs in-process inside the API for the MVP), `SugarBags.UI` (Razor Class Library), `SugarBags.Web` (Blazor WebAssembly with PWA support), `tools/SugarBags.Eval` (console). **Do not** create `SugarBags.Mobile` (Phase 2). Add matching test projects (`xUnit`) under `tests/`.
2. Target **.NET 10**. Add `Directory.Build.props` (nullable enabled, warnings as errors for new code, implicit usings), `Directory.Packages.props` (central package management), `.editorconfig`, and a .NET `.gitignore` extended with: `models/*.onnx`, `testdata/`, `eval-results/`, `*.dump`.
3. Set up project references following the layering (Domain ← Application ← Infrastructure/Vision ← Api; UI ← Web).
4. Write **`CLAUDE.md`** at the repository root containing:
   - one-paragraph project summary and links to the three documents (documents are the source of truth);
   - conventions: C#/.NET 10, Minimal APIs, EF Core with Npgsql + pgvector, modular monolith, ML behind interfaces (`IImageEmbedder`, `IGeometricVerifier`, `ITextExtractor`), **OpenCvSharp4 (not Emgu CV)**, prefer OpenCvSharp over ImageSharp for image work, all libraries and models must allow commercial use (NFR-12);
   - the step rules from "How to use this plan" (read docs first, one commit per step, tick the status table, update docs instead of deviating, never handle secrets);
   - common commands (build, test, run API, run Web, run Eval) — fill in as they become available.
5. Update `README.md`: short description, link to docs, "how to build".

**Done when:** `dotnet build` and `dotnet test` succeed on the empty solution; `CLAUDE.md` exists; step 2 ticked; committed.

---

# Phase B — Algorithm spike (go/no-go)

> Architecture §11 says: *do not build UI until the algorithm works.* Steps 3–8 prove it on your real bags.

## Step 3 — Photograph the test set

**Type:** EXTERNAL · **Depends on:** — (can be done in parallel with steps 1–2) · **Docs:** requirements §7 · **Time:** 2–4 hours

**Goal:** a labelled photo set that measures how well the duplicate check works on *your* bags.

1. **Pick ~120 bags:**
   - **~100 bags** for the "collection" (include ~20 **tricky pairs**: two different bags that look alike — same café or brand, different design or number);
   - **~20 bags** that will play "not in my collection" (distractors).
2. **Create this folder structure** in the repository root (it is git-ignored, so the photos stay on your machine):
   ```
   testdata/
     gallery/        one "catalogue" photo per collection bag: 001.jpg, 002.jpg, … 100.jpg
     queries/        a SECOND photo of the same bag, same number: 001.jpg, 002.jpg, …
     distractors/    photos of the ~20 extra bags, any names
     tricky.csv      one line per tricky pair, e.g.  014,015
   ```
3. **Gallery photos** (how you would normally catalogue a bag): front side, plain background, good light, phone held flat above the bag.
4. **Query photos** — take them **later, in different conditions**, like in a café: different light, slight angle (up to ~30°), different background, some held in the hand, some slightly crumpled. Same side (front) as the gallery photo.
5. Use your normal phone camera, JPG, default resolution. Don't edit or crop the photos.

**Done when:** `testdata/` has ~100 gallery photos, matching query photos with the same numbers, ~20 distractors and `tricky.csv`.

---

## Step 4 — Get the DINOv2 model (ONNX)

**Type:** CLAUDE CODE · **Depends on:** 2 · **Docs:** architecture §2.2 (Stage 1), §2.3

**Goal:** a reproducible way to obtain `models/dinov2-small.onnx` (ViT-S/14, Apache-2.0) on any machine and in CI.

**Tasks**
1. Write `tools/get-model.ps1` **and** `tools/get-model.sh` that obtain the model into `models/dinov2-small.onnx`:
   - preferred: download an existing ONNX export of `facebook/dinov2-small` from Hugging Face (verify the source repository and its license before using it);
   - fallback: export it with Python (`optimum` ONNX export of `facebook/dinov2-small`), documented in the script.
2. Verify the file with a SHA-256 checksum stored in `models/model.lock` (the scripts fail if the checksum differs).
3. Record the **model version string** (e.g. `dinov2-vits14-onnx-1`) in one shared constant — it is stored with every embedding (`PHOTO_EMBEDDING.ModelVersion`).
4. Document the outputs of the model (tensor names and shapes) in a short comment in the script or in `models/README.md`, so step 5 knows whether to use the CLS token or the pooled output.

**Done when:** running the script on a clean clone produces the model with a matching checksum; the model file is git-ignored; step ticked; committed.

---

## Step 5 — Vision core: normalization, corner detection, embeddings

**Type:** CLAUDE CODE · **Depends on:** 4 · **Docs:** architecture §2.2 (Stage 0 and Stage 1), §2.3

**Goal:** the image-processing building blocks in `SugarBags.Vision`, behind interfaces, with tests.

**Tasks**
1. Packages: `OpenCvSharp4` with the runtime package for your OS (for local development) **and** `OpenCvSharp4.official.runtime.linux-x64` (for the container); `Microsoft.ML.OnnxRuntime`.
2. `ImageNormalizer`: apply EXIF orientation, limit the long side to 1024 px, mild contrast normalization (CLAHE).
3. `CornerDetector`: document-scanner approach — edges → contours → largest 4-point polygon. Returns 4 corners plus a confidence; if no quadrilateral is found, returns the full image rectangle with low confidence.
4. `PerspectiveWarper`: warp the bag to a flat rectangle from 4 corners (from the detector **or** from the user, later).
5. `Dinov2Embedder : IImageEmbedder`: resize/center to 224×224, ImageNet mean/std, run ONNX, take the CLS token (or the model's pooled output, per step 4 notes), **L2-normalize** → 384 floats. Loads the model once (singleton, thread-safe).
6. `RotationVariants`: produce 0° and 180° variants; also 90° and 270° when the bag is roughly square (aspect ratio below ~1.2).
7. Unit tests using generated images (no personal photos in the repo): identical image → cosine ≈ 1.0; a 180°-rotated copy matches via rotation variants; a warped synthetic rectangle is detected and flattened.

**Done when:** tests pass; embedding output has 384 values with norm 1.0; step ticked; committed.

---

## Step 6 — Evaluation tool v1: retrieval accuracy

**Type:** CLAUDE CODE · **Depends on:** 3, 5 · **Docs:** requirements §7, NFR-01/02; architecture §2.2 (Stage 1)

**Goal:** `tools/SugarBags.Eval` measures how often the right bag is found using embeddings only.

**Tasks**
1. Read `testdata/` (structure from step 3). Index the gallery: normalize → auto-detect corners → warp → embeddings for all rotation variants. Keep vectors in memory (brute-force cosine search, exactly like the MVP will do in SQL).
2. For every query photo: same preprocessing, rank gallery bags by best cosine similarity (best over rotation variants).
3. Report:
   - **Recall@1** and **Recall@5** (targets: ≥ 85% and ≥ 95%);
   - for distractors: the top-1 similarity distribution (how similar do *unknown* bags look?);
   - for tricky pairs: how often the twin outranks the true bag;
   - score distributions of true matches vs. best wrong matches (needed to choose thresholds);
   - timing per query.
4. Options: `--no-crop` (to measure how much the corner detection helps), `--model <path>`.
5. Write results to `eval-results/<timestamp>/`: a `summary.md` **and** an `report.html` that shows each failed query next to the expected bag and the wrongly ranked bags (thumbnails), so you can judge failures by eye in step 8.

**Done when:** `dotnet run --project tools/SugarBags.Eval -- --data testdata` produces the summary and the HTML report; the numbers are printed in the console; step ticked; committed (results folder stays git-ignored).

---

## Step 7 — Geometric verification, score fusion, tiers + evaluation v2

**Type:** CLAUDE CODE · **Depends on:** 6 · **Docs:** architecture §2.2 (Stages 2–3), §2.4

**Goal:** the full "retrieve → verify → fuse" pipeline as reusable code, measured by the Eval tool.

**Tasks**
1. `SiftVerifier : IGeometricVerifier`: SIFT keypoints (max ~1,000 per image), k-NN matching with ratio test 0.75, homography with RANSAC, returns the **inlier count**. Keypoints and descriptors can be **serialized** to a compact binary file (the indexing pipeline stores them in Blob Storage later).
2. `ScoreFusion`: the §2.2 formula (`0.45·visual + 0.45·geometric + 0.10·text`, `geometric = min(inliers/50, 1)`), the confidence tiers (High / Medium / Low / hidden) and the §2.4 rules (name-only capped at "Similar"; geometric-confirmed visual matches rank above name-only matches with equal scores). All weights and thresholds come from a `DuplicateDetectionOptions` class (configurable, not hard-coded).
3. Put the whole pipeline (normalize → embed → retrieve top 50 → verify → fuse → top 10) in one `DuplicateDetectionPipeline` class in `SugarBags.Application`/`SugarBags.Vision` that **both** the Eval tool and the API (step 14) will use. Retrieval is behind an interface (`ICandidateRetriever`) — in-memory for Eval, SQL for the API.
4. Extend Eval to report **stage 1 only vs. stage 1+2**: Recall@1/@5, false "Very likely the same" rate on distractors and tricky pairs (target < 5%), verification time per candidate and an extrapolation for 10,000 items (NFR-03: ≤ 3 s).
5. Based on the measured score distributions, **propose** thresholds and weights in `summary.md`; put the proposed values into the default `DuplicateDetectionOptions`. If they differ from architecture §2.2, update the document (with change-log entry).

**Done when:** Eval runs both variants and the report shows the comparison; proposed thresholds are in the options class and in the document; tests for `ScoreFusion` pass (tiers, name-only cap, tie rule); step ticked; committed.

---

## Step 8 — Go/no-go review of the algorithm

**Type:** EXTERNAL · **Depends on:** 7 · **Docs:** requirements §7 · **Time:** ~1 hour

**Goal:** you decide, with evidence, whether the algorithm is good enough to build the app around.

1. Open the latest `eval-results/<timestamp>/report.html` in your browser.
2. Check the targets:
   - Recall@5 **≥ 95%**, Recall@1 **≥ 85%** (stage 1+2);
   - false "Very likely the same" on distractors and tricky pairs **< 5%**;
   - extrapolated time for 10,000 items **≤ 3 s**.
3. Look at **every failure** in the report and note the cause: bad crop, reflection, crumpled bag, back side, genuinely near-identical design, a bad photo.
4. **Decide:**
   - **Go** — all targets met (or close, with understood causes). Continue with step 9.
   - **Improve** — tell Claude Code one targeted change at a time and re-run Eval, for example: *"Try DINOv2 ViT-B/14 instead of ViT-S/14 and compare"*, *"Improve corner detection for photos taken at an angle"*, *"Raise the keypoint limit to 2,000 and compare"*.
   - **No-go** — if nothing reaches the targets, stop here and rethink before building the app (the rest of the plan depends on this).
5. Ask Claude Code: *"Record the step 8 go/no-go result with the final Eval numbers in the architecture document change log"*.

**Done when:** a written go decision with the final numbers is in the architecture document.

---

# Phase C — Backend

## Step 9 — Create the Neon database and verify extensions

**Type:** EXTERNAL · **Depends on:** — · **Docs:** architecture §8.1 · **Time:** ~20 minutes

**Goal:** the production database exists, and the three required extensions are confirmed (otherwise the fallback in §8.1 applies).

1. Go to `neon.tech` → **Sign up** (signing in with GitHub is fine). Choose the **Free** plan.
2. **Create a project:**
   - Project name: `sugarbags`
   - PostgreSQL version: the newest offered — **write the major version down** (e.g. 17); steps 10 and 22 must use the same version.
   - Region: an **EU region, e.g. Frankfurt** (close to the Azure region you will pick in step 18).
   - Database name: `sugarbags`.
3. Open the **SQL Editor** in the Neon console, select database `sugarbags`, and run:
   ```sql
   CREATE EXTENSION IF NOT EXISTS vector;
   CREATE EXTENSION IF NOT EXISTS pg_trgm;
   CREATE EXTENSION IF NOT EXISTS unaccent;
   SELECT extname, extversion FROM pg_extension ORDER BY extname;
   ```
   The last query must list `pg_trgm`, `unaccent` and `vector`.
4. **If any `CREATE EXTENSION` fails:** stop and tell Claude Code: *"Neon does not support <extension>; switch the plan to Azure Database for PostgreSQL (architecture §8.1 fallback)"*. The Azure server would then be created in step 23 instead.
5. Open **Connection details** (Dashboard → *Connect*), turn on **connection pooling**, choose the **.NET / Npgsql** format if offered, and **copy the pooled connection string into your password manager**. Do not paste it into Claude Code or commit it — you will enter it in step 23.

**Done when:** the project exists in an EU region, the three extensions are listed, the pooled connection string is stored safely, and you know the PostgreSQL major version.

---

## Step 10 — Local development environment (Docker)

**Type:** CLAUDE CODE · **Depends on:** 2, 9 (PostgreSQL version) · **Docs:** architecture §6 (indexes), this plan's "Decisions"

**Goal:** one command starts a local database and storage emulator that behave like production.

**Tasks**
1. `docker-compose.yml` with:
   - PostgreSQL using the `pgvector/pgvector` image **with the same major version as Neon** (ask me for the version if it is not in `CLAUDE.md` yet, then add it there);
   - **Azurite** (Azure Storage emulator) for blobs and queues.
2. An init script that creates the `vector`, `pg_trgm` and `unaccent` extensions in the local database.
3. `appsettings.Development.json` for the API pointing at the local services (local-only dummy credentials are fine here).
4. README section "Run locally": `docker compose up -d`, then run the API and the Web app.

**Done when:** `docker compose up -d` starts both services; `psql` (or a test) shows the three extensions; step ticked; committed.

---

## Step 11 — Domain, persistence and file storage

**Type:** CLAUDE CODE · **Depends on:** 10 · **Docs:** architecture §6 (data model and notes), §2.4, §8.1 (no HNSW), requirements FR-01/02/05/07/51

**Goal:** the data model in code and in the database, plus a storage service for photos.

**Tasks**
1. **Domain entities** per §6: `User`, `Collection`, `Item`, `ItemPhoto`, `PhotoEmbedding`, `DuplicateCheck`, `CheckCandidate`. Photos optional (0..n per item); `Side` nullable; `CopiesOwned` default 1 (not exposed in the API/UI in the MVP); soft delete via `DeletedAt`; `MatchMode` on `CheckCandidate`; `ModelVersion` on embeddings.
2. **EF Core** with Npgsql and `Pgvector.EntityFrameworkCore`; `vector(384)` column. Global query filters: soft delete **and ownership** (`ICurrentUser` — for now a development implementation that returns one fixed local user; real users come in step 21).
3. **Migrations:** create the three extensions; an **immutable wrapper function** around `unaccent` (plain `unaccent()` cannot be used in an index); GIN trigram indexes on the unaccented, lower-cased `Item.Name` and `Item.OcrText`; **no HNSW index** (architecture §8.1).
4. The API **applies pending migrations at startup** (MVP decision; max 1 replica).
5. Npgsql/EF Core **retry on transient failures** (Neon wakes from idle — §8.1).
6. **Storage service** (`IPhotoStorage`) over Azure Blob Storage: containers `originals`, `normalized`, `thumbnails`, `keypoints`, `queries`; short-lived read URLs (SAS). Two credential modes: connection string for Azurite locally, **managed identity with user-delegation SAS** in Azure.
7. **Queue client** abstraction (`IIndexQueue`) over Azure Storage Queues.
8. Integration tests with **Testcontainers** (pgvector image, same version) and Azurite: migrations apply, filters work, a vector round-trips.

**Done when:** `dotnet ef database update` works against the local database; integration tests pass; step ticked; committed.

---

## Step 12 — Catalog API

**Type:** CLAUDE CODE · **Depends on:** 11 · **Docs:** architecture §7 (API), §5.2; requirements FR-01…16, NFR-04

**Goal:** everything needed to manage and browse the collection, without the duplicate check yet.

**Tasks**
1. Minimal API endpoints (architecture §7): `GET /api/items` (filters `name`, `hasPhoto`; sort by name/date; paging), `GET /api/items/{id}`, `POST /api/items` (photo optional), `PUT /api/items/{id}`, `DELETE /api/items/{id}` (soft delete), `POST /api/items/{id}/photos` (multipart; `side` optional; optional 4 user-adjusted corners), `DELETE /api/photos/{id}`, `GET /api/stats` (item count, share of items with photos, database size via `pg_database_size`), `GET /health`.
2. Name filter: case-insensitive, accent-insensitive partial match ("cafe" finds "Café") using the immutable unaccent wrapper and trigram index.
3. Photo upload stores the **original** in Blob Storage, creates the `ItemPhoto` with status `Pending` and puts an `IndexPhoto` message on the queue (processing comes in step 13). Responses contain thumbnail URLs or `null` (UI shows a placeholder — FR-07).
4. Validation and consistent error responses (ProblemDetails); OpenAPI document with a browsable UI in Development.
5. A development **seed command** that creates 10,000 synthetic items (for performance checks).
6. Integration tests: CRUD, diacritics filter, `hasPhoto`, soft delete, ownership isolation (two dev users), stats.

**Done when:** tests pass; with 10,000 seeded items `GET /api/items?name=…` responds in well under 500 ms locally (NFR-04); step ticked; committed.

---

## Step 13 — Photo indexing pipeline

**Type:** CLAUDE CODE · **Depends on:** 7, 12 · **Docs:** architecture §5.2, §2.2 (Stage 0), §6 notes

**Goal:** every uploaded photo becomes searchable automatically.

**Tasks**
1. A `BackgroundService` running **inside the API process** (`SugarBags.Worker`) consumes `IndexPhoto` messages.
2. For each photo: load original → normalize → warp (user corners if provided, otherwise auto-detected) → store **normalized** image and **thumbnail** (~300 px) → embeddings for all rotation variants with `ModelVersion` → **SIFT keypoints** file to `keypoints` → perceptual hash → status `Indexed`.
3. Failures: retry with backoff; after N attempts status `Failed` and the message goes to a poison queue; errors are logged.
4. A development/admin command to **re-index all photos** (needed when the model version changes).
5. Tests: upload → status becomes `Indexed`; a failing image ends as `Failed` without blocking the queue.

**Done when:** uploading a photo through the API results in thumbnail, normalized image, embeddings and keypoints within seconds locally; tests pass; step ticked; committed.

---

## Step 14 — Duplicate check API

**Type:** CLAUDE CODE · **Depends on:** 13 · **Docs:** architecture §2.2, §2.4, §5.1, §7; requirements FR-20…32, NFR-03

**Goal:** the core feature — "Check if I already have it" — as an API.

**Tasks**
1. `POST /api/images/detect-corners` — returns suggested corners for the crop tool.
2. `POST /api/duplicate-checks` — multipart with **photo and/or name** (at least one, otherwise 400) and optional corners:
   - stores the query photo in `queries` (deleted after 7 days by a lifecycle rule in Azure — step 22);
   - **visual retrieval** (only when a photo is given): top 50 items by best cosine distance over all photos and rotation variants, using pgvector in SQL (`ICandidateRetriever` SQL implementation);
   - **text retrieval**: top 20 by trigram similarity on name (and OCR text, empty in the MVP) — includes items without photos;
   - geometric verification for candidates that have photos (keypoints loaded from Blob Storage, cached in memory);
   - fusion, tiers and §2.4 rules via the shared pipeline from step 7; each candidate gets a `MatchMode`;
   - persists `DuplicateCheck` + `CheckCandidate` rows; returns `checkId` and up to 10 candidates with thumbnail URLs.
3. `POST /api/duplicate-checks/{id}/decision` — "duplicate" or "new" (logged for later tuning).
4. `POST /api/items` accepts an optional `checkId`: the query photo is reused as the new item's first photo (copied to `originals`, then indexed) — no second upload.
5. `POST /api/search/by-photo` — same pipeline without creating an item (FR-15).
6. Tests for every mode in §2.4 (photo+photo, photo vs. item without photo, name only, neither → 400), the "Similar" cap, decision logging, `checkId` reuse.
7. Performance check with the 10,000-item seed (synthetic vectors): end-to-end check time ≤ 3 s locally (NFR-03).

**Done when:** tests pass; a check with a test photo from `testdata/queries` finds the right gallery item when the gallery has been uploaded through the API (provide a small dev command that imports `testdata/gallery` as items); timing within budget; step ticked; committed.

---

# Phase D — Frontend

## Step 15 — Blazor PWA shell

**Type:** CLAUDE CODE · **Depends on:** 12 · **Docs:** architecture §4.1; requirements NFR-06/07

**Goal:** a running, installable web app that talks to the API.

**Tasks**
1. `SugarBags.Web` (Blazor WebAssembly, PWA: manifest, service worker, simple **original** icon) hosting components from `SugarBags.UI` (Razor Class Library — all pages and components live there for later MAUI reuse).
2. **Fixed ports:** Web `https://localhost:7100`, API `https://localhost:7200`. API CORS allows the Web origin (configurable list).
3. Typed API client; `ICameraService` interface with a browser implementation (file input with camera capture).
4. Layout: mobile-first, navigation "Collection" and "Add item"; loading and error states.
5. `staticwebapp.config.json` with navigation fallback (for Azure Static Web Apps).
6. Calls `GET /api/stats` on the home page to prove the connection.

**Done when:** API and Web run locally; the home page shows stats; Chrome offers "Install app"; step ticked; committed.

---

## Step 16 — Collection browsing UI

**Type:** CLAUDE CODE · **Depends on:** 15 · **Docs:** requirements FR-07, FR-10…16; architecture §7

**Goal:** browse, filter and manage the collection.

**Tasks**
1. Collection page: thumbnail grid with names; **placeholder** for items without photos; infinite scroll.
2. **Live name filter** (debounced, accent-insensitive via the API); "Only items without photo" toggle; sort by name / date added.
3. Photo coverage indicator ("X % of your collection has photos") from `/api/stats`.
4. Item detail: all photos full-size with zoom (pinch on mobile); edit name/description/metadata; delete with confirmation; delete a photo. (Adding photos with the crop tool comes in step 17.)

**Done when:** all of the above works locally against seeded data, including on a narrow (phone-sized) browser window; step ticked; committed.

---

## Step 17 — Add item, photo capture and duplicate check UI

**Type:** CLAUDE CODE · **Depends on:** 14, 16 · **Docs:** requirements §6.1 flow, FR-20…32, NFR-06; architecture §2.2 Stage 0 (front-side hint), §2.4

**Goal:** the headline feature, end to end.

**Tasks**
1. Reusable **`PhotoCapture`** component: take photo / choose file; the hint **"Photograph the front side of the bag"** above the camera button; after capture, show the photo with **4 draggable corners** pre-set from `detect-corners`; touch-friendly; preview of the flattened result; "skip photo" option.
2. **Add item page** following requirements §6.1: optional photo (`PhotoCapture`) → name → **"Check if I already have it"** button, enabled when a photo **or** a name is present.
3. **Results:** up to 10 candidates with thumbnail or placeholder, name and tier badge (🟢 Very likely the same / 🟡 Similar / ⚪ Weak match); label **"Matched by name only — no photo"** for name-only candidates; clear message **"No similar item found in your collection."** when empty.
4. **Side-by-side comparison** of the new photo and a candidate with synchronized zoom/pan; for name-only candidates show the candidate's description and metadata instead.
5. **Decision buttons:** "It's a duplicate — don't add" and "It's new — add to collection" (both post the decision); "new" continues to the details form and saves with `checkId`. Users can also save without running the check.
6. Use `PhotoCapture` in item detail to **add a photo** to an existing item (side optional).

**Done when:** locally, with test photos: adding a known bag shows it as a candidate; adding an unknown bag shows "No similar item found"; a name-only check works; the flow is usable one-handed in a phone-sized window in under 30 seconds (NFR-06); step ticked; committed.

---

# Phase E — Authentication

## Step 18 — Azure account, budget alert and resource group

**Type:** EXTERNAL · **Depends on:** — · **Docs:** architecture §8 · **Time:** ~30 minutes

1. **Create an Azure account** at `azure.microsoft.com` (new accounts get free credit for the first 30 days). Sign in to `portal.azure.com`.
2. **Budget alert** (protects you from surprises): search **Cost Management** → **Budgets** → **Add** → scope: your subscription → amount **10** (USD/EUR) per month → alerts at **80%** and **100%** → your e-mail → Create.
3. **Choose a region** close to Neon's Frankfurt region, e.g. **Germany West Central** (any EU region works). Write it down.
4. **Create a resource group:** search **Resource groups** → **Create** → name `rg-sugarbags` → region from above → Review + create.
5. **Register resource providers** (needed by Container Apps). In a terminal:
   ```bash
   az login
   az account set --subscription "<your subscription name or ID>"
   az provider register --namespace Microsoft.App
   az provider register --namespace Microsoft.OperationalInsights
   ```
6. Write down: subscription ID, region, resource group name.

**Done when:** the resource group exists, the budget alert is set, providers are registering/registered.

---

## Step 19 — Entra External ID tenant, app registrations, user flow

**Type:** EXTERNAL · **Depends on:** 18 · **Docs:** architecture §3 · **Time:** ~45 minutes

> Two different tenants exist after this step: your **default Azure directory** (owns the subscription and resources) and the new **external tenant** (holds the app's users). Always check in the top-right corner of the portal which one you are in.

1. Open `entra.microsoft.com` → **Manage tenants** (or *Overview → Manage tenants*) → **Create** → choose **External** → Continue.
2. Fill in: tenant name `SugarBags`, domain name (subdomain) e.g. `sugarbags` (if taken, choose another — this becomes `<subdomain>.onmicrosoft.com` and `<subdomain>.ciamlogin.com`), location **Europe**; link your subscription and resource group `rg-sugarbags`. Create (takes a few minutes).
3. **Switch to the new tenant** (Settings icon → *Directories + subscriptions* → switch). On its **Overview** page, write down the **Tenant ID** and the **subdomain**.
4. **Register the API app:** *App registrations* → **New registration** → name `SugarBags API` → *Accounts in this organizational directory only* → Register. Write down its **Application (client) ID**.
   - **Expose an API** → *Add* next to Application ID URI (accept `api://<client-id>`) → **Add a scope**: name `access_as_user`, who can consent: *Admins only*, admin consent display name "Access SugarBags API" → Add scope.
5. **Register the web app:** **New registration** → name `SugarBags Web` → *Accounts in this organizational directory only* → Redirect URI: platform **Single-page application (SPA)**, URI `https://localhost:7100/authentication/login-callback` → Register. Write down its **Application (client) ID**.
   - **API permissions** → *Add a permission* → *APIs my organization uses* (or *My APIs*) → `SugarBags API` → Delegated → `access_as_user` → Add. Also ensure Microsoft Graph delegated `openid` and `offline_access` are present.
   - Click **Grant admin consent for SugarBags** → Yes (in external tenants users cannot consent themselves).
6. **Create the user flow:** *External Identities → User flows* → **New user flow** → name `signup_signin` → identity providers: for now keep **Email with password** (Google is added in step 20; you can remove email later if the portal allows) → user attributes: *Display Name* → Create.
7. **Connect the web app to the user flow:** open user flow `signup_signin` → **Applications** → *Add application* → select `SugarBags Web` → Select.

**Done when:** you have written down tenant ID, subdomain, API client ID, scope `api://<api-client-id>/access_as_user`, and web client ID; the user flow contains the web app.

---

## Step 20 — Google sign-in: OAuth client and federation

**Type:** EXTERNAL · **Depends on:** 19 · **Time:** ~30 minutes
**Reference:** Microsoft's guide *"Add Google as an identity provider"* for External ID (learn.microsoft.com → `entra/external-id/customers/how-to-google-federation-customers`) — use it if any screen differs from the steps below.

**Part A — Google Cloud Console** (`console.cloud.google.com`)
1. Create a new project, e.g. `SugarBags` (project picker at the top → *New project*).
2. Open **APIs & Services → OAuth consent screen** (in the newer console this is **Google Auth Platform** with pages *Branding*, *Audience*, *Clients*):
   - App name: `SugarBags` (Microsoft's guide suggests a name such as "Microsoft Entra External ID" — either is fine), user support e-mail: yours.
   - **Authorized domains:** add `ciamlogin.com` and `microsoftonline.com`.
   - **Audience / user type: External.** Leave publishing status **Testing** and add **your own Gmail address as a test user** — enough for the single-user MVP. (Before inviting other people, publish the app.)
   - Scopes: only the basic `openid`, `email`, `profile`.
3. **Create the OAuth client:** *Credentials* (or *Clients*) → **Create credentials → OAuth client ID** → type **Web application** → name `Entra External ID`. Add these **Authorized redirect URIs**, replacing `<tenant-ID>` and `<tenant-subdomain>` with your values from step 19:
   ```
   https://login.microsoftonline.com/te/<tenant-ID>/oauth2/authresp
   https://login.microsoftonline.com/te/<tenant-subdomain>.onmicrosoft.com/oauth2/authresp
   https://<tenant-ID>.ciamlogin.com/<tenant-ID>/federation/oidc/accounts.google.com
   https://<tenant-ID>.ciamlogin.com/<tenant-subdomain>.onmicrosoft.com/federation/oidc/accounts.google.com
   https://<tenant-subdomain>.ciamlogin.com/<tenant-ID>/federation/oauth2
   https://<tenant-subdomain>.ciamlogin.com/<tenant-subdomain>.onmicrosoft.com/federation/oauth2
   ```
   → Create. Copy the **Client ID** and **Client secret** into your password manager.

**Part B — Entra admin center** (in the **external** tenant)
4. *External Identities → All identity providers* → **Google** → paste the Client ID and Client secret → Save.
5. *External Identities → User flows* → `signup_signin` → **Identity providers** → tick **Google** → Save.

**Done when:** Google appears in the user flow's identity providers. (You test the actual sign-in in step 21.)

---

## Step 21 — Integrate authentication in API and UI

**Type:** CLAUDE CODE · **Depends on:** 17, 20 · **Docs:** this plan's "Values you will collect"; requirements FR-50/51; architecture §6

**Goal:** sign in with Google; every request runs as the signed-in user; users see only their own data.

**Before starting, give Claude Code the non-secret values from step 19** (tenant ID, subdomain, API client ID, web client ID, scope). No secrets are needed for this step.

**Tasks**
1. **API:** `Microsoft.Identity.Web` JWT bearer validation for the external tenant (`https://<subdomain>.ciamlogin.com/...` authority, audience = API client ID, required scope `access_as_user`). All `/api/*` endpoints require authentication; `/health` stays anonymous.
2. **`ICurrentUser`** from the token's `oid` claim (never the e-mail). **First sign-in provisioning:** create the `User` row (`ExternalId = oid`) and a default `Collection` ("Sugar bags", `ItemType = "SugarBag"`). The development fixed user from step 11 remains available only behind an explicit development switch.
3. **Web:** MSAL for Blazor WebAssembly (`Microsoft.Authentication.WebAssembly.Msal`) with the external-tenant authority, web client ID and default scope; sign-in/sign-out buttons; protected pages; access token attached to API calls. Send `domain_hint` for Google so users go straight to Google while it is the only provider.
4. Configuration in `appsettings.json` / `appsettings.Production.json` for both apps (non-secret values only).
5. Integration tests with test tokens: unauthenticated → 401; user A cannot read, change or check against user B's items (ownership filter).

**Done when:** locally you can sign in with your Google account at `https://localhost:7100`, your user and default collection are created, existing functionality works signed-in; tests pass; step ticked; committed.

---

# Phase F — Deployment

## Step 22 — Infrastructure as code (Bicep) and backup job

**Type:** CLAUDE CODE · **Depends on:** 21 · **Docs:** architecture §8, §8.1, §8.2; this plan's "Decisions"

**Goal:** all Azure resources described in code, deployable with one command; documents updated with this plan's decisions.

**Tasks**
1. `infra/main.bicep` (+ `main.bicepparam`) creating in `rg-sugarbags`:
   - **Storage account**: blob containers (`originals`, `normalized`, `thumbnails`, `keypoints`, `queries`), queue for indexing, **lifecycle rules** (delete `queries` blobs after 7 days; move `originals` to Cool), **Azure Files share** `backups`;
   - **Log Analytics workspace** and **Application Insights**;
   - **Container Apps environment** (Consumption) with the `backups` share registered as storage;
   - **Container App** `sugarbags-api`: external HTTPS ingress, **min replicas 0, max replicas 1**, system-assigned **managed identity** with the role assignments needed for blobs, queues and user-delegation SAS; environment variables for storage endpoints, External ID settings and **CORS origin = the Static Web App hostname**; a secret slot for the database connection string (value set by you in step 23); a public placeholder image until CI deploys the real one;
   - **Container Apps Job** `sugarbags-backup`: scheduled nightly (e.g. 02:00), image `postgres:<Neon major version>`, mounts the `backups` share, runs `pg_dump -Fc` to a dated file and deletes dumps older than 30 days; connection string from a secret;
   - **Static Web App** (Free plan).
   - Outputs: API URL, Static Web App URL and name, Container App name.
2. `infra/README.md` with the exact deployment commands for step 23 (what-if, then create) and the commands to set the two secrets.
3. Check that the template compiles (`az bicep build`).
4. **Update the documents**: add this plan's "Decisions" to architecture (§8, §8.1 backup target = Azure Files share, Container Apps secrets instead of Key Vault, max 1 replica with startup migrations, local Docker development) with a change-log entry.

**Done when:** `az bicep build` succeeds; README has copy-paste commands; architecture document updated; step ticked; committed.

---

## Step 23 — Deploy the Azure infrastructure

**Type:** EXTERNAL · **Depends on:** 22, 9 · **Time:** ~30 minutes

1. In a terminal in the repository root:
   ```bash
   az login
   az account set --subscription "<subscription ID>"
   ```
2. Run the **what-if** command from `infra/README.md` and read the list of resources it will create.
3. Run the **create** command from `infra/README.md`. When it finishes, copy the outputs: **API URL**, **Static Web App URL**, names.
4. **Set the secrets** using the commands from `infra/README.md` — paste the **Neon pooled connection string** (step 9) into your terminal yourself, for both the Container App and the backup job. (Tip: to keep it out of your shell history, follow the README's suggestion for reading it from a prompt.)
5. **Add the production redirect URI** in Entra (external tenant) → *App registrations* → `SugarBags Web` → *Authentication* → SPA redirect URIs → add `https://<static-web-app-hostname>/authentication/login-callback` → Save.
6. In the Azure portal, open `rg-sugarbags` and check the resources exist. The Container App still runs the placeholder image — that is expected.
7. Give Claude Code the non-secret outputs (API URL, Static Web App URL, resource names) so it can use them in step 24.

**Done when:** all resources exist; both secrets are set; the production redirect URI is registered.

---

## Step 24 — Dockerfile and GitHub Actions (CI/CD)

**Type:** CLAUDE CODE · **Depends on:** 22, 23 (outputs) · **Docs:** architecture §8.2

**Goal:** every push to `main` is built, tested and deployed automatically; the image is stored on ghcr.io as a **public** package (the repository is public).

**Tasks**
1. **Dockerfile** for `SugarBags.Api` (multi-stage, .NET 10, Linux x64): includes the OpenCV native runtime and its system dependencies, and the ONNX model (downloaded with `tools/get-model.sh` and checksum-verified during the build). Non-root user. `/health` endpoint used as the health probe. Because the image will be **public**: add a `.dockerignore` (no `.env`, `testdata/`, `eval-results/`, `*.dump`, user-secrets) and make sure no secret is baked in; secrets reach the app only as Container Apps secrets at runtime.
2. **`ci.yml`** (pull requests and pushes): restore, build, run all tests (Testcontainers works on GitHub's Ubuntu runners).
3. **`deploy-api.yml`** (push to `main`): build the image, push to `ghcr.io/<owner>/sugarbags-api` tagged with the commit SHA (login with the built-in `GITHUB_TOKEN`, `packages: write` permission), then log in to Azure with **OpenID Connect** (`azure/login`, no stored password) and update the Container App to the new image.
4. **`deploy-web.yml`** (push to `main`): publish the Blazor app and deploy it to the Static Web App with the deployment token secret; production configuration (API URL, External ID values) from `appsettings.Production.json`.
5. Document in `README.md` the required GitHub **secrets** (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`, `AZURE_STATIC_WEB_APPS_API_TOKEN`) and **variables** (resource group, Container App name).
6. Verify locally: `docker build` succeeds and the container answers `/health`.

**Done when:** the image builds and runs locally; the workflows are valid YAML and reference only documented secrets; step ticked; committed.

---

## Step 25 — Connect GitHub to Azure

**Type:** EXTERNAL · **Depends on:** 24 · **Time:** ~30 minutes

> This identity lives in your **default Azure directory** (where the subscription is) — **not** in the External ID tenant.

1. In `portal.azure.com` (default directory) → **Microsoft Entra ID → App registrations → New registration** → name `github-sugarbags-deploy` → Register. Write down its **Application (client) ID** and the **Directory (tenant) ID**.
2. In that app → **Certificates & secrets → Federated credentials → Add credential** → scenario **GitHub Actions deploying Azure resources** → Organization `jborowiak`, Repository `Collection-Organizer`, Entity type **Branch**, Branch `main`, name `main` → Add.
3. **Give it access to the resource group:** **Resource groups → rg-sugarbags → Access control (IAM) → Add → Add role assignment** → role **Contributor** → Members: *User, group, or service principal* → select `github-sugarbags-deploy` → Review + assign.
4. **Static Web Apps token:** open the Static Web App → **Overview → Manage deployment token** → copy it.
5. **GitHub repository → Settings → Secrets and variables → Actions:**
   - **Secrets:** `AZURE_CLIENT_ID` (from 1), `AZURE_TENANT_ID` (default directory ID from 1), `AZURE_SUBSCRIPTION_ID`, `AZURE_STATIC_WEB_APPS_API_TOKEN` (from 4).
   - **Variables:** the names listed in `README.md` (resource group, Container App name).

**Done when:** the federated credential, the role assignment and all secrets/variables exist.

---

## Step 26 — First release and acceptance test

**Type:** EXTERNAL · **Depends on:** 25 · **Docs:** requirements §3.1, §7 · **Time:** ~1–2 hours

1. **Push to `main`** (`git push`) and watch **GitHub → Actions**. Both deploy workflows should run.
2. **Make the image public** (the repository is public, so a public image is free and needs no credentials). After the first run, the package is created **private** by default: GitHub → your profile → **Packages** → `sugarbags-api` → **Package settings** → *Danger Zone* → **Change visibility** → **Public** → confirm. Then re-run the failed `deploy-api` workflow (Actions → the run → *Re-run jobs*). You only do this once.
   - *If you ever make the repository private:* tell Claude Code *"The repository is now private — apply the private-image option from architecture §8.2"*.
3. **Open the Static Web App URL** in a desktop browser → **Sign in with Google** → your collection is empty.
4. **Smoke test** (each must work):
   - add an item **without** a photo; add an item **with** a photo; filter by name with and without accents; open details and zoom; delete an item;
   - run **"Check if I already have it"** with a photo of a bag you just added → it is found; with an unknown bag → "No similar item found"; with only a name → name-only candidates are labelled.
5. **Android:** open the URL in **Chrome** on your phone → sign in → menu (⋮) → **Install app** / *Add to Home screen*. From the home-screen icon: take a photo with the camera, adjust corners, run a check. Note anything awkward for later.
6. **Backup test:** Azure portal → **Container Apps Jobs → sugarbags-backup → Run now**. Then **Storage account → File shares → backups** — a new `.dump` file appears. Ask Claude Code: *"Restore the latest backup dump into the local Docker database and verify the item count"* (download the file first and tell Claude Code where it is).
7. Optional: tag the release:
   ```bash
   git tag v0.1.0
   git push --tags
   ```

**Done when:** all smoke tests pass on desktop and Android, a backup file exists and was restored once successfully. **The MVP is live** — start adding your collection.

---

## After the MVP

Not part of this plan (see requirements §8 and architecture §11): Phase 1.1 (threshold tuning from logged decisions, OCR, front/back matching improvements), Phase 2 (native Android app, "copies owned"), Phase 3 (iOS). Before starting any of them, ask Claude Code to extend this plan with new numbered steps in the same format.
