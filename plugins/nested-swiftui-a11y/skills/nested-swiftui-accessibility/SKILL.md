---
name: nested-swiftui-accessibility
description: Integrate or review the NestedA11yIDs Swift package in a SwiftUI app, including hierarchical identifiers and UI test queries. Do not use for general VoiceOver audits unrelated to this package.
---

# NestedA11yIDs in SwiftUI projects

Use this skill when a project needs stable, hierarchical accessibility identifiers with the `NestedA11yIDs` package. Inspect the app's existing view structure, dependency setup, and UI tests before editing.

## Integration

1. Confirm the app can use the package's supported deployment targets: iOS 15, macOS 12, tvOS 15, and watchOS 8 or later. Add `https://github.com/SoundBlaster/NestedA11yIDs.git` through the project's existing dependency workflow, using the version range appropriate to the project. In SwiftPM, add the package dependency and the `NestedA11yIDs` product to the app target; in Xcode-managed projects, use the package manager already in use.
2. Import `NestedA11yIDs` in files that use its view modifiers.
3. Put `.a11yRoot("screen")` on the screen or component whose identifiers should share a stable root. Apply `.nestedAccessibilityIdentifier("child")` to meaningful descendants. Nested containers compose dot-separated paths, such as `login.form.email`.
4. Use concise, stable names that describe a control's role. Keep accessibility labels, values, hints, traits, and actions meaningful; identifiers are for automation and do not replace user-facing accessibility.
5. Add or update UI tests to query the resulting identifiers, for example `app.buttons["login.submit"]` or `app.textFields["login.email"]`. Prefer the query type that matches the actual control.

```swift
import NestedA11yIDs
import SwiftUI

struct LoginView: View {
    var body: some View {
        VStack {
            TextField("Email", text: .constant(""))
                .nestedAccessibilityIdentifier("email")
            Button("Sign In") {}
                .nestedAccessibilityIdentifier("submit")
        }
        .nestedAccessibilityIdentifier("form")
        .a11yRoot("login")
    }
}
```

This produces descendant paths `login.form.email` and `login.form.submit` when SwiftUI exposes the controls as expected.

## Review and verification

- Preserve existing `.accessibilityLabel`, `.accessibilityValue`, `.accessibilityHint`, `.accessibilityAddTraits`, actions, and grouping unless the task specifically calls for changing them.
- Each package modifier applies `.accessibilityElement(children: .contain)` to its view. Check that this container behavior fits the view's existing grouping, especially for buttons, rows, and composites. Avoid adding identifiers to every layout-only stack when doing so would create unwanted accessibility containers.
- An empty identifier is ignored by the package. Do not depend on it to clear an identifier inherited from an ancestor.
- A modifier's placement in SwiftUI's view tree matters. If a query fails, inspect the actual accessibility tree and move the modifier to the semantic element SwiftUI exposes; do not guess from the source nesting alone.
- Button accessibility can differ from ordinary leaf views. Verify button queries in a running UI test on each relevant platform instead of treating a modifier-chain or unit test as proof of the runtime identifier.
- Run the focused app UI tests and inspect failures. Package unit tests can verify composition logic, but they do not prove how a particular app's rendered accessibility tree behaves.
- Do not change app code just to add this package when the task only asks for general VoiceOver guidance or when the existing project uses a different identifier convention that must remain intact.

## Source of truth

When behavior is uncertain, inspect the package version resolved by the app and its public API in `Sources/NestedA11yIDs/Public/View+NestedA11y.swift`, modifier implementation, DocC guides, and tests. Do not assume newer repository behavior is present in an older resolved release.
