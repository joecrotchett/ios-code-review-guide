# iOS Code Review Guide

A codebase-agnostic guide for conducting code reviews of iOS/Swift/SwiftUI projects — written to be used by human reviewers and AI coding agents alike.

The guide lives in **[CODE-REVIEW.md](CODE-REVIEW.md)**.

## What's inside

- **Code review conduct** — feedback principles first: every comment must earn its place, genuine (never performative) positive feedback, justify-and-cite, respect for a codebase's internal consistency over external convention, tone, and scope discipline.
- **Technical review guidance** — Swift language and concurrency, SwiftUI, state and data flow, previews, SwiftData/CloudKit, security, user-facing error UX, logging/telemetry, testing, code organization, and localization.
- **SwiftUI invalidation and performance** — the practices from Apple's official `swiftui-specialist` coding skill (shipped with Xcode 27), condensed and deep-linked: view structure, data flow and Observation, environment hygiene, ForEach/List identity, and soft-deprecated APIs.
- **Liquid Glass (iOS 26+)** — adoption and review guidance, including the `if #available(iOS 26, *)` progressive-adoption path for codebases with older deployment targets.
- **References** — direct links to the Apple documentation, WWDC sessions, and community sources each practice is based on, so review comments can cite ground truth instead of reviewer preference.

## Design principles

1. **Version-aware.** Rules carry inline version floors (iOS 16+, iOS 17+, Swift 6.1+, …). A reviewer following this guide should never flag code for using the only API its deployment target allows.
2. **Sourced.** Wherever a practice comes from a documented recommendation, the bullet links the source directly. Official links are for citing in feedback when helpful; condensed community context is for shaping empathetic feedback and needs no attribution in a review comment.
3. **Anti-false-positive.** Carve-outs and "not a fix" traps are recorded alongside each rule (framework action types in the environment are fine; stable defaults don't need "fixing"; multiple property reads don't need model-splitting) so reviews stay credible.
4. **Consistency over convention.** An established local pattern usually beats an external recommendation for the changeset at hand. Convention changes belong to the team, not to one pull request.
5. **Condensed.** One to three lines per practice. The linked sources carry the depth.

## Using it with an AI reviewer

Point your agent at `CODE-REVIEW.md` as its review instructions (for example, reference it from a `CLAUDE.md`, load it as a skill, or paste it into a review prompt). The conduct section governs how findings are reported; the technical sections govern what to look for; the references section supplies the links to cite.

## Sources

Practices are drawn from Apple documentation, the Human Interface Guidelines, WWDC sessions, Apple's Xcode 27 agent skills (via the [community export](https://github.com/superagents-lab/xcode27-skills) — content authored by Apple), and credited community write-ups. See the References section of the guide for the full list.

## Contributing / evolving

The guide grows practice-by-practice: each addition should carry a direct source link where one exists, an inline version floor where it matters, and at most a line or two of community context. Keep it condensed — depth belongs in the linked sources.
