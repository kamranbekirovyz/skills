# FlutterPro Design Detail Catalog

The rules behind `flutter-improve-design`. Each rule: a title, the link to its full article on the line below (fetched only at plan time), **Why** (what the user lives with today and what the app gains by fixing it, in plain words, written to be shown to the developer as-is), then **Detect** (what to flag) and **Hunt** (how to search for it). 

---

## Show changelog after an update

https://flutterpro.design/details/md/in-app-changelog · Subtle · Effort: L

**Why:** users update the app and never find out what changed. Showing a list of what changed the first time they open the new version makes the work visible, and a user who asked for something or reported an issue sees it was done.

**Detect:** the app never shows what changed after an update. Flag as an opportunity, not a defect.

**Hunt:** grep `changelog|whats_new|whatsNew|release_notes|last_seen_version`; nothing found means there's none.

---

## Give the last item room to breathe

https://flutterpro.design/details/md/safe-area-replacement · Noticeable · Effort: S

**Why:** the last item of a list ends up under the phone's bottom bar, half readable. `SafeArea` doesn't fix it, it cuts items off while they scroll. Lists should slide behind the bottom bar and end with space as tall as that bar.

**Detect:** a vertical list that runs to the bottom of the screen and doesn't end with space the size of the device's bottom bar. Almost no app does this.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView` for the list; grep `viewPaddingOf|viewPadding.bottom|BottomPadding` for the space. Nothing found means it's missing.

---

## Make horizontal lists feel scrollable

https://flutterpro.design/details/md/shader-mask · Noticeable · Effort: S

**Why:** a horizontal row that ends in a hard edge looks like it has ended. Users don't swipe toward something they can't tell is there, so the rest of the row goes unseen. Fading the edge shows there's more, and the items you put there get looked at.

**Detect:** a horizontal scrollable whose end edge doesn't fade.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`; then grep `ShaderMask|LinearGradient` around it to see if a fade is already there.

---

## Match text selection to your app's colors

https://flutterpro.design/details/md/selection-color · Subtle · Effort: S

**Why:** when a user selects text, in a text field or anywhere on the screen, the highlight is Material's tint color, not the app's.

**Detect:** the app has text the user can select or type into, and the theme doesn't set `textSelectionTheme`.

**Hunt:** grep `textSelectionTheme` first; if it's there, there's nothing to flag. Otherwise grep `TextField|TextFormField|CupertinoTextField|EditableText|SelectableText|SelectionArea|SelectableRegion` and stop at the first hit.

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

**Detect:** the app draws icons from assets, `SvgPicture` most of all, and nothing preloads them at startup.

**Hunt:** grep `SvgPicture|Image.asset|AssetImage|Assets\.`; then grep `precacheImage|svg.cache`, and if that's already there, there's nothing to flag.

---

## Show users your swipe actions exist

https://flutterpro.design/details/md/flutter-slidable-controller · Noticeable · Effort: M

**Why:** delete and edit sit behind a swipe, with nothing on screen pointing at them. Users who don't think to swipe never find out those actions exist, so work you already shipped goes unused. Sliding the first row open by itself, once, the first time someone opens the list, teaches the gesture without a tutorial.

**Detect:** a list whose rows have swipe actions, and nothing ever shows them.

**Hunt:** grep `Slidable|Dismissible|SlidableController`; a controller paired with a one-time flag means the hint already exists.
