# Expo universal overlay (React Native + react-native-web)

**Verified against:** Expo SDK 53 · React Native 0.79 · React 19 · react-native-web · Expo Router ·
August 2026.
**Verify before applying:** read `app.json` and `package.json` first and trust the installed versions.
SDK releases move fast; anything below that contradicts the repo is stale — report it, do not apply it.

## Plan deltas

- **Read versions, never assume them:** `package.json` (expo, react-native, react versions),
  `app.json` (`newArchEnabled`, `experiments.reactCompiler`). The React Compiler flag decides whether
  memoization is even a valid topic — on SDK 53 it is opt-in beta, automatic only from SDK 54.
- **Routes are not features.** `src/app/**` is routing only: a route file maps a path to a screen and
  returns it. State, fetching, or conditionals in a route file is a planning error.
- **Name the platform-fork rule per task**: inline `Platform.select` for small branches,
  `.ios.tsx` / `.android.tsx` / `.web.tsx` for distinct implementations. Target ~90% shared code.
- **Six decisions this stack forces that have no external authority** — if the conventions file lacks
  them, writing them is task 1:
  1. `src/app/` (Expo routing) vs an `app` layer (providers/global setup) — same word, two meanings
  2. Where routes sit relative to the rest of the directory structure
  3. Whether an architecture linter runs, and how it treats `src/app/`
  4. Barrel/public-API policy, reconciled against Metro bundle size and server/client boundaries
  5. Which directories may contain platform-specific files
  6. Whether the React Compiler is on
- **Native is not failure.** Hardware-intensive work, background sync, and platform widgets stay native.
  A task contorting something into JS to keep the codebase pure is the wrong task.

## Build deltas

**Universal semantics — the trap:** on the web almost everything renders as a `View`, which means
nothing to a screen reader unless annotated. A component that is accessible on iOS can be a semantic
void on web, and neither platform's tests catch the other.

- Express semantics with **`aria-*` props**; `role` infers the HTML element
  (`role="heading"` + `aria-level={2}` → `<h2>`).
- Use **`tabIndex`** for web focusability, not RN's `accessible` / `focusable`.
- **Never change `role` dynamically** — accessibility APIs cannot notify assistive tech of a role change.
- Links use **`href` / `hrefAttrs`**, not an `onPress` that navigates. An `onPress`-only link loses
  middle-click, open-in-new-tab, and crawlability on web.
- Expect divergence to check on both targets: Yoga vs CSS layout, shadow props, hover states creating
  the double-tap problem on touch, nested pressables.
- Push `"use client"` to the interactive leaf, never the route root.
- Lists beyond one screen use a virtualised list, not the basic one. Animations run on worklets, off
  the JS thread.
- Server-only modules need their own entry point (`index.server.ts`), or a barrel drags server code
  into the client graph.

## Review deltas

- `[blocker]` Interactive element with no accessible name on **either** target; `role` mutated at runtime.
- `[blocker]` Navigation rendered without `href` on web.
- `[major]` New code importing `expo-av` (use `expo-video` / `expo-audio`) or `expo-background-fetch`
  (use `expo-background-task`).
- `[major]` Logic in a route file under `src/app/`.
- `[major]` Deep `require('react-native/x/y')` paths — `package.json` `exports` is enabled by default
  from RN 0.79 and these break.
- `[major]` New Architecture disabled with no stated reason (it is the default from SDK 53).
- `[major]` Android layout that ignores edge-to-edge insets.
- `[minor]` Several `Platform.OS` branches in one component — split by file extension instead.
- **Do not comment on missing `useMemo`/`useCallback`** unless `experiments.reactCompiler` is off, or
  the identity is an integration contract with a non-React system, or the code is inside `try/catch`
  (which the compiler cannot analyse).
- **Do not assert a performance problem without a measurement.** Measure → change → re-measure.
- Tests: prefer `userEvent` over `fireEvent`; whole-screen tests over shallow component tests; import
  through the public surface, never internals.
