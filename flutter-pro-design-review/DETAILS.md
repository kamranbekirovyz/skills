# FlutterPro Design Detail Catalog

The rules behind `flutter-pro-design-review`. Each rule: a title, the `slug` on the line below (the machine handle: findings cite it, and the full article lives at `https://flutterpro.design/details/<slug>`), a fix in one phrase, then **Detect** (what to flag), **Hunt** (how to search for it), **Why** (what users feel), and **Gotchas** (edge cases). The catalog detects; the article fixes.

---

## Give the last item room to breathe

`safe-area-replacement` · Noticeable · Effort: S
Fix in one phrase: don't clip content at the bottom, let it scroll past the edge, and give the last item breathing room with dynamic bottom padding.

**Detect:** every vertical scrollable that reaches the bottom of the screen or of a bottom sheet must end with dynamic bottom padding (a `BottomPadding`-style helper reading `MediaQuery.viewPaddingOf(context).bottom`), whether `SafeArea` is present or not. Flag when it's hardcoded (`bottom: 32`), missing, or left to `SafeArea`; `SafeArea` around a scrollable is additionally wrong (it clips scrolled content). Content not reaching the bottom today doesn't matter; one more item and it will. Only the outermost scrollable needs it, not lists nested inside. List every offending scrollable separately, `file:line` each.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView`, then for each check the three places the padding could live: the `padding:` parameter, the last child, a wrapping `Padding`.

**Why:** scroll to the end and the last item is either cut off or crammed against the very edge of the phone. Everything works, but the screen feels unfinished, like nobody ever scrolled it on a real device.

**Gotchas:** the padding must keep a minimum on devices with no bottom system bar, so it never collapses to zero. If the app already has its own bottom-padding helper, flag the screens that skip it; don't introduce a second helper.

---

## Make it obvious there's more to scroll

`shader-mask` · Noticeable for horizontal rows and small lists, Subtle for full-screen lists · Effort: S
Fix in one phrase: fade the edge so it's clear there's more to scroll.

**Detect:** in priority order:

1. **Horizontal scrollables: always flag when there's no edge fade.** No exceptions for a peeking half-item: what peeks on one screen width ends exactly at the edge on another, so the hint can't be trusted.
2. **Small bounded scrollables** (pickers, dropdown-style menus, lists inside sheets): a few visible items with the boundary sitting mid-screen looks like a designed end, not a window onto more. Flag when there's no fade.
3. **Full-screen vertical lists:** users expect screens to scroll, so flag missing bottom fade at Subtle, not higher.

**Hunt:** grep `Axis.horizontal` for rows; grep `ListView|GridView|ListWheelScrollView|CupertinoPicker` inside sheets, dialogs, and fixed-height boxes for the small ones.

**Why:** a hard-cut edge looks like the end. Users don't scroll toward what they can't tell exists, and whatever lives past the edge goes unseen.

**Gotchas:** fade the trailing edge (bottom for vertical, end for horizontal); once users have scrolled, they know where they came from. An always-visible scrollbar already does the job, don't flag those. The fade is alpha-only, so it works unchanged in light and dark mode.

---

## Show users what you shipped

`in-app-changelog` · Subtle · Effort: L
Fix in one phrase: show users what changed after an update, so shipped work gets seen.

**Detect:** the app has no changelog surface. Flag as an opportunity, not a defect.

**Hunt:** grep `changelog`, `whats_new`, `last_seen_version`; nothing found means there's none.

**Why:** features ship, bugs get fixed, and users never hear about it. A changelog makes updates visible, and when a change came from feedback, the user who asked sees they were heard.
