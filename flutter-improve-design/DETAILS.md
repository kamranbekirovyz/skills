# Flutter Design Detail Catalog

The rules behind `flutter-improve-design`. Each rule: a title, then a **Link** line with its full article (the article is fetched only at plan time), **Why** (what the user lives with today and what the app gains by fixing it, in plain words, written to be shown to the developer as-is), then **Detect** (what to flag) and **Hunt** (how to search for it). 

---

## Show what's new after an update

**Link:** https://flutterpro.design/details/md/in-app-changelog

**Why:** after an update, users open the app and nothing tells them what's new. The bug you fixed and the feature they asked for go unnoticed. Show them as a list on first open after an update, and the user who reported that bug or asked for that feature knows you listened.

**Detect:** the app never shows what changed after an update. Absence is the whole signal, so nothing in the code will match. A changelog or what's-new page sitting in settings or a profile does not count as handled, because the user has to go find it. Only a view that comes up on its own, the first time they open a new version, counts as handled.

**Hunt:** grep `changelog|whats_new|whatsNew|release_notes|last_seen_version|version_history`. Nothing found means the app's missing it.

---

## Don't let the system navigation bar cover the bottom of scrollable lists

**Link:** https://flutterpro.design/details/md/safe-area-replacement

**Why:** the last item of scrollable lists gets obscured by the system navigation bar. Leave space as tall as that bar at the end of the list so the last item has a breathing room against the bar.

**Detect:** a vertical scrollable that runs to the bottom of the screen and doesn't end with space as tall as the system navigation bar. `SafeArea` around it doesn't fix it: it shrinks the scrolling area, so items get cut off as they pass the bottom instead of sliding under the bar. A fixed padding like 24 on every side isn't a fix either: the bar's height changes by device.

**Hunt:** grep `ListView|SingleChildScrollView|CustomScrollView|GridView|NestedScrollView` for the scrollables, then `viewPaddingOf|viewPadding.bottom|BottomPadding` for the space and `SafeArea` for the wrong fix. A scrollable that doesn't have a dynamic bottom padding based on device's `MediaQuery` is a match (meaning it needs the fix).

---

## Make horizontal lists feel scrollable

**Link:** https://flutterpro.design/details/md/shader-mask

**Why:** a horizontal scrollable list might not always signal that it's scrollable. Especially when the visible content just fits the view and no item is cut in half. Fading the edge hints users that there's more and they can scroll it.

**Detect:** a horizontal scrollable whose end edge doesn't fade.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`; then grep `ShaderMask|LinearGradient` around it to see if a fade is already there.

---

## Match text selection to your app's colors

**Link:** https://flutterpro.design/details/md/selection-color

**Why:** `MaterialApp` applies default tinted colors for text selection in inputs. Match them to your brand colors instead.

**Detect:** the app has text the user can select or type into, and the theme doesn't set `textSelectionTheme`.

**Hunt:** grep `textSelectionTheme` first; if it's there, there's nothing to flag. Otherwise grep `TextField|TextFormField|CupertinoTextField|EditableText|SelectableText|SelectionArea|SelectableRegion` and stop at the first hit.

---

## Load network images smoothly

**Link:** https://flutterpro.design/details/md/smooth-image-loading

**Why:** images in Flutter load with no transition, no placeholder and no failure state. They just pop in. Make it calmer: Show a plain grey box until each picture is ready and fade it in. If fails show a subtle broken image icon, never a technical message.

**Detect:** images loaded over the network that appears with no transition, no placeholder while loading and no failure state. Assets are out of scope; they're bundled and fast enough to appear instantly.

**Hunt:** grep `Image.network|NetworkImage|CachedNetworkImage|ExtendedImage|FadeInImage|image_fade`. Even if the app uses `FadeInImage` or `image_fade`, it must have subtle placeholder while loading and a subtle error state. Otherwise count as rule not adhered.

---

## Preload images and icons so they don't pop in

**Link:** https://flutterpro.design/details/md/precache-icons

**Why:** Flutter loads images and icons into memory when a widget first asks for them, and decoding takes time. So they are painted a few frames late. Precache them during splash view so they are ready when painted.

**Detect:** the app shows icons and images from assets, mostly using `SvgPicture`,`Image.asset` or `AssetImage`, and nothing preloads them at startup.

**Hunt:** grep `SvgPicture|Image.asset|AssetImage|Assets\.`; then grep `precacheImage|svg.cache`, and if that's already there, and precaching all asset images and icons, only then, there's nothing to flag.

---

## Teach users swiping an item right and left for actions

**Link:** https://flutterpro.design/details/md/flutter-slidable-controller

**Why:** users who don't have muscle memory to look for actions behing items by swiping left and right, never find out those actions exist. Programatically open and close those actions for the first time user opens that page to teach them the gesture.

**Detect:** a list whose rows have swipe actions, and nothing ever shows them.

**Hunt:** grep `Slidable|Dismissible|SlidableController`; a controller paired with a one-time flag means the hint already exists.

---

## Reschedule notifications when the timezone changes

**Link:** https://flutterpro.design/details/md/timezone-change

**Why:** when user flies to another country or the clocks shift for daylight saving, their 9 AM reminder fires at 7 AM. Reschedule notifications on timezone change so reminders keep landing at the time they picked.

**Detect:** the app schedules local notifications (`zonedSchedule` or similar) and nothing compares the timezone name and UTC offset on app open or resume to cancel and reschedule them. There must be a saved name/offset pair, a comparison on resume, and a full reschedule when they differ.

**Hunt:** grep `zonedSchedule|flutter_local_notifications|TZDateTime` for the scheduling; then grep `getLocalTimezone|timeZoneOffset|FlutterTimezone` near a saved comparison (e.g. in `SharedPreferences`) and an `AppLifecycleListener|onResume` hook. Scheduling with no comparison-and-reschedule path is a match.

---

## Never show "null" on screen

**Link:** https://flutterpro.design/details/md/never-show-null

**Why:** when a string field comes back `null` or empty from the API and gets displayed directly, the user sees the word "null" or a blank spot on screen. A bad experience, and something they should never see. Instead, gate those values to show "-" or "N/A".

**Detect:** display text built from API-backed values with no shared guard covering all the cases: `null`, `"null"` as a serialized string, and empty string. Scattered `?? '-'` doesn't count as handled; it only covers `null`. Only a shared helper (like an `orPlaceholder()` extension) covering all cases counts.

**Hunt:** grep `orPlaceholder|isUsable|!= 'null'`; a shared helper used for display text means it's handled. Nothing found means the app's missing it.

---

## Preload Google Fonts so text doesn't swap fonts

**Link:** https://flutterpro.design/details/md/google-fonts-glitch

**Why:** the google_fonts package downloads fonts on first use, so on a fresh install users see text in the default font for a split second before it swaps to yours. Not a good first impression. Preload the fonts during splash and text shows in the right typography from frame one.

**Detect:** the app uses `google_fonts` and nothing awaits `GoogleFonts.pendingFonts` at startup. Preloading must cover the exact weights and styles the app uses; a bare `GoogleFonts.pendingFonts([GoogleFonts.inter()])` only loads weight 400 normal.

**Hunt:** grep `google_fonts` in pubspec; no package means the rule doesn't apply. If present, grep `pendingFonts`; the package without `pendingFonts` means the app's missing it.

---

## Format numbers for user's locale

**Link:** https://flutterpro.design/details/md/format-numbers-for-humans

**Why:** a raw `1234567` is hard to read. Users expect numbers the way their region writes them: `1,234,567` in the US, `1.234.567` in Germany. Every count, price and big number should be displayed that way.

**Detect:** amounts, counts or prices rendered as raw values (`.toString()`, string interpolation) instead of through `NumberFormat` from `intl`.

**Hunt:** grep `NumberFormat|decimalPattern|simpleCurrency|compact\(` first; a shared helper used for display means it's handled. Otherwise grep `\$\{.*(price|amount|total|count|balance)|\.toStringAsFixed|\.toString\(\)` inside `Text(` widgets; numeric values rendered raw are a match.

---

## Show loading progress while Flutter web boots

**Link:** https://flutterpro.design/details/md/flutter-web-loading-progress

**Why:** Flutter web takes a few seconds to boot, and users stare at a blank white page wondering if the site is broken. Show a splash or progress bar instead, so they know the app is coming.

**Detect:** the app has a `web/` folder and `web/index.html` has an empty `<body>` with nothing shown while Flutter boots: no splash, logo or progress indicator.

**Hunt:** check `web/index.html` exists; if not, the rule doesn't apply. If it does, look at its `<body>`: only script tags and no visible markup (or `flutter_bootstrap.js` alone) means the app's missing it.
