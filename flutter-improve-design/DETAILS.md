# FlutterPro Design Detail Catalog

The rules behind `flutter-improve-design`. Each rule: a title, then a **Link** line with its full article (the article is fetched only at plan time), **Why** (what the user lives with today and what the app gains by fixing it, in plain words, written to be shown to the developer as-is), then **Detect** (what to flag) and **Hunt** (how to search for it). 

---

## Show what's new after an update

**Link:** https://flutterpro.design/details/md/in-app-changelog

**Why:** after an update, users open the app and nothing tells them what's new. The bug you fixed and the feature they asked for go unnoticed. A short list on first open shows them, and the user who asked for that feature knows you listened.

**Detect:** the app never shows what changed after an update. Absence is the whole signal, so nothing in the code will match. A changelog or what's-new page sitting in settings or a profile does not count as handled, because the user has to go find it. Only a screen that comes up on its own, the first time they open a new version, counts as handled.

**Hunt:** grep `changelog|whats_new|whatsNew|release_notes|last_seen_version|version_history`. Nothing found means the app has none. If something is found, check it against Detect before clearing the rule.

---

## Don't let the system navigation bar cover the bottom of a scrollable list

**Link:** https://flutterpro.design/details/md/safe-area-replacement

**Why:** the last item of scrollable lists can be obscured by the system navigation bar, and taps go to the bar instead of the item. Leave space as tall as that bar at the end of the list and the last item stays fully visible and tappable.

**Detect:** a vertical scrollable that runs to the bottom of the screen and doesn't end with space as tall as the system navigation bar. `SafeArea` around it doesn't fix it: it shrinks the scrolling area, so items get cut off as they pass the bottom instead of sliding under the bar. A fixed padding like 24 on every side isn't a fix either: the bar's height changes by device.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView` for the scrollables, then `viewPaddingOf|viewPadding.bottom|BottomPadding` for the space and `SafeArea` for the wrong fix. A scrollable that ends without bottom padding read from the device is a match, and so is one wrapped in `SafeArea`. No scrollables at all means the rule doesn't apply.

---

## Make horizontal lists feel scrollable

**Link:** https://flutterpro.design/details/md/shader-mask

**Why:** a horizontal row that ends in a hard edge looks like it has ended. Users don't swipe toward something they can't tell is there, so the rest of the row goes unseen. Fading the edge shows there's more, and the items you put there get looked at.

**Detect:** a horizontal scrollable whose end edge doesn't fade.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`; then grep `ShaderMask|LinearGradient` around it to see if a fade is already there.

---

## Match text selection to your app's colors

**Link:** https://flutterpro.design/details/md/selection-color

**Why:** when a user selects text, in a text field or anywhere on the screen, the highlight is Material's tint color, not the app's.

**Detect:** the app has text the user can select or type into, and the theme doesn't set `textSelectionTheme`.

**Hunt:** grep `textSelectionTheme` first; if it's there, there's nothing to flag. Otherwise grep `TextField|TextFormField|CupertinoTextField|EditableText|SelectableText|SelectionArea|SelectableRegion` and stop at the first hit.

---

## Load network images smoothly

**Link:** https://flutterpro.design/details/md/smooth-image-loading

**Why:** the screen shows up first and the pictures snap in one by one a moment later, pushing the layout around as they land. It makes the app look like it's still putting itself together while the user watches. Show a plain grey box until each picture is ready and fade it in, and the same screen feels calm.

**Detect:** every image loaded over the network that appears with no transition. Flag when there's no placeholder, when it snaps in instead of fading, or when the failure case is a spinner, a broken-image glyph, or an error message the user can read. Assets are out of scope; they're bundled and fast enough to appear instantly. If the app renders network images through one shared widget, fixing that widget covers the app.

**Hunt:** grep `Image.network|NetworkImage|CachedNetworkImage|ExtendedImage|FadeInImage|image_fade`. `FadeInImage` and `image_fade` mean the app already knows the pattern, so the finding becomes the screens that skip it.

**Gotchas:** the placeholder must be quiet, a flat neutral block or a shimmer; a spinner on every thumbnail is worse than the pop it replaced. Error states stay just as quiet: a small muted icon, never a message about the network. The fix needs a package unless the app already has an equivalent, so the finding says so.

---

## Preload icons so they don't pop in

**Link:** https://flutterpro.design/details/md/precache-icons

**Why:** open the app and everything is there except the icons, which show up a moment later. Nobody thinks about why; they just feel the app is slow to wake up. Load the first screen's icons while the splash is still showing and it opens ready.

**Detect:** the app draws icons from assets, `SvgPicture` most of all, and nothing preloads them at startup.

**Hunt:** grep `SvgPicture|Image.asset|AssetImage|Assets\.`; then grep `precacheImage|svg.cache`, and if that's already there, there's nothing to flag.

---

## Show users your swipe actions exist

**Link:** https://flutterpro.design/details/md/flutter-slidable-controller

**Why:** delete and edit sit behind a swipe, with nothing on screen pointing at them. Users who don't think to swipe never find out those actions exist, so work you already shipped goes unused. Sliding the first row open by itself, once, the first time someone opens the list, teaches the gesture without a tutorial.

**Detect:** a list whose rows have swipe actions, and nothing ever shows them.

**Hunt:** grep `Slidable|Dismissible|SlidableController`; a controller paired with a one-time flag means the hint already exists.
