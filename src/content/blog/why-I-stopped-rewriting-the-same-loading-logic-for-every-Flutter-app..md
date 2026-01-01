---
title: "Why I stopped rewriting the same loading logic for every Flutter app."
description: "Learn how to standardize UI states in Flutter to reduce boilerplate and improve consistency using the loading_state_handler package."
pubDate: 2026-01-01
heroImage: "https://storage.googleapis.com/48877118-7272-4a4d-b302-0465d8aa4548/9c56255d-dc54-4f5f-a2b8-204878d59dea/c1eda2c6-15ef-473b-b4bb-6b6f56f82bfb.jpg"
tags: ["flutter", "dart", "loading-states", "clean-code", "mobile-development"]
---

As a **Flutter** developer, I kept running into the same friction: UI states like loading, error, and success create repetitive boilerplate that quickly clutters widget trees and increases the chance of bugs. I was rewriting the same FutureBuilder/StreamBuilder patterns across screens and projects — wasting time and introducing inconsistent UX across apps.

**Standardizing UI states** is essential for clean code and predictable behavior. That’s why I built [loading\_state\_handler](https://pub.dev/packages/loading_state_handler) — a small package that encapsulates common **loading states**, **error handling**, retry strategies, and rendering hooks so you don’t have to copy-paste the same logic across widgets or files.

Below I’ll show a concise migration path and examples so you can replace ad-hoc FutureBuilder patterns with a reusable approach that plays nicely with provider and other state management methods.

### Key Takeaways

*   Simplify **loading logic** in **Flutter** apps with a reusable solution.
*   Reduce code duplication and unnecessary rebuilds by standardizing UI states across your app.
*   Improve development efficiency: fewer repeated patterns, clearer files, and easier refactors.
*   Enhance the user experience by consistently handling loading, success, and error screens with meaningful messages and retry options.
*   Maintain cleaner code and faster iteration cycles by using a small package rather than re-implementing the same patterns in every project.

## The Repetitive Nature of Loading States in Flutter

In **Flutter** development, handling common **loading states**—loading, success, and error—shows up everywhere you fetch data or perform async work. I found myself re-implementing nearly identical patterns across screens: show a spinner, show content, show an error with a retry button. That repetition adds boilerplate to widget trees and slows development time across every app and project.

Developers routinely copy similar **code** to manage these states in different widgets and files. Over a few projects this repeats dozens of times and raises the chance of subtle inconsistencies and bugs: different error messages, mismatched retry behavior, or inconsistent layouts.

### Common Loading Scenarios in API Calls and Data Fetching

When making **API calls** or streaming real-time updates, apps move through predictable states. In Flutter you’ll often reach for widgets like FutureBuilder and StreamBuilder to react to asynchronous sources; they work well, but the surrounding boilerplate (connectionState checks, null-safety guards, loading indicators) quickly multiplies across the app.

For example, a typical Future-based user profile fetch requires checking the connection state, rendering a loader while waiting, handling data when complete, and showing an error UI when something goes wrong. A Stream-based chat feed has similar concerns but repeated in other places. That duplication is not just tedious — it makes refactors painful and increases the surface area for regressions.

The following table summarizes the usual scenarios and how developers commonly handle them in Flutter:

<table style="border: 1px solid #000;"><tbody><tr><td>Loading ScenarioHandling MechanismExample Use Case</td></tr><tr><td>API Call</td><td>FutureBuilder</td><td>Fetching user profile from a backend server</td></tr><tr><td>Data Fetching</td><td>StreamBuilder</td><td>Live chat messages or real-time updates from a database</td></tr><tr><td>Local Data Processing</td><td>setState or BLoC</td><td>Processing images, heavy parsing, or local transforms</td></tr></tbody></table>

### The Technical Debt of Copy-Pasted Loading Code

Copy-pasting loading **logic** seems fast in the moment, but it creates technical debt. Update a behavior (change the spinner, tweak retry limits, adjust wording) and you must edit the same pattern across multiple widgets and files. That increases maintenance cost and the odds that one place stays out-of-date.

**When you duplicate loading code, a single change requires edits in many places**, which is error-prone and time-consuming. It also leads to inconsistent UX — one screen shows a full-screen loader, another shows an inline skeleton — and inconsistent handling of network issues or retries.

Understanding this repetitive pattern is the first step to a more scalable approach: centralize the loading behavior, keep the widget tree lean, and reduce redundant rebuilds and duplicated code across your apps. See the example and migration suggestions in the next section to learn how a reusable abstraction (like loading\_state\_handler) can replace repeated FutureBuilder/StreamBuilder snippets and integrate with provider or other state management tools.

## Why I Stopped Rewriting the Same Loading Logic for Every Flutter App

After shipping a string of Flutter projects, I reached the same conclusion: the repeated work around loading states needed to be standardized. Re-implementing the same loading/success/error UI across screens and apps was tedious, introduced subtle inconsistencies, and slowed down development — so I decided to stop rewriting that logic and build a small, reusable solution instead.

### Breaking the FutureBuilder and StreamBuilder Cycle

Flutter's **FutureBuilder** and **StreamBuilder** are great tools for reacting to asynchronous data, but leaning on them everywhere tightly couples UI code to the async source and often spreads similar boilerplate across many widgets. The typical pattern — check connection state, show a spinner while waiting, render data on success, show an error with retry on failure — repeats in dozens of places in medium-sized **apps**, and each repetition is another place to maintain.

Using **setState** for these flows works for very small widgets, but it quickly becomes a maintenance headache as the number of async operations grows. Mixing business logic and UI updates in the same widget leads to deeper widget trees, duplicated code across files, and more points where unnecessary rebuilds and lifecycle issues can creep in.

### The Limitations of setState and BLoC for Loading States

**BLoC (Business Logic Component)** helps separate concerns and is a solid choice when your app requires complex coordination. Still, for many screens the overhead of creating and wiring a BLoC — streams, events, states, tests — can feel disproportionate. In small-to-medium projects you often want something lighter that standardizes loading behavior without forcing a full architectural shift.

So I built loading\_state\_handler as a pragmatic compromise: a tiny package that captures common loading patterns (loading, success, error), integrates with provider and other state management tools, and provides a simple widget API to render consistent UI across the **app**. It reduces duplicated **code**, keeps widget trees cleaner, and makes updates across multiple screens straightforward.

> "Find the balance between simplicity and the right level of structure for your project."

Community best practice

In short: stop re-implementing the same FutureBuilder/StreamBuilder patterns in every widget. Centralize the loading behavior, keep UI concerns focused on rendering, and use a small, reusable abstraction (like loading\_state\_handler) when you need consistent loading semantics across **apps**. The next section walks through a concrete before/after example and migration steps, including when you should still reach for BLoC or provider instead.

## Creating a Reusable Loading Pattern with Flutter

When building **Flutter** **apps**, a consistent, reusable loading pattern reduces complexity and keeps widget trees lean. Instead of scattering similar loading/success/error handling across many **files** and widgets, centralize the behavior so individual **widgets** focus on rendering UI while a small abstraction manages the async lifecycle and retries.

### Building a Generic LoadingState Class

Start with a minimal, copy-paste-ready Dart model that represents the three primary states an async operation can be in. This generic **class** works with different data types and plays well with provider or other state management tools.

```dart 
enum LoadingStatus { loading, success, error }

class LoadingState<T> {

final T? data;

final LoadingStatus status;

final String? errorMessage;

LoadingState({this.data, required this.status, this.errorMessage});

}
```

This small model lets you represent a loading flow in a single place instead of repeating similar logic across many **widgets**. Use a provider-backed ChangeNotifier or a lightweight wrapper to expose LoadingState<T> to the UI layer.

### Implementing Error Handling and Automatic Retry Logic

**Error handling** should be deliberate: identify likely failure modes (network timeout, parsing error, empty response) and provide actionable messages. Automatic retry can reduce transient failures, but keep UX and backend load in mind: use a backoff strategy and a reasonable retry cap.

*   Identify error sources and map them to friendly messages the user can act on.
*   Offer a retry button in the error UI and consider an automatic retry with exponential backoff for transient errors.
*   Log failures for diagnostics and surface clear error text for accessibility and localization.

Implementation tips to avoid common pitfalls:

*   Integrate LoadingState with provider: expose LoadingState<T> from a ChangeNotifier so widgets can read state without duplicating async logic.
*   Keep small render-only widgets (const where possible) to minimize unnecessary rebuilds and respect the widget lifecycle.
*   Throttle retries and add a maxAttempts limit to avoid hammering APIs — a common method is exponential backoff: wait 1s, 2s, 4s, etc., up to a cap.

### How loading\_state\_handler Wraps This

loading\_state\_handler (the small package I built) wraps these concepts into a compact API:

*   fromFuture / fromStream helpers to create LoadingState from async sources.
*   Built-in retry options (max attempts, backoff), so you don't copy retry code across **projects**.
*   A widget builder callback that receives LoadingState<T> and renders loading, error (with retry), or success UI, keeping your build methods concise.

Example usage pattern (conceptual): use loading\_state\_handler to convert a Future into a LoadingState, provide it via provider, and use a small LoadingStateWidget to render the spinner, error screen, or the actual content. This replaces repeated FutureBuilder boilerplate with a clear, testable pattern.

### Accessibility, Performance, and Migration

Accessibility: always surface clear, localized error messages and ensure retry buttons are reachable and announced by screen readers.

Performance: avoid unnecessary rebuilds by splitting UI into small, const-renderable widgets and only listening to LoadingState where needed. This reduces rebuilds and improves perceived load times.

Migration checklist (quick):

*   Identify duplicate FutureBuilder/StreamBuilder patterns in your app.
*   Create a LoadingState<T> provider backed by loading\_state\_handler.fromFuture or your own wrapper.
*   Replace in-widget async handling with a small LoadingStateWidget that consumes the provider.
*   Run tests and tweak retry/backoff parameters where appropriate.

Following this approach keeps the widget tree focused on rendering, moves business logic out of build methods, reduces duplicated **code**, and makes updates across screens straightforward.

## Conclusion: The Time-Saving Benefits of Standardized Loading Logic

Adopting a standardized approach to loading logic in **Flutter** **apps** cuts down on repetitive work and technical debt. A small, generic Dart **class** (like LoadingState<T>) or a compact package such as [_loading\_state\_handler_](https://pub.dev/packages/loading_state_handler) centralizes loading, success, and error handling so individual **widgets** can focus on rendering. That simplicity reduces duplicated **code** across files, minimizes unnecessary rebuilds, and shortens development cycles.

Practically speaking, standardizing loading behavior means fewer places to update when you change UX (spinner style, retry limits, wording), and it makes the app more consistent across screens and features. Teams spend less time hunting down divergent implementations and more time building product features.

Here’s a short, practical migration checklist to adopt this approach in your project:

*   Scan your codebase for repeated FutureBuilder / StreamBuilder patterns and inline setState loading flows.
*   Introduce a LoadingState<T> model or install loading\_state\_handler to wrap Future/Stream sources with retry and error metadata.
*   Expose LoadingState via provider (or your state management tool of choice) so UI widgets can read the state without embedding async logic.
*   Replace per-widget FutureBuilder boilerplate with a small LoadingStateWidget or builder callback that renders loading, error (with retry), or the content.
*   Run a quick visual pass and tests; tune retry/backoff and error messages for accessibility and UX.

Compatibility note: this pattern works with provider, Riverpod, or even alongside BLoC — use the approach that fits your app’s scale. For small-to-medium projects, a lightweight LoadingState-based approach is often faster to implement than a full BLoC refactor, while still providing consistent behavior and easier maintenance.

Want to try it now? Install the [loading_state_handler](https://pub.dev/packages/loading_state_handler) package, wrap a Future with the helper, and swap one FutureBuilder with a LoadingStateWidget — you’ll immediately see cleaner build methods and fewer duplicated snippets. If you have metrics from your projects (hours saved, reduced duplicated code), add them to quantify the benefits — anecdotal improvements are great, but measured outcomes help teams make the case for adopting the pattern.

By embracing a reusable loading pattern, you streamline development, reduce errors, and deliver higher-quality apps faster. It’s a small upfront investment that pays off across the lifetime of a project and multiple apps.

## FAQ

### What is the main issue with loading logic in Flutter apps?

The core issue is repetition: loading, success, and error handling patterns are implemented repeatedly across widgets and files. That duplication creates technical debt, inconsistent UX, and extra maintenance work when you need to change spinner styles, retry behavior, or error text.

### How can FutureBuilder and StreamBuilder be limiting when handling loading states?

**FutureBuilder** and **StreamBuilder** are appropriate tools for many async cases, but they often lead to repeated boilerplate (connectionState checks, null guards, inline error handling). Overuse couples UI to async logic and can scatter similar implementations across the app — making consistent updates harder. Use them when local, ad-hoc handling suffices; consider a shared abstraction for app-wide consistency.

### What are the benefits of creating a reusable loading pattern in Flutter?

A reusable loading pattern reduces duplicated **code**, lowers the chance of inconsistent behavior, and centralizes retry/backoff strategies and error messaging. It keeps individual **widgets** focused on rendering while a single component handles async lifecycle, improving maintainability and speeding up future changes across multiple screens and apps.

### How can a generic LoadingState class be used to simplify loading logic?

A generic LoadingState model (loading, success, error) centralizes state shape and metadata (data, error message). Expose LoadingState<T> via provider or another management tool and use a small builder widget to render spinner, error UI with retry, or success content — replacing repeated FutureBuilder snippets with a single, testable pattern.

### What is loading\_state\_handler and when should I use it?

loading\_state\_handler is a compact package that wraps common loading patterns (fromFuture/fromStream), exposes retry/backoff options, and provides a builder-style widget to render consistent loading, error, and success UIs. Use it when you want consistent behavior across screens without re-implementing boilerplate — particularly useful in small-to-medium projects or when migrating many FutureBuilder/StreamBuilder instances.

### How do I migrate from FutureBuilder to loading\_state\_handler?

Migration is straightforward: wrap your Future with loading\_state\_handler.fromFuture (or create a LoadingState provider), expose the LoadingState<T> to the UI via provider, and replace the inline FutureBuilder with a LoadingStateWidget that renders loading, error (with retry), or the content. This removes repeated connectionState logic from build methods.

### Does loading\_state\_handler work with provider, Riverpod, or BLoC?

Yes. The package is designed to be state-management-agnostic: you can expose LoadingState via provider/ChangeNotifier, Riverpod providers, or adapt it inside a BLoC. It’s intended to complement existing management methods rather than replace them entirely.

### Where can I report issues or contribute?

Report issues, request features, or contribute to the [repository](https://pub.dev/packages/loading_state_handler). If you try the package and run into integration questions (provider, Riverpod, BLoC), file an issue or open a PR — community feedback helps improve retry defaults, APIs, and examples.

### Can this approach be applied to other frameworks?

The pattern — centralizing loading semantics and rendering via a small model and builder — is conceptually transferable to other frameworks, but the specific implementation details here target Flutter and Dart. Adapt the idea to fit the idioms and tools of other platforms if needed.