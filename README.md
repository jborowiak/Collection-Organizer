# Collection-Organizer

Sugar Bag Collection Organizer — catalogue a sugar-bag collection with photos and check whether a bag is already in the collection before adding it, using photo-based duplicate detection.

See [`docs/01-requirements.md`](docs/01-requirements.md), [`docs/02-architecture.md`](docs/02-architecture.md) and [`docs/03-mvp-plan.md`](docs/03-mvp-plan.md) for requirements, architecture and the implementation plan.

## How to build

```bash
dotnet build
dotnet test
```

Requires the .NET 10 SDK. Running the API and Web app locally, and the Docker-based local environment, are documented in `CLAUDE.md` and will be extended as the plan's setup steps (10+) land.
