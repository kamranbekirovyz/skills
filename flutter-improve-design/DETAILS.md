# FlutterPro Design Detail Catalog

The rules behind `flutter-improve-design`. Each rule: a title, the link to its full article on the line below (fetched only at plan time), **Why** (what the user lives with today and what the app gains by fixing it, in plain words, written to be shown to the developer as-is), then **Detect** (what to flag), **Hunt** (how to search for it), and **Gotchas** (edge cases). The catalog detects; the article fixes.

---

## Give the last item room to breathe

https://flutterpro.design/details/md/safe-area-replacement · Noticeable · Effort: S

**Why:** scroll to the end of a list and the last item is either cut off or squeezed right up against the bottom of the phone. Everything works, but the screen looks unfinished, like nobody ever scrolled it on a real phone. Fix it and every list ends the way it should, on every phone.

**Detect:** every vertical scrollable that reaches the bottom of the screen or of a bottom sheet must end with dynamic bottom padding (a `BottomPadding`-style helper reading `MediaQuery.viewPaddingOf(context).bottom`), whether `SafeArea` is present or not. Flag when it's hardcoded (`bottom: 32`), missing, or left to `SafeArea`; `SafeArea` around a scrollable is additionally wrong (it clips scrolled content). Content not reaching the bottom today doesn't matter; one more item and it will. Only the outermost scrollable needs it, not lists nested inside.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView`, then for each check the three places the padding could live: the `padding:` parameter, the last child, a wrapping `Padding`.

**Gotchas:** the padding must keep a minimum on devices with no bottom system bar, so it never collapses to zero. If the app already has its own bottom-padding helper, flag the screens that skip it; don't introduce a second helper.

---

## Make it obvious there's more to scroll

https://flutterpro.design/details/md/shader-mask · Noticeable for horizontal rows and small lists, Subtle for full-screen lists · Effort: S

**Why:** a list that ends in a hard edge looks like it has ended. People don't scroll toward something they can't tell is there, so whatever sits past the edge goes unseen. Fading the edge tells them to keep going, and the content you put there actually gets looked at.

**Detect:** in priority order:

1. **Horizontal scrollables: always flag when there's no edge fade.** No exceptions for a peeking half-item: what peeks on one screen width ends exactly at the edge on another, so the hint can't be trusted.
2. **Small bounded scrollables** (pickers, dropdown-style menus, lists inside sheets): a few visible items with the boundary sitting mid-screen looks like a designed end, not a window onto more. Flag when there's no fade.
3. **Full-screen vertical lists:** users expect screens to scroll, so flag missing bottom fade at Subtle, not higher.

**Hunt:** grep `Axis.horizontal` for rows; grep `ListView|GridView|ListWheelScrollView|CupertinoPicker` inside sheets, dialogs, and fixed-height boxes for the small ones.

**Gotchas:** fade the trailing edge (bottom for vertical, end for horizontal); once users have scrolled, they know where they came from. An always-visible scrollbar already does the job, don't flag those. The fade is alpha-only, so it works unchanged in light and dark mode.

---

## Show users what you shipped

https://flutterpro.design/details/md/in-app-changelog · Subtle · Effort: L

**Why:** features ship, bugs get fixed, and users never hear about it. A short what's-new after an update makes the work visible, and when a change came from someone's feedback, that person sees they were heard.

**Detect:** the app has no changelog surface. Flag as an opportunity, not a defect.

**Hunt:** grep `changelog`, `whats_new`, `last_seen_version`; nothing found means there's none.

---

## Match text selection to your app's colors

https://flutterpro.design/details/md/selection-color · Subtle · Effort: S

**Why:** select a few words to copy them and the highlight is the default one Flutter picked, in an app where every other color is yours. It shows up every time someone copies a code, an address, or an amount. It's one change in the theme and it covers the whole app.

**Detect:** the app's theme doesn't set `textSelectionTheme`, and the app has text the user can select or type into. One text field or one selectable text is enough for the rule to apply. Flag as an opportunity, not a defect.

**Hunt:** grep `textSelectionTheme` first; if it's there, there's nothing to flag. Otherwise grep `TextField|TextFormField|CupertinoTextField|SelectableText|SelectionArea` and stop at the first hit.

**Gotchas:** the highlight sits behind the text, so a dark brand color needs opacity until the text still reads cleanly. The drag handles and the caret ride along in the same theme; they're small enough to stay at full color. If the app ships separate light and dark themes, both need it.

---

## Load network images smoothly

https://flutterpro.design/details/md/smooth-image-loading · Noticeable · Effort: S

**Why:** the screen shows up first and the pictures snap in one by one a moment later, pushing the layout around as they land. It makes the app look like it's still putting itself together while the user watches. Show a plain grey box until each picture is ready and fade it in, and the same screen feels calm.

**Detect:** every image loaded over the network that appears with no transition. Flag when there's no placeholder, when it snaps in instead of fading, or when the failure case is a spinner, a broken-image glyph, or an error message the user can read. Assets are out of scope; they're bundled and fast enough to appear instantly. If the app renders network images through one shared widget, fixing that widget covers the app.

**Hunt:** grep `Image.network|NetworkImage|CachedNetworkImage|ExtendedImage|FadeInImage|image_fade`. `FadeInImage` and `image_fade` mean the app already knows the pattern, so the finding becomes the screens that skip it.

**Gotchas:** the placeholder must be quiet, a flat neutral block or a shimmer; a spinner on every thumbnail is worse than the pop it replaced. Error states stay just as quiet: a small muted icon, never a message about the network. The fix needs a package unless the app already has an equivalent, so the finding says so.

---

## Preload icons so they don't pop in

https://flutterpro.design/details/md/precache-icons · Subtle · Effort: S

**Why:** open the app and everything is there except the icons, which show up a moment later. Nobody thinks about why; they just feel the app is slow to wake up. Load the first screen's icons while the splash is still showing and it opens ready.

**Detect:** the app draws icons from assets, `SvgPicture` most of all, and nothing preloads them at startup. One is enough.

**Hunt:** grep `SvgPicture|Image.asset|AssetImage|Assets\.`; then grep `precacheImage|svg.cache`, and if that's already there, there's nothing to flag.

---

## Show users your swipe actions exist

https://flutterpro.design/details/md/flutter-slidable-controller · Noticeable · Effort: M

**Why:** delete and edit sit behind a swipe, with nothing on screen pointing at them. Users who don't think to swipe never find out those actions exist, so work you already shipped goes unused. Sliding the first row open by itself, once, the first time someone opens the list, teaches the gesture without a tutorial.

**Detect:** a list whose rows have swipe actions, and nothing ever shows them. One list is enough.

**Hunt:** grep `Slidable|Dismissible|SlidableController`; a controller paired with a one-time flag means the hint already exists.
