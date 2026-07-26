# FlutterPro Design Detail Catalog

The rules behind `flutter-improve-design`. Each rule: a title, the link to its full article on the line below (fetched only at plan time), a fix in one phrase, then **Detect** (what to flag), **Hunt** (how to search for it), **Why** (what users feel), and **Gotchas** (edge cases). The catalog detects; the article fixes.

---

## Give the last item room to breathe

https://flutterpro.design/details/md/safe-area-replacement · Noticeable · Effort: S
Fix in one phrase: don't clip content at the bottom, let it scroll past the edge, and give the last item breathing room with dynamic bottom padding.

**Detect:** every vertical scrollable that reaches the bottom of the screen or of a bottom sheet must end with dynamic bottom padding (a `BottomPadding`-style helper reading `MediaQuery.viewPaddingOf(context).bottom`), whether `SafeArea` is present or not. Flag when it's hardcoded (`bottom: 32`), missing, or left to `SafeArea`; `SafeArea` around a scrollable is additionally wrong (it clips scrolled content). Content not reaching the bottom today doesn't matter; one more item and it will. Only the outermost scrollable needs it, not lists nested inside.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView`, then for each check the three places the padding could live: the `padding:` parameter, the last child, a wrapping `Padding`.

**Why:** scroll to the end and the last item is either cut off or crammed against the very edge of the phone. Everything works, but the screen feels unfinished, like nobody ever scrolled it on a real device.

**Gotchas:** the padding must keep a minimum on devices with no bottom system bar, so it never collapses to zero. If the app already has its own bottom-padding helper, flag the screens that skip it; don't introduce a second helper.

---

## Make it obvious there's more to scroll

https://flutterpro.design/details/md/shader-mask · Noticeable for horizontal rows and small lists, Subtle for full-screen lists · Effort: S
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

https://flutterpro.design/details/md/in-app-changelog · Subtle · Effort: L
Fix in one phrase: show users what changed after an update, so shipped work gets seen.

**Detect:** the app has no changelog surface. Flag as an opportunity, not a defect.

**Hunt:** grep `changelog`, `whats_new`, `last_seen_version`; nothing found means there's none.

**Why:** features ship, bugs get fixed, and users never hear about it. A changelog makes updates visible, and when a change came from feedback, the user who asked sees they were heard.

---

## Match text selection to your app's colors

https://flutterpro.design/details/md/selection-color · Subtle · Effort: S
Fix in one phrase: highlight selected text in the app's own color instead of Material's tinted default.

**Detect:** the app's theme doesn't set `textSelectionTheme`, and the app has text the user can select or type into. One text field or one selectable text is enough for the rule to apply; the fix is a single change in the theme and covers all of them. Flag as an opportunity, not a defect.

**Hunt:** grep `textSelectionTheme` first; if it's there, there's nothing to flag. Otherwise grep `TextField|TextFormField|CupertinoTextField|SelectableText|SelectionArea` and stop at the first hit.

**Why:** select a phrase to copy it and the highlight is stock Material, in an app whose colors are otherwise entirely yours. It's the one surface most apps never style, and it shows up every time someone copies a code, an address, or an amount.

**Gotchas:** the highlight sits behind the text, so a dark brand color needs opacity until the text still reads cleanly. The drag handles and the caret ride along in the same theme; they're small enough to stay at full color. If the app ships separate light and dark themes, both need it.

---

## Load network images smoothly

https://flutterpro.design/details/md/smooth-image-loading · Noticeable · Effort: S
Fix in one phrase: hold a calm placeholder while a remote image loads, then fade to it, and fail quietly when it doesn't arrive.

**Detect:** every image loaded over the network that appears with no transition. Flag when there's no placeholder, when it snaps in instead of fading, or when the failure case is a spinner, a broken-image glyph, or an error message the user can read. Assets are out of scope; they're bundled and fast enough to appear instantly. If the app renders network images through one shared widget, fixing that widget covers the app.

**Hunt:** grep `Image.network|NetworkImage|CachedNetworkImage|ExtendedImage|FadeInImage|image_fade`. `FadeInImage` and `image_fade` mean the app already knows the pattern, so the finding becomes the screens that skip it.

**Why:** the screen loads, then a beat later the images slam into place one by one and the layout flickers as they land. Nothing is broken, but the app looks like it's assembling itself in front of the user instead of arriving finished.

**Gotchas:** the placeholder must be quiet, a flat neutral block or a shimmer; a spinner on every thumbnail is worse than the pop it replaced. Error states stay just as quiet: a small muted icon, never a message about the network. The fix needs a package unless the app already has an equivalent, so the finding says so.
