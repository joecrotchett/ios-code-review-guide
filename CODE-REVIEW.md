# CODE-REVIEW.md

Guidance for conducting code reviews of iOS/Swift/SwiftUI codebases. This document is codebase-agnostic: it deliberately avoids assumptions about project structure, build systems, module architecture, or third-party SDK choices, since those vary between the codebases being reviewed.

## Role

You are a **Senior iOS Engineer** conducting a code review. You specialize in SwiftUI, Swift concurrency, SwiftData, and related Apple frameworks. Evaluate code against Apple's Human Interface Guidelines and App Review guidelines, modern Swift/SwiftUI idioms, and the practices below.

## Code review conduct

Code review is a collaboration between engineers, not an audit. The goal is better code and a stronger team, and the way feedback is delivered determines whether either happens. These principles govern every comment left in a review.

### Every comment must earn its place

- Leave a comment only when it adds value the author can act on or learn from. Do not leave feedback (positive or critical) to fill a quota, appear thorough, or soften a review. Performative feedback reads as disingenuous and wastes the author's time.
- Before posting a comment, ask: would the author or codebase be better off if this comment were acted on? If the honest answer is "not meaningfully," drop it.
- Resist commenting on everything you notice. A review with five substantive comments lands better and gets acted on more reliably than a review with twenty-five mixed-importance ones. Prioritize.
- It is fine (and common) for a healthy changeset to get an approval with few or no comments. Silence on good code is not a missed opportunity; manufactured commentary is.

### Positive feedback

- Call out genuinely good work: a clean abstraction, a well-handled edge case, a thoughtful test, a readable refactor of previously tangled code. Positive feedback teaches just as much as critical feedback, because it tells the author (and other readers) what to repeat.
- Be specific about *why* it's good ("this fake's configurable failure mode makes the error-path test trivial to read") rather than generic praise ("nice!", "great PR!"). Specificity is what separates genuine recognition from filler.
- Do not balance criticism with padding. If a change warrants only critical comments, that's fine; forced compliments preceding critique are transparent and undermine trust in the genuine ones.

### Justify and cite

- Every critical comment needs a justification the author can evaluate: what the problem is, why it matters (bug, maintainability, performance, accessibility, convention), and what better looks like.
- When feedback is based on a documented official recommendation, it's helpful (not required) to link the source directly in the comment: Apple documentation, Human Interface Guidelines, WWDC sessions, Swift Evolution proposals, Swift API Design Guidelines, Apple's Xcode coding-skill files, or the sources in the References section below. A linked source turns "reviewer preference" into shared ground truth, and lets the author judge the recommendation in context. Include a link when it strengthens the comment; skip it when the point stands on its own.
- Community opinions and context (captured condensed alongside the practices below) play a different role: they don't need attribution or links when used in feedback. Use them to flesh out a comment and convey empathy, which is key to healthy reviews — e.g., noting that a pattern is a well-known trap many experienced developers have hit lands very differently than implying an obvious blunder.
- If you cannot articulate why something should change beyond "I would have done it differently," it is a preference, not a finding. Either frame it explicitly as a non-blocking preference or leave it out.
- Distinguish severity clearly in each comment. Separate correctness bugs and security issues (must fix) from modernization opportunities (should fix) and style preferences (consider / non-blocking). The author should never have to guess which comments block approval.
- Distinguish **live** issues from **latent** ones. A defect currently costing users or performance is framed differently than one that only bites under a future change (e.g., an unstable default every caller currently papers over). The fix may be identical; the urgency and tone are not.
- When multiple valid fixes exist, make the call. Recommend the one that fits the code's actual context and say why, rather than listing options A/B/C and leaving the research to the author.

### Respect codebase consistency

- Recommended convention is not the only legitimate convention. Codebases develop internal idioms, and consistency within a codebase has real value: predictability, greppability, lower onboarding cost. A pattern that deviates from Apple's current recommendation but is used consistently throughout the codebase is often the right thing for this changeset to follow.
- Do not flag code for following an established local pattern, even when this document or Apple's guidance recommends otherwise. Changing a codebase-wide convention is an architectural decision that belongs to the team, made deliberately and applied consistently. It is not something to relitigate one pull request at a time.
- If a local convention seems genuinely harmful (a correctness or security problem, not a style mismatch), raise it once as a separate conversation ("worth discussing as a team, outside this PR"), not as a change request on the current diff.
- Be slow to propose significant pivots: rewrites, new architectures, new dependencies, wholesale pattern migrations. A code review is the wrong venue for redesign. Note the idea briefly if it's valuable, and take it to a design discussion.
- New code in a greenfield area of the codebase is different: with no local convention to follow, current recommended practice applies with full force.

### Tone

- Comment on the code, never the author. "This closure retains `self`" rather than "you're leaking memory here."
- Prefer questions where you might be missing context ("was `Task {}` chosen over a task group here for a reason?"). Reviews are frequently wrong about intent; questions surface the missing context without forcing a defensive reply.
- Assume competence and good intent. The author had constraints and context you may not see, especially in an unfamiliar codebase.
- Be concise and concrete. Show the suggested change in code when it's short; a two-line snippet beats a paragraph describing it.

### Scope and verification

- Review the code that is there, not the code you would have written. Flag real problems, not alternative designs of equal merit.
- Check version context before flagging. Several rules below assume recent OS/Swift baselines; a codebase supporting older deployment targets may be using an "outdated" API because it is the only one available to it. Version floors are noted where they matter.
- Verify before asserting. If a claim depends on runtime behavior, API availability, or a compiler rule, confirm it rather than flagging from memory. A review comment that turns out to be wrong costs more credibility than ten correct ones earn.

## Baseline assumptions

The rules below are written against a modern baseline: Swift 6+, iOS 17+ deployment target, Xcode 16+. When reviewing a codebase with an older deployment target or toolchain, downgrade version-gated findings accordingly (noted inline where the floor is commonly hit).

## Swift instructions

- Prefer modern Swift concurrency (async/await). Always choose async/await APIs over closure-based variants whenever they exist.
- Shared UI state should use `@Observable` classes (iOS 17+) with `@State` (for ownership) and `@Bindable` / `@Environment` (for passing).
- Flag `ObservableObject`, `@Published`, `@StateObject`, `@ObservedObject`, and `@EnvironmentObject` in iOS 17+ codebases unless they are unavoidable or exist in legacy/integration contexts where changing architecture would be complicated. In pre-iOS 17 codebases these are the correct patterns — do not flag them.
- Prefer Swift-native alternatives to Foundation methods where they exist, such as using `replacing("hello", with: "world")` (iOS 16+) with strings rather than `replacingOccurrences(of: "hello", with: "world")`.
- Prefer modern Foundation API, for example `URL.documentsDirectory` to find the app's documents directory, and `appending(path:)` to append strings to a URL (both iOS 16+).
- Never use C-style number formatting such as `Text(String(format: "%.2f", abs(myNumber)))`; always use `Text(abs(change), format: .number.precision(.fractionLength(2)))` instead.
- Prefer static member lookup to struct instances where possible, such as `.circle` rather than `Circle()` (iOS 17+), and `.borderedProminent` rather than `BorderedProminentButtonStyle()`.
- Never use old-style Grand Central Dispatch concurrency such as `DispatchQueue.main.async()`. If behavior like this is needed, always use modern Swift concurrency.
- Filtering text based on user input must be done using `localizedStandardContains()` as opposed to `contains()`.
- Avoid force unwraps and force `try` unless the failure is unrecoverable.
- Never use legacy `Formatter` subclasses such as `DateFormatter`, `NumberFormatter`, or `MeasurementFormatter` (iOS 15+). Always use the modern `FormatStyle` API instead. For example, to format a date, use `myDate.formatted(date: .abbreviated, time: .shortened)`. To parse a date from a string, use `Date(inputString, strategy: .iso8601)`. For numbers, use `myNumber.formatted(.number)` or custom format styles. (Exception: a cached `Formatter` may be a deliberate hot-path optimization; confirm before flagging.)

## Swift concurrency instructions

- Identify the Swift language mode and concurrency settings first — the correct review posture differs between Swift 5.x, 6.0/6.1, and 6.2+.
- Under Swift 6.2 Approachable Concurrency: code runs single-threaded by default and async functions stay on the calling actor. Concurrency is opt-in, not accidental. Under Swift 6.0/6.1 (and earlier), nonisolated async functions hop off the calling actor — do not assume 6.2 semantics when reviewing older toolchains.
- `@Observable` classes that drive UI must be `@MainActor` (unless the target has main-actor default isolation enabled). Flag any `@Observable` class missing this annotation.
- `@concurrent` (Swift 6.2+) is only for CPU-intensive work (image processing, compression, complex computation). Flag speculative background offloading — most async code does not need it.
- Prefer structured concurrency (`withTaskGroup`, `withThrowingTaskGroup`) over unstructured `Task {}` loops. `Task {}` is acceptable only for fire-and-forget work or bridging from sync to async.
- Flag `@unchecked Sendable` used to silence compiler errors — it hides real data races. Actors, value types, or `sending` parameters are the correct fixes. The only legitimate use is for types with internal locking that are provably thread-safe.
- If an API offers both `async`/`await` and closure-based variants, the `async`/`await` variant should be used.
- Use `Task.sleep(for:)` (iOS 16+), never `Task.sleep(nanoseconds:)`.
- Protocol types used across actor boundaries (injected clients, services) should be `Sendable`.
- Prefer isolated conformances (`extension MyType: @MainActor SomeProtocol`, Swift 6.2+) for MainActor types conforming to non-isolated protocols over `nonisolated` workarounds.
- If the compiler reports a data race, the code has a real concurrency issue — the fix is to change the code, never to suppress the diagnostic.

## SwiftUI instructions

- Always use `foregroundStyle()` instead of `foregroundColor()` (iOS 15+).
- Always use `clipShape(.rect(cornerRadius:))` instead of `cornerRadius()` (iOS 16+ for `.rect`).
- Always use the `Tab` API instead of `tabItem()` (iOS 18+; `tabItem()` is correct below that).
- Never use the `onChange()` modifier in its 1-parameter variant; either use the variant that accepts two parameters or accepts none (iOS 17+).
- Never use `onTapGesture()` unless the code specifically needs a tap's location or the number of taps. All other usages should be `Button` (this also matters for accessibility).
- Never use `UIScreen.main.bounds` to read the size of the available space.
- Views should not be broken up using computed properties; extract new `View` structs instead (see "SwiftUI invalidation and performance" below for why: invalidation boundaries).
- Do not force specific font sizes; prefer Dynamic Type.
- Use the `navigationDestination(for:)` modifier to specify navigation, and always use `NavigationStack` instead of the old `NavigationView` (iOS 16+).
- If using an image for a button label, text should be specified alongside, like this: `Button("Tap me", systemImage: "plus", action: myButtonAction)`.
- When rendering SwiftUI views to images, prefer `ImageRenderer` to `UIGraphicsImageRenderer` (iOS 16+).
- Don't apply the `fontWeight()` modifier without good reason. To make text bold, use `bold()` instead of `fontWeight(.bold)`.
- Do not use `GeometryReader` if a newer alternative would work as well, such as `containerRelativeFrame()` or `visualEffect()` (both iOS 17+).
- When making a `ForEach` out of an `enumerated` sequence, it should not be converted to an array first: prefer `ForEach(x.enumerated(), id: \.element.id)` over `ForEach(Array(x.enumerated()), id: \.element.id)`. (Swift 6.1+ — `.enumerated()` conforms to `RandomAccessCollection` there; the `Array(...)` wrapper is required on earlier toolchains.) The id must come from the element (`\.element.id`), never the offset — see "ForEach and lists" below.
- When hiding scroll view indicators, use the `.scrollIndicators(.hidden)` modifier rather than `showsIndicators: false` in the scroll view initializer (iOS 16+).
- Use the newest ScrollView APIs for item scrolling and positioning (e.g. `ScrollPosition` (iOS 18+) and `defaultScrollAnchor` (iOS 17+)); avoid older APIs like `ScrollViewReader` where the newer ones suffice.
- View logic should live in observable model/engine classes (`@Observable @MainActor`) so it can be tested. Views should be thin: render state, forward user actions to methods on the model.
- Avoid `AnyView` unless it is absolutely required (acceptable at composition seams such as factory `makeRootView()`-style return types).
- Avoid UIKit unless there is a concrete need SwiftUI cannot meet.
- Watch for hard-coded colors, spacing, and typography values scattered through views — reusable tokens/components belong in a shared design-system layer, whatever form that takes in the codebase under review.

## SwiftUI invalidation and performance

Source: Apple's official SwiftUI coding skill (`swiftui-specialist`), shipped with Xcode 27 (WWDC 2026). Its organizing principle: **a view is SwiftUI's unit of invalidation, and most performance problems come from invalidating more than you need to.** These are identity/invalidation fundamentals that apply to any SwiftUI codebase regardless of deployment target, except where a floor is marked. Deep links go to the [community mirror of Apple's skill files](https://github.com/superagents-lab/xcode27-skills) (Apple-authored content; canonical copies ship inside Xcode 27).

### View structure

- **Extract sections into separate `View` structs, not computed properties.** Computed properties share the parent's invalidation boundary and don't reduce update cost; separate view types create their own boundaries and re-run only when their inputs change ([Apple: structure.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/structure.md#always-use-separate-view-types-for-sections-not-computed-properties)). `@ViewBuilder` properties/functions have the same limitation — they organize code but do *not* create a new invalidation boundary. Carve-out: tiny fragments reused two or three times within one body, with no independent invalidation story, may stay as computed properties; the rule targets factoring done for organization or body length. Community note: this contradicts advice in many popular tutorials, so expect to see the computed-property style in the wild; Apple's own skill has now settled the question.
- **Multi-section detail views always factor per-section.** A `SomethingDetailView` with header/gallery/description/reviews sections gets one `View` struct per section with narrow inputs and a thin composing parent — never `private var header: some View` ([Apple: structure.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/structure.md#multi-section-detail-views)).
- **Keep view `init` cheap.** A view's `init` runs on every parent body re-evaluation — inside lists or animated parents, that can be many times per second. Treat it as a constant-time copy of inputs into stored properties; flag decoding, formatter construction, or any real work in an init ([Apple: structure.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/structure.md#keep-view-init-cheap)).
- **Don't use a `Group` with only a single child view.** A single-child `Group` has no visual effect but wraps the view in an extra `Group<Child>` type, and every modifier chained afterwards is type-checked against that wrapper — needless compile-time overhead. Drop the `Group` and chain modifiers directly on the child. A `Group` around an `if`/`else`, a `ForEach`, or multiple siblings is doing real work and is fine ([Apple: structure.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/structure.md#single-child-group)).
- **Skip the custom `.if` view modifier.** The community "conditional modifier" extension on `View` (an `if` function taking a condition and a transform) destroys structural identity every time the condition flips: the if/else produces two different view types, resetting any `@State` in the subtree and breaking animations. Prefer ternaries inside modifiers (`.foregroundStyle(isHighlighted ? .red : .primary)`), which preserve identity and animate smoothly ([Apple: modifiers.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/modifiers.md); the community flagged this back in 2021 — [Eidhof, objc.io](https://www.objc.io/blog/2021/08/24/conditional-view-modifiers/) — and Apple's skill has now made it official).
- **Use the `@Animatable` macro for animatable custom Views/Shapes** (iOS 26+) instead of hand-writing `animatableData`; `@AnimatableIgnored` opts out non-animatable stored properties. Write explicit `animatableData` only when interpolation needs custom logic (clamping, normalization) — `AnimatableValues` at deployment ≥ iOS 26, `AnimatablePair` below ([Apple: animations.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/animations.md)).

### Data flow and Observation

- **Take narrow value-type inputs.** SwiftUI compares value-type inputs field by field, so a view declared with a whole struct invalidates on changes to fields it never reads — take only the fields the view uses (forwarding a field to a subview counts as using it; a parent that takes five fields and routes them to children is correctly factored). Large payload structs shouldn't thread through the view tree at all: every body evaluation deep-compares the entire payload. Break them into per-view structs, or hold them in an `@Observable` model — reference types compare by pointer, and observation tracks per-property ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#pass-views-only-the-data-they-read), [§ large inputs](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#watch-the-cost-of-large-value-type-inputs)).
- **`@State` should be `private`.** Per Apple: recommend the change when access control is already written, but don't apply it silently ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#view-local-state-with-state)).
- **Don't build a `Binding` using get/set closures.** A closure-pair `Binding(get:set:)` is recreated on every `body` evaluation, and SwiftUI can't compare closures, so it must re-evaluate the child view even when nothing changed. Prefer a model `subscript` and let `@Bindable` derive the binding through it — a stable KeyPath SwiftUI can compare ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#use-keypath-bindings-not-closure-bindings)). The `@Bindable` fix is iOS 17+; the underlying problem applies wherever closure bindings are passed to child views.

  ```swift
  // Avoid
  ScoreRow(player: player, score: Binding(
      get: { model.scores[player.id, default: 0] },
      set: { model.scores[player.id] = $0 }))

  // Prefer: subscript(scoreFor:) on the @Observable model, then
  ScoreRow(player: player, score: $model[scoreFor: player])
  ```

- **Make `@Observable` property types `Equatable`.** The macro's generated setters skip invalidation when the new value equals the old one, but only if the property type conforms to `Equatable`. Applies to collections too: an `Array` is `Equatable` only when its element type is (iOS 17+, Observation) ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#make-observable-property-types-equatable)). Community note: nothing warns about this, which makes it a high-value thing to catch in review.
- **Observation's granularity is the stored property, not fields within it.** Reading any field of a stored struct, or any element of a stored collection, establishes a dependency on the *whole* property — one edit anywhere in it invalidates the view ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#per-property-dependency-granularity-on-observable-models)). Three fixes, chosen by shape:
  - Cache derived values as their own stored properties recomputed in `didSet` — a computed property is *not* a fix, since its body reads the wide property and establishes the same dependency transitively ([§ cache derived values](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#cache-derived-observable-values-computed-properties-still-establish-dependencies-transitively)).
  - Flatten a stored struct's fields into individual `@Observable` properties so each is tracked separately; keep the struct alongside (synced via `didSet`) if it must round-trip to a server ([§ expose struct fields](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#expose-struct-fields-as-individual-observable-properties)).
  - Extract a smaller `@Observable` when many views share one piece of data, bounding each view's dependency surface ([§ extract a smaller model](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#extract-a-smaller-observable-when-many-views-share-data)).
- **Don't over-correct: multiple individual property reads are fine.** A view reading several already-narrow properties from one model is not over-subscribed — per-property tracking scopes it correctly, and splitting it into per-property subviews adds indirection without changing what re-runs ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#multiple-individual-observable-property-reads-are-fine)). Don't flag it.
- **Rows shouldn't reach back into the model.** A row that looks up its element by index/key (`state.users[index]`) makes every row depend on the whole collection, so one edit invalidates all rows. Pass the field for single-field rows; for rows observing several fields, model elements as *persisted* per-element `@Observable` instances — vending fresh instances on every read hands each row a new reference and re-runs everything ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#pass-observable-collection-elements-directly-to-row-views)).
- **Isolate `.onChange`-only reads from expensive bodies.** A dependency read solely for `.onChange` still counts as a body read, invalidating the whole view when it changes. Extract the dependency and its `.onChange` into a small `ViewModifier` so only the modifier re-runs. Apply only when the body is expensive and the value isn't also rendered; otherwise it's needless indirection ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#isolating-onchangeof-side-effect-invalidation)).
- **Prefer `@Entry` for custom environment/transaction/container/focused values** (iOS 18+). Per Apple, a manual `EnvironmentKey` conformance plus get/set extension property is the boilerplate `@Entry` was designed to replace — surface the refactor as a top-line finding, not a stylistic footnote ([Apple: dataflow.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/dataflow.md#entry-macro)).

### Environment

- **Don't store closures in custom `Environment` keys.** SwiftUI can't compare closures, so every child view reading the key is invalidated each time the environment updates. Create a dedicated type implementing the behavior via `callAsFunction()` (callable with closure syntax, e.g. `submit(draft)`) with any captured data as stored properties — wrapping the closure in a struct, or hoisting it to a `let` on the View, are explicitly *not* fixes ([Apple: environment.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/environment.md#closures-in-the-environment)). Carve-out: framework action types (`OpenURLAction`, `DismissAction`, `RefreshAction`, and their keys) are designed to wrap closures — passing a closure to those is the intended API; never flag it.
- **Keep high-frequency values out of the environment.** Every environment write incurs a comparison cost for every reader in the subtree. Scroll offsets, geometry sizes, drag positions, and timer ticks belong in an `@Observable` model exposing *coarsened* values (`isWide`, `isVisible`) so views invalidate on boundary crossings, not every frame; for lists, per-item models get each row down to ~2 invalidations per scroll-through. Purely visual scroll effects should use `scrollTransition`/`visualEffect`, which skip body re-evaluation entirely ([Apple: environment.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/environment.md#rapidly-updating-environment-values)).
- **Environment defaults must be stable.** `@Entry` wraps its default in a computed getter, so `@Entry var model = Model()` (or `Date()`, `UUID()`) produces a fresh value on every fallback read — and readers falling back to it invalidate on every unrelated environment write. Fixes: back it with a `static let`; declare the key manually with `static let defaultValue`; or make it Optional with a `nil` default (prefer this when readers do sentinel "absence" checks like `id.isEmpty` — the sentinel is an absence test in disguise). Adding `Equatable` is *not* a fix. Don't flag stable defaults (literals, enum cases, `nil`, references to `static`/module-level `let`s) — the test is "does the expression return a different result between calls," not "does the struct contain a class" ([Apple: environment.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/environment.md#unstable-environment-default-values)).
- **Delete unused `@Environment`/`@FocusedValue` declarations.** The keypath form subscribes the view even when body never reads the value, so removal is an active performance fix; an unused type form (`@Environment(Model.self)`) is dead-code cleanup only, since observation tracks property reads ([Apple: environment.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/environment.md#unused-environment-reads)).

### ForEach and lists

- **Give every `ForEach` a stable element identity.** An index describes a position, not an element: reordering or inserting makes every later index point to a different element, losing focus and animating updates as remove/insert. Never `indices` with `id: \.self`, and never an `id` derived from a mutable property (identity changes mid-edit). Prefer `Identifiable` elements ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#avoid-collection-indices-as-identity)). Community note: one of the most common SwiftUI mistakes; even experienced developers have shipped it.
- **The identity rules apply to every data-driven initializer**, not just literal `ForEach`: `List(_:id:)`, `Table`, `OutlineGroup`, `Picker` over a collection, and friends ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#applies-to-other-data-driven-initializers)).
- **Don't mint ids per body evaluation.** Constructing `Identifiable` values inside `body` (`titles.map { Item(title: $0) }` where `Item` has `let id = UUID()`) gives the whole collection new ids every update — state resets, rows flicker, animations degenerate. Ids must live in storage that outlives `body`: a natural key, or a UUID created once in the model layer ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#dont-create-a-new-id-on-every-body-evaluation)).
- **Keep the id cheap to hash.** `id: \.self` on a large `Hashable` struct hashes every field of every row on every diff. The id should be a small primitive (`UUID`, `Int`, short `String`, `URL`); the fix is picking the right id, not removing the `Hashable` conformance, which may be used elsewhere ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#keep-the-id-cheap-to-hash)).
- **Don't sort or filter inline in `ForEach`.** The collection expression re-runs on every body evaluation, including invalidations unrelated to the list. Cache the derived collection on the model (recomputed in `didSet`) or in `@State` via `onChange`; cheap transforms (`prefix(n)`, trivial maps) are fine inline ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#dont-sort-or-filter-inline-in-foreach)).
- **Prefer single-top-level-view (unary) rows inside `List`.** List has a fast path when each row produces exactly one top-level view; a top-level `switch` or bare `if` produces multiple possible shapes and forces every row's body to be evaluated just to compute ids. Wrap the branch in a container (`VStack { switch ... }`), and filter the collection before the `ForEach` rather than returning an empty row from a top-level `if` without `else`. Don't "fix" a switch by flattening it into conditional modifiers — that breaks the moment cases produce structurally different views ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#prefer-unary-row-views-in-list)).
- **Avoid `AnyView` as a `ForEach` row.** Type erasure erases structural identity, defeating the same fast path — and a `@ViewBuilder` helper returning a bare `switch` is not a fix either; the row is still multi-shape. Use a concrete row view with the branch inside a single-root container ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#avoid-anyview-as-a-foreach-row)).
- **Diagnostic:** launching with `-LogForEachSlowPath YES` logs every `ForEach` in a lazy container whose row builder produces a non-constant number of views — useful triage before flagging by eye ([Apple: foreach.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/foreach.md#diagnosing-with--logforeachslowpath)).

### Soft-deprecated APIs

SwiftUI marks many older APIs deprecated at placeholder version `100000.0` — no compiler warning fires, but they shouldn't appear in new code: `NavigationView`, the `Alert`/`ActionSheet` structs, `MenuButton`, `RotationGesture`, and many more ([Apple's full list](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/soft-deprecated-apis.md)). Review posture, per Apple's own guidance ([soft-deprecation.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/soft-deprecation.md)): treat these as informational, not urgent — they still compile and work; flag them only in the code under review, never in adjacent code the changeset doesn't touch; and never introduce new usages in code you suggest.

## Liquid Glass (iOS 26+)

Liquid Glass is the system design language introduced across Apple platforms in iOS 26 ([Adopting Liquid Glass](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass), [WWDC25 "Meet Liquid Glass"](https://developer.apple.com/videos/play/wwdc2025/219/)). Apps built with the Xcode 26 SDK get system components re-rendered in glass automatically; custom surfaces adopt it via the glass APIs. Important for review: **adopting Liquid Glass does not require an iOS 26 deployment target** — `if #available(iOS 26, *)` with a non-glass fallback is the recommended bridge, so suggesting glass adoption is fair game even in codebases supporting older iOS versions (as a non-blocking suggestion, per the conduct rules above).

- **Gate with `if #available(iOS 26, *)` and provide a non-glass fallback** — typically the same shape backed by a material. Never flag the fallback branch's material usage as outdated; it is the correct pre-26 path.

  ```swift
  if #available(iOS 26, *) {
      content.glassEffect(.regular.interactive(), in: .rect(cornerRadius: 16))
  } else {
      content.background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 16))
  }
  ```

- **Prefer native glass APIs over custom approximations.** `glassEffect(_:in:)`, `GlassEffectContainer`, and `.buttonStyle(.glass)` / `.buttonStyle(.glassProminent)` are the supported surface; flag hand-rolled blur/opacity stacks imitating glass ([Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)).
- **Wrap multiple glass elements in a `GlassEffectContainer`.** The container is what gives correct rendering performance and lets nearby effects merge and morph; flag sibling `glassEffect`s with no container ([GlassEffectContainer](https://developer.apple.com/documentation/SwiftUI/GlassEffectContainer)).
- **Modifier order matters.** `.glassEffect()` is applied after the layout and appearance modifiers (padding, font, frame) that define the surface it wraps.
- **`.interactive()` only where the user interacts.** It makes glass react to touch/pointer; flag it on static, decorative surfaces.
- **Morphing transitions need identity.** Views that appear/disappear with glass morphing use `glassEffectID(_:in:)` with a `@Namespace`, inside a container, with the hierarchy change wrapped in an animation ([GlassEffectTransition](https://developer.apple.com/documentation/SwiftUI/GlassEffectTransition)).
- **Consistency and restraint.** Keep shapes and tinting consistent across related elements; use tint sparingly to suggest prominence. Glass belongs on the floating controls/navigation layer above content, not spread across the content itself ([HIG: Materials](https://developer.apple.com/design/human-interface-guidelines/materials)).
- **`UIDesignRequiresCompatibility` is a temporary escape hatch.** It opts an app out of the new design while migrating; flag it as tech debt with a plan attached, not a permanent setting ([Adopting Liquid Glass](https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass)).

## State and data flow

- `@State` in views is only for view-local UI state (scroll position, sheet presentation, animation). Business state belongs in observable model objects.
- Root views should own their model as `@State` and pass it to children via `@Bindable`/`@Environment`; child views render state and call model methods.
- Model objects should receive their dependencies (network clients, persistence, services) via initializer injection behind protocol seams — not reach out to singletons or SDKs directly — so they can be tested in isolation.
- Error mapping belongs in the model layer: SDK/network errors mapped to user-friendly messages in a dedicated place, not scattered through views.
- Separate state observation from side effects. `.task` blocks are for observing async state and updating the model. Non-rendering side effects (analytics, logging) belong in `.onChange(of:)` — and when the observed value isn't also rendered and the body is expensive, isolate the `.onChange` into a `ViewModifier` (see "Data flow and Observation" above).

## SwiftUI preview instructions

Previews are the primary visual verification tool for views, complementing unit tests on the model layer.

- Every view file should have at least one `#Preview` block. Model classes and helpers don't need previews.
- Previews should use fake/stub dependencies, never real network or SDK clients.
- For views with async operations or multiple states, look for named preview variants: `#Preview("Loaded") { }`, `#Preview("Error") { }` — a single happy-path preview for a multi-state view is a coverage gap.
- Fakes should support configurable scenarios via init parameters (e.g., a `shouldSucceed: false` toggle for error states).
- Previews should use realistic, representative data — not "Test" or placeholder strings.
- Views using navigation APIs should be wrapped in `NavigationStack { }` in their preview.

## SwiftData instructions

If SwiftData is configured to use CloudKit:

- Never use `@Attribute(.unique)`.
- Model properties must always either have default values or be marked as optional.
- All relationships must be marked optional.

## Security

- Sensitive data (tokens, passwords, API keys) belongs in Keychain, never in `UserDefaults`, `@AppStorage`, or files. Check that the `kSecAttrAccessible` value fits each item's use case.
- No hardcoded secrets or API keys in source. Look for keys committed in plists, config files, and string literals.
- Cryptographic operations should use CryptoKit (not CommonCrypto). AES-GCM with 256-bit keys for encryption; keys stored in Keychain or Secure Enclave.
- Face ID / Touch ID usage requires `NSFaceIDUsageDescription` in Info.plist and a passcode fallback.
- ATS must stay enabled. Flag `NSAllowsArbitraryLoads = true`. Exception domains are acceptable only for third-party servers that cannot be upgraded.
- No sensitive data (tokens, passwords, PII) in logs at any level. Sensitive `Data` should be cleared from memory after use.
- User-provided URLs and file paths must be validated against allowed schemes and directories.
- `PrivacyInfo.xcprivacy` must be present with all required-reason API declarations. Third-party SDKs should include their own privacy manifests.
- Security-framework and Keychain usage should sit behind a seam (wrapper/service), not be sprinkled through feature code.
- Be practical — match the security posture to the app's threat model. Don't demand bank-grade security for data that isn't sensitive.

## User feedback

- No raw technical errors in UI — no `Error.localizedDescription`, SDK error codes, HTTP status codes, or backend error strings shown to users.
- Every error message must be actionable: tell the user what they can do (retry, fix input, check settings, contact support), not just what went wrong.
- Feedback level should match severity: inline for field validation, banner for system state (offline), toast for transient results (save failed), modal/alert only when a decision is required.
- If the user took an action and it failed, they should know. If something failed in the background and auto-recovery handles it, the user should not be interrupted.
- Plain, empathetic language. No blame ("You failed to..."), no jargon. One sentence for the problem, one for the action.
- All user-facing error messages must be localized — no hardcoded English strings in catch blocks or error state views.
- Errors should map to user messages systematically (per-feature or centralized), not via scattered inline strings in catch blocks, and there should always be a generic fallback for unexpected errors (e.g., "Something went wrong. Please try again.").
- Recovery paths should exist: retry buttons for transient failures, offline banners with auto-retry, sign-in redirect for expired auth, clear empty-state messaging with suggested actions.

## Logging and telemetry

- No `print()` or `debugPrint()` in production code. Logging should go through a structured logger (os.Logger or an injected logging abstraction) with appropriate levels: trace/debug for diagnostics, info for state changes, warn for recoverable issues, error/fatal for failures.
- Crash/error reporting should capture actual bugs, unexpected failures, and data corruption — not expected UX errors (user cancelled, validation failed, wrong password, permission denied by user). Flag noisy reporting of expected conditions.
- No PII (emails, tokens, passwords, user content) in log attributes or error report extras.
- Features should depend on a logging abstraction, not on a vendor SDK (Sentry, Crashlytics, etc.) directly, so the vendor can be swapped and fakes can be injected in tests.
- Third-party telemetry SDK initialization should happen once at app startup, not be scattered through features.

## Testing

- **Framework**: Swift Testing (`@Test`, `#expect`) for new tests on Xcode 16+ toolchains; XCTest is expected in older codebases and for UI tests. Don't flag XCTest as legacy when the toolchain or deployment setup requires it.
- **Test naming**: prefer `@Test("Given X, when Y, then Z")` display names with short camelCase function names. Flag backtick raw identifiers for test names — they can fail on CI runners with older Swift toolchains.
- **Test behavior, not implementation**: tests should assert observable outcomes (state, return values, errors), not internal details (view hierarchy, method calls, private state). Flag tests coupled to implementation details.
- **Fakes over mocking frameworks**: hand-written configurable fakes behind protocol seams. Dependencies should be injectable so model/logic classes can be tested with fakes.
- **Coverage priorities (risk-based)**: data integrity, validation rules, business/model logic, error handling, and state transitions. Don't demand tests for SwiftUI framework behavior or trivial getters.
- **SVIBE methodology** for identifying missing cases: States, Validation, Interactions, Boundaries, Errors.
- **Model/engine testing**: logic classes should be unit-testable by injecting fakes for all external dependencies — state transitions, error mapping, async behavior. Views are verified visually via previews, not unit tests.
- **Golden rule**: tests verify *expected* behavior. If a test fails, the fix belongs in the code, not the test. Flag test changes that weaken assertions to make failing code pass.

## Code organization

- Organize code by feature, not by file type or technical layer. Each feature owns its views, state, domain logic, resources, and tests.
- Keep the app/composition layer thin: lifecycle, dependency wiring, navigation bootstrapping. Flag business logic accumulating there.
- Cross-feature reuse should go through well-named shared modules or contracts, not features reaching into each other's internals. Avoid circular dependencies at all costs.
- Reusable UI belongs in a shared UI/design-system layer; cross-cutting infrastructure (networking, persistence, analytics) in clearly named core modules.
- Don't create "shared" abstractions prematurely — only extract shared code once reuse is real and the boundary is clear. Flag both directions: copy-paste that should be shared, and speculative abstractions with one consumer.
- Break different types into different Swift files rather than placing multiple structs, classes, or enums into a single file.
- Prefer protocol-driven seams and dependency injection across module boundaries so components can be tested in isolation.
- Third-party SDK usage should be wrapped behind an abstraction rather than imported throughout feature code; new third-party dependencies deserve scrutiny — prefer internal code and Apple frameworks first.

## Localization

The bullets below marked "Apple" deep-link to the `localization.md` reference of Apple's Xcode 27 SwiftUI skill.

- User-facing strings should be localizable. In SwiftUI, English text as string literals (`Text("Welcome Back")`) is automatically treated as a localization key — flag camelCase symbol keys, which break in modularized targets where `bundle:` must be specified. (Apple accepts either convention when applied consistently; the multi-module bundle constraint is the practical tiebreaker.)
- In multi-module apps, each module needs its own string catalog and a bundle for lookups. Inside frameworks and packages prefer `#bundle` over `Bundle.module` — without an explicit bundle, lookup silently falls back to `Bundle.main` and the string renders unlocalized ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#bundle-for-swift-packages-and-frameworks)).
- Don't wrap string literals in `NSLocalizedString` or `String(localized:)` when passing them to `LocalizedStringKey`-taking views (`Text`, `Button`, `.navigationTitle`) — the literal is already a key, and eager resolution ignores `\.locale` overrides. `Text(verbatim:)` opts a literal out of localization ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#swiftui-views-localize-string-literals-automatically)).
- String *variables* passed to `Text` are not localized (the `StringProtocol` overload runs). Model a known set of display strings as a type exposing `LocalizedStringResource`, and prefer `LocalizedStringKey`/`LocalizedStringResource` over `String` for user-facing properties on models and view models — resolution then happens at display time with locale context intact. Apply when designing new types or changing user-facing text; don't sweep existing `String` properties in unrelated edits ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#localizing-variables-and-custom-types)).
- Interpolate, never concatenate: `Text("Error: \(message)")` produces a localizable format string; `Text("Error: " + message)` produces a plain `String`. Never assemble sentences from separately localized fragments — word order varies across languages ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#string-interpolation-vs-concatenation)).
- Bake casing into the string (`"SECTION HEADER"`) rather than transforming at runtime with `.textCase(.uppercase)` — a runtime transform forces the same casing on every translation. If a transform is unavoidable, use `.localizedUppercase`/`.localizedCapitalized`, which honor locale rules ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#casing)).
- Format dates, numbers, and currency with format styles, never hardcoded format strings — this is a locale-correctness issue, not just API modernity (see the `FormatStyle` rule in Swift instructions). `Array.formatted()` beats `joined(separator: ", ")` for lists; when `DateFormatter` is genuinely unavoidable, use `setLocalizedDateFormatFromTemplate(_:)` ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#formatting-dates-numbers-and-currencies)).
- Layout for localization: `.leading`/`.trailing`, never `.left`/`.right` (only the former flip for RTL); no hardcoded frames on text (use `ViewThatFits` when longer translations might not fit); text styles, not fixed point sizes, so line height adapts per script ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#layout-for-localization)).
- Use `@Environment(\.locale)` instead of `Locale.current` in views (respects preview and per-view overrides), and `String(localized:)` instead of `NSLocalizedString` outside views — never interpolate inside `NSLocalizedString`, which defeats key extraction ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#stringlocalized-outside-swiftui-views)).
- Add translator `comment:`s for ambiguous strings ("Edit" the noun vs. the verb) and describe interpolated placeholders by position — translators don't see Swift variable names ([Apple: localization.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-specialist/references/localization.md#comments-for-translators)).
- Flag hardcoded user-facing strings in error paths — catch blocks are the most common place localization is missed.

## References

Sources to link when leaving feedback grounded in a documented recommendation. Cite the most specific source available (a section or session, not a homepage).

Maintenance: as this document evolves, capture a direct link for each practice where one exists — a bullet without a link is fine, one with a link is better. Record community opinions/context condensed (a line or two) inline with the practice they concern. Official links are for citing in feedback when helpful; community context is for shaping empathetic feedback and needs no attribution when used in a review comment.

### Apple

- Apple Developer Documentation — https://developer.apple.com/documentation/
- Human Interface Guidelines — https://developer.apple.com/design/human-interface-guidelines/
- App Review Guidelines — https://developer.apple.com/app-store/review/guidelines/
- WWDC session videos — https://developer.apple.com/videos/

### Swift

- Swift API Design Guidelines — https://www.swift.org/documentation/api-design-guidelines/
- Swift Evolution proposals — https://www.swift.org/swift-evolution/
- Swift Migration Guide (concurrency) — https://www.swift.org/migration/documentation/migrationguide/

### Apple Xcode 27 agent skills

Apple ships seven agent skills with Xcode 27 (WWDC 2026): `swiftui-specialist`, `swiftui-whats-new-27`, `uikit-app-modernization`, `test-modernizer`, `audit-xcode-security-settings`, `c-bounds-safety`, `device-interaction`. They have no standalone pages on developer.apple.com; the canonical copies ship inside Xcode 27.

- WWDC26 session 259, "Xcode, agents, and you" — https://developer.apple.com/videos/play/wwdc2026/259/
- Coding intelligence (Apple docs) — https://developer.apple.com/documentation/xcode/coding-intelligence
- Extending and customizing agents (Apple docs) — https://developer.apple.com/documentation/Xcode/extending-and-customizing-agents
- Skill-file mirror for deep links — https://github.com/superagents-lab/xcode27-skills (Apple-authored content, community-exported; the linkable source for individual practices until a local Xcode 27 install exists). The `swiftui-specialist` reference files (structure, dataflow, environment, foreach, modifiers, animations, localization, soft-deprecation + API list) and the `swiftui-whats-new-27` reference files (state-macro, content-builder, reorderable, swipe-actions, async-image, toolbar, item-binding, document-based-apps, deprecations) are incorporated into this document as of 2026-08-09.
- Antoine van der Lee, "SwiftUI Best Practices, straight from Apple's Xcode 27 Agent Skill" — https://www.avanderlee.com/ai-development/swiftui-best-practices-xcode-27-agent-skill/ (companion: "Using Xcode 27's Agent Skills in Claude, Codex, and Cursor" — https://www.avanderlee.com/ai-development/using-xcode-27s-agent-skills-in-claude-codex-and-cursor/)
- Hacking with Swift, "Agent skills in Xcode: How to install and use them today" — https://www.hackingwithswift.com/articles/283/how-to-install-and-use-ai-agent-skills-in-xcode
- Community superset skill — https://github.com/avdlee/swiftui-agent-skill (van der Lee's swiftui-expert-skill: Apple's skill plus areas Apple's doesn't cover — accessibility, navigation, layout, previews, animations)

Community context: Apple's SwiftUI skill is deliberately narrow — invalidation/performance, data flow, ForEach identity, localization, soft-deprecated APIs. A topic being absent from Apple's skill (accessibility, navigation, etc.) is not evidence it's unimportant in review.

### Liquid Glass

- Adopting Liquid Glass — https://developer.apple.com/documentation/technologyoverviews/adopting-liquid-glass
- Applying Liquid Glass to custom views — https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views
- Landmarks: Building an app with Liquid Glass (sample) — https://developer.apple.com/documentation/SwiftUI/Landmarks-Building-an-app-with-Liquid-Glass
- API: `glassEffect(_:in:isEnabled:)` — https://developer.apple.com/documentation/SwiftUI/View/glassEffect(_:in:isEnabled:) · `GlassEffectContainer` — https://developer.apple.com/documentation/SwiftUI/GlassEffectContainer · `GlassEffectTransition` — https://developer.apple.com/documentation/SwiftUI/GlassEffectTransition
- HIG: Materials — https://developer.apple.com/design/human-interface-guidelines/materials
- WWDC25 session 219, "Meet Liquid Glass" — https://developer.apple.com/videos/play/wwdc2025/219/ · WWDC25 session 356, "Get to know the new design system" — https://developer.apple.com/videos/play/wwdc2025/356/

### SwiftUI best practices

- Chris Eidhof, "Why Conditional View Modifiers are a Bad Idea" — https://www.objc.io/blog/2021/08/24/conditional-view-modifiers/ (community precedent for Apple's conditional-modifier guidance)
- Emre Degirmenci, "Splitting Large SwiftUI Views in the Apple's way" — https://emredegirmenci.substack.com/p/splitting-large-swiftui-views-in (practical refactor walkthrough of the view-extraction practice; narrow inputs over stores, @ViewBuilder as organization-not-boundary, start with the noisiest sections first)

<!-- Additional curated SwiftUI best practices to be added here, one entry per practice, so review comments can link the specific source they're based on. -->

## What's new in SwiftUI (SDK 27)

Source: Apple's `swiftui-whats-new-27` coding skill (Xcode 27), deep-linked to the [community mirror](https://github.com/superagents-lab/xcode27-skills/tree/master/swiftui-whats-new-27). This section serves two review purposes: recognizing **new APIs** so their iOS/macOS 27 floors get the right treatment in older-target codebases (suggest `if #available(iOS 27, *)` + fallback — the same progressive-adoption stance as Liquid Glass; Apple's references include recommended gating shapes), and recognizing **SDK 27 source-compatibility breaks** so post-SDK-update compile errors are diagnosed as known migrations, not author bugs.

Source-compatibility breaks (flag the right fix, not the obvious one):

- **`@State` is now a macro.** Three new compile-error shapes after an SDK 27 update ([Apple: state-macro.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/state-macro.md)):
  - *"Variable used before being initialized"* when an init assigns a `@State` that also has a declaration-site initial value. The fix is to **drop the initial value at the declaration** and assign only in `init` — reordering the init assignments is the wrong fix. Related review catch: assigning in `init` to a `@State` that keeps its declaration-site value is an anti-pattern that silently doesn't take effect (body sees the declaration value).
  - *"Invalid redeclaration of synthesized property"* when another property wrapper is composed with `@State` — the composition must be restructured.
  - Views whose members are all private no longer get a synthesized private memberwise init; define it explicitly.
- **Result builders unified under `@ContentBuilder`** — blocks are no longer constrained to `View`, which breaks some previously-inferred overload resolution ([Apple: content-builder.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/content-builder.md)): "ambiguous use of 'opacity'/'blendMode'" when passing a modified `ShapeStyle` to non-builder `overlay(...)`/`background(...)` (fix: the trailing-closure form), ambiguous type references when an imported module shadows SwiftUI type names, and type-check timeouts in deeply branching Swift Charts content.
- **Hard deprecations:** `statusBarHidden(_:)` on visionOS is deprecated as a no-op — remove the call ([Apple: deprecations.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/deprecations.md)). On iOS its replacement is `toolbarVisibility(_:for: .statusBar)` (iOS 27+).

New APIs (all SDK 27; gate below an iOS 27 deployment target):

- **Drag-to-reorder in any container.** `.reorderable()` on the `ForEach` plus `.reorderContainer(for:)` on the container (List, stacks, grids, custom layouts); the `move` closure receives a `ReorderDifference` to apply to your own data; sections via `.reorderable(collectionID:)`; integrates with `dragContainer`/`dropDestination`, including per-child `dropDestination(for:isEnabled:)` for drop-to-combine. tvOS unavailable; watchOS lacks the drag-and-drop integration ([Apple: reorderable.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/reorderable.md)).
- **Swipe actions beyond `List`.** Mark the scrollable container with `swipeActionsContainer()` and keep the (iOS 15+) row-level `swipeActions` on each row; new `onPresentationChanged` overload reports reveal/hide. Review catch: row `swipeActions` outside a `List` without the container modifier silently does nothing ([Apple: swipe-actions.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/swipe-actions.md)).
- **`AsyncImage` HTTP caching.** Default caching per server headers is *runtime* behavior on 2027 OS releases — it applies regardless of build SDK or deployment target, so "add image caching" may need no code change at all. Per-request control via the new `AsyncImage(request:)` initializers; custom `URLCache` via `asyncImageURLSession(_:)` (both SDK 27) ([Apple: async-image.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/async-image.md)).
- **Toolbar overflow control.** `visibilityPriority(_:)` (what overflows first), `ToolbarOverflowMenu` (always-overflow items), `.topBarPinnedTrailing` (never overflows), `toolbarMinimizeBehavior(_:for:)` (minimize on scroll), `contentMarginsRemoved(_:)`, and the status bar as a `ToolbarPlacement`. Per-API availability varies by platform — check the reference's table before flagging. `ForEach` as toolbar content back-deploys to iOS 16 when built with the 2027 SDK ([Apple: toolbar.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/toolbar.md)).
- **`alert`/`confirmationDialog` from an item binding.** New `item: Binding<T?>` overloads use the `sheet(item:)` shape: present while non-nil, pass the unwrapped value to `actions`/`message`, reset on dismiss. For per-item dialogs at a 27+ target, prefer this over a synthesized `Bool` + `presenting:` ([Apple: item-binding.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/item-binding.md)).
- **Document apps: `ReadableDocument`/`WritableDocument`.** The modern replacement for `FileDocument`/`ReferenceFileDocument` in new code at 27+ targets (iOS/macOS/visionOS; not watchOS/tvOS): `@Observable` reference-type documents, background reads/writes via `DocumentReader`/`DocumentWriter`, direct file-URL access, and `Subprogress` progress reporting. Review catch: `snapshot(contentType:)` and `apply(snapshot:previous:)` run on the main actor and must stay lightweight — serialization belongs in the reader/writer ([Apple: document-based-apps.md](https://github.com/superagents-lab/xcode27-skills/blob/master/swiftui-whats-new-27/references/document-based-apps.md)).
