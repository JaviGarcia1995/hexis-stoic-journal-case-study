# Technical Decisions

## Overview

This project was designed to be practical rather than over-engineered. The main goal was to build a maintainable Kotlin Multiplatform codebase that could share business logic and UI while keeping the architecture clear enough to evolve over time.

The decisions below explain why the project is structured the way it is and what tradeoffs were intentionally accepted.

## Package-Based Clean Architecture

Instead of splitting the project into many small Gradle modules, the app keeps Clean Architecture inside the shared codebase through package organization.

This choice was made to:
- reduce build complexity
- keep the project easier to navigate
- preserve clear architectural boundaries
- avoid fragmenting a relatively small team or codebase into unnecessary modules

The architecture still follows the same dependency direction as traditional Clean Architecture. The difference is that separation is enforced by package conventions, project structure, and validation scripts rather than by many physical modules.

## Compose Multiplatform for Shared UI

Compose Multiplatform is used to share most of the app UI across platforms.

This was a deliberate choice because the app is centered around repeated journaling flows, similar layouts, and reusable components. Sharing the UI reduces duplication and keeps the experience visually consistent.

The tradeoff is that some platform-specific concerns still need to be handled carefully, but the benefit is a single UI strategy for the shared codebase.

## Presentation Owns Orchestration

The presentation layer is responsible for state management, validation, orchestration, and UI-facing models.

This keeps the UI layer focused on rendering and user interaction callbacks. It also prevents business rules from leaking into Compose screens and components.

The result is a cleaner separation between:
- what the user sees
- what the app decides
- what the domain layer defines

This is especially important in the journaling flows, where validation and step progression need to stay consistent.

## SQLDelight for Local Persistence

SQLDelight is used as the production persistence layer.

The app is designed around local storage and offline-first behavior, so having a typed SQL-backed persistence layer makes sense for long-term maintainability. It also keeps the repository implementations explicit and allows the domain layer to stay independent from storage details.

The data layer owns:
- SQLDelight-backed repositories
- local datasources
- mappers between persistence and domain models
- journal-specific SQL operations

This prevents persistence details from leaking into the domain layer.

## Typed Navigation

Navigation is modeled with serializable destinations instead of string-based routes.

This was chosen to reduce fragile navigation code and make routes explicit. Type-safe navigation is a better fit for a shared Compose codebase, especially when multiple flows and overlays need to be coordinated.

The navigation layer is also separated from screen rendering so route orchestration, overlays, and transitions stay centralized.

## Multiplatform Abstractions for Time and Dispatchers

Two platform-dependent concerns are abstracted through dedicated providers:

- `TimeProvider`
- `DispatcherProvider`

These abstractions exist so common code does not depend directly on the system clock or on platform-specific coroutine dispatchers.

This improves:
- portability across targets
- testability
- consistency in shared code

The alternative would be to couple common code to platform runtime details, which would weaken the multiplatform design.

## Koin for Dependency Wiring

Koin is used to connect repositories, use cases, ViewModels, and shared infrastructure.

This was a pragmatic choice because it keeps dependency wiring readable and avoids manual object graph construction. In a shared KMP project, that simplicity is valuable.

The DI setup is split by responsibility:
- domain use cases
- data implementations
- presentation ViewModels
- shared infrastructure

This keeps composition explicit without adding unnecessary complexity.

## Single-Responsibility Use Cases

Use cases are intentionally small and focused on one operation.

This keeps the domain layer easy to understand and makes the business logic more reusable across presentation flows.

The benefits are:
- clear naming
- easier testing
- easier orchestration from ViewModels
- less accidental coupling between unrelated operations

## Summary

The architecture and tooling choices in this project are intentionally conservative. The goal was not to maximize abstraction, but to build a stable shared codebase with:
- clean layer boundaries
- shared UI where it adds value
- explicit dependency wiring
- offline-first persistence
- maintainable multiplatform abstractions

That balance makes the project easier to extend without losing clarity.
