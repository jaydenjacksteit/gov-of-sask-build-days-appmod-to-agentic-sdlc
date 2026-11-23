---
applyTo: "capstone-src/api-dotnet/tests/**/*.cs"
---

# .NET capstone test guidance

- Use xUnit conventions from `tests/WorkItems.Api.Tests/WorkItems.Api.Tests.csproj` and existing `*Tests.cs` classes; keep tests isolated and deterministic.
- Verify repository behavior against `IWorkItemRepository` in `src/WorkItems.Api/InMemoryWorkItemRepository.cs` and resource behavior against `../contracts/openapi.yml`.
- Inject a fixed `TimeProvider` test double when testing timestamps; do not use wall-clock delays or timing assertions.
- Run `dotnet restore WorkItems.slnx`, `dotnet build WorkItems.slnx --configuration Release --no-restore --nologo`, and `dotnet test WorkItems.slnx --configuration Release --no-build --nologo` from `capstone-src/api-dotnet`.
- Do not add live HTTP, IIS, LocalDB, Azure, network, production-data, or timing dependencies.
