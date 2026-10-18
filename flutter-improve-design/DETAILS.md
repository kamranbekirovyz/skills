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

**Why:** users who don't have muscle memory to look for actions behind items by swiping left and right, never find out those actions exist. Programmatically open and close those actions for the first time user opens that page to teach them the gesture.

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

---

## Add haptic feedback to key moments

**Link:** https://flutterpro.design/details/md/haptic-feedback

**Why:** the app feels flat when taps and results happen in silence. A subtle haptic vibration on a tab switch, a successful submit or an error makes the app feel responsive in the hand.

**Detect:** a moment that deserves a haptic and doesn't have one: a tab bar item tapped, a form submitted successfully (login, register, payment), an error shown in a snackbar, a toggle flipped, a picker scrolled. The app using haptics somewhere doesn't count as handled; one key moment without a haptic is enough to count the rule as not adhered.

**Hunt:** grep `HapticFeedback|haptic_feedback|Haptics\.|vibrate` to map what's covered. Then find a moment without one: grep `BottomNavigationBar|NavigationBar|TabBar` for tab taps, `SnackBar|showSnackBar` for success/error results, `Switch|Checkbox|onLongPress` for toggles and presses. Stop at the first one with no haptic call on its path; that's a match.

---

## Use tabular figures for changing numbers

**Link:** https://flutterpro.design/details/md/tabular-figures

**Why:** digits have different widths in most fonts, so a timer or counter jumps around as it changes. Tabular figures make every digit the same width: numbers stay still and line up.

**Detect:** a number that changes (timer, counter, price) or lines up with others (table, totals) rendered without `FontFeature.tabularFigures()`. Numbers sitting in a sentence don't need it.

**Hunt:** grep `tabularFigures` first; if used where it matters, nothing to flag. Otherwise grep `Timer|Stopwatch|Duration|countdown|price|total|amount` near `Text(`; a changing or aligned number without the feature is a match.

---

## Give Flutter web links a preview card

**Link:** https://flutterpro.design/details/md/flutter-web-og-image

**Why:** shared in WhatsApp, Slack or X, the app's link shows as a bare URL. It should show a proper preview card with a title, description and image.

**Detect:** the app has a `web/` folder and `web/index.html` has no Open Graph tags. `og:image` must be an absolute URL, and `twitter:image` must be set too since X reads its own tags.

**Hunt:** check `web/index.html` exists; if not, the rule doesn't apply. If it does, grep it for `og:title|og:image|twitter:card`; nothing found means the app's missing it.

---

## Dismiss the keyboard when the user scrolls

**Link:** https://flutterpro.design/details/md/dismiss-keyboard-on-scroll

**Why:** the user finishes typing and scrolls to see the rest, but the keyboard stays covering half the screen. Scrolling means they're done with the field, so close it for them.

**Detect:** a scrollable with a text field inside that doesn't set `keyboardDismissBehavior` to `onDrag`. One such scrollable is enough to count the rule as not adhered. Chat screens are the exception: there the user scrolls while still typing, so leave those on `manual`.

**Hunt:** grep `TextField|TextFormField|CupertinoTextField` and check if the enclosing `ListView|SingleChildScrollView|CustomScrollView|GridView` sets `keyboardDismissBehavior`. Stop at the first scrollable-with-field that doesn't; that's a match.

---

## Scroll to top when the current bottom nav item is tapped again

**Link:** https://flutterpro.design/details/md/bottom-nav-reselect

**Why:** tapping the bottom nav bar item you're already on should scroll that page to the top. It's muscle memory for native app users, so they'll expect it from your apps too.

**Detect:** the app has a bottom nav bar and tapping the already-selected tab does nothing. Look at where the tab tap is handled (the nav bar's `onTap`, or the state/cubit behind it): no reselect branch means not adhered.

**Hunt:** grep `BottomNavigationBar|NavigationBar|CupertinoTabBar`; no bar means the rule doesn't apply. If there's one, check its tap handler for a same-index branch (like `index == currentIndex`) that scrolls to top. Missing means the app's missing it.

---

## Autofocus the field when the page has only one

**Link:** https://flutterpro.design/details/md/autofocus-single-field

**Why:** a page that exists to collect one input (OTP, phone number, change email) makes the user tap the field first before they can type. Focus it on open: the keyboard is up, they type and move on.

**Detect:** a page whose only interactive element is one text field, and the field doesn't set `autofocus: true`. One such page is enough to count the rule as not adhered. Pages with other choices next to the field (like login with social buttons) are correct without it, so don't flag them.

**Hunt:** the OTP page is the clearest case: grep `Pinput|pin_code|otp|Otp|OTP|smsCode|verification` and check `autofocus`. Then look up other single-field pages by name: grep files like `change_email|change_phone|edit_name|forgot_password` and check their field. Stop at the first one without `autofocus`; that's a match.

---

## Show the app version in settings

**Link:** https://flutterpro.design/details/md/show-app-version

**Why:** when a user reports a bug, the first question is which version they're on, and the app has no place to answer it.

**Detect:** the app has nothing that displays the app version with its build number, and if the app uses Shorebird, the patch number too. Missing any of these counts as not adhered.

**Hunt:** grep `PackageInfo|package_info_plus|buildNumber`; a version read and displayed on a page means it's handled. Nothing found means the app's missing it. If `shorebird` is in pubspec, the display must also include `readCurrentPatch` output; otherwise it's a match.

---

## Don't use mobile page transitions on web and desktop

**Link:** https://flutterpro.design/details/md/web-page-transitions

**Why:** web and desktop are click and open. Mobile slide and zoom transitions between pages look cheap there, so pages should switch with no transition.

**Detect:** the app has a `web/`, `macos/`, `windows/` or `linux/` folder and its page transitions have no branch for those platforms: nothing disables transitions for web and desktop, in `pageTransitionsTheme` or at the router.

**Hunt:** check a `web/`, `macos/`, `windows/` or `linux/` folder exists; if none, the rule doesn't apply. If one does, grep `pageTransitionsTheme|PageTransitionsBuilder|pageBuilder` and look for `kIsWeb` or desktop platform builders around it. No such branch means the app's missing it.

---

## Scroll the tapped tab fully into view

**Link:** https://flutterpro.design/details/md/tab-visibility

**Why:** in a tab bar, when the user taps a tab that's only half visible, it should scroll fully into view.

**Detect:** a custom-built horizontal tab (chips, filters, categories) where selecting a tab doesn't scroll it into view. Material's `TabBar` handles it already, so only custom ones count. 

**Hunt:** grep `TabBar` usages first; those are fine. Then find custom tab bars: a horizontal `ListView|SingleChildScrollView` whose items are tappable and track a selected index. If its selection handler has no `Scrollable.ensureVisible` (or equivalent scroll), that's a match.

---

## Open social media links in their apps

**Link:** https://flutterpro.design/details/md/launch-social-media-apps

**Why:** a social media link should open the app directly if it's installed, not the browser.

**Detect:** a social media link (Instagram, X, WhatsApp and the like) launched without `mode: LaunchMode.externalApplication`. Custom schemes like `instagram://` count as wrong too: they fail when the app isn't installed.

**Hunt:** grep `instagram|twitter|x\.com|facebook|whatsapp|wa\.me|tiktok|linkedin|youtube` near `launchUrl`; no social links means the rule doesn't apply. A launch without `externalApplication`, or via a custom scheme, is a match.

---

## Show scrollbars on vertical scrollables

**Link:** https://flutterpro.design/details/md/scrollbars

**Why:** a scrollbar shows the user where they are in the list and how much is left.

**Detect:** a vertically scrollable widget with no scrollbar: neither wrapped in `Scrollbar` nor covered by an app-wide `scrollBehavior` override. One is enough to count the rule as not adhered.

**Hunt:** grep `scrollBehavior|MaterialScrollBehavior` first; it only counts as handled if the override's `buildScrollbar` actually wraps in a `Scrollbar` (overrides exist for other reasons too, like drag devices or removing glow). Otherwise grep `ListView|SingleChildScrollView|CustomScrollView|GridView` and check for a wrapping `Scrollbar`. Stop at the first one without; that's a match.

---

## Make the whole GestureDetector area tappable

**Link:** https://flutterpro.design/details/md/gesture-detector-hit-area

**Why:** by default `GestureDetector` only takes taps on what its child paints, so the padding and the gaps between an icon and a text do nothing. The user taps the row and misses. The whole box should take the tap.

**Detect:** a `GestureDetector` whose `behavior` isn't `opaque` or `translucent` and whose child has unpainted areas: padding, gaps in a `Row`/`Column`, or a `Container` without a color. One is enough to count the rule as not adhered.

**Hunt:** grep `GestureDetector` and check each for `behavior:`. One without `HitTestBehavior.opaque` or `.translucent`, whose child has padding or gaps, is a match.

---

## Format dates for user's locale

**Link:** https://flutterpro.design/details/md/format-date-times

**Why:** `2016-06-24 14:44:00.000` is what `DateTime` prints, and no user should read a date like that. Show `24 July 2016, 14:44` instead, in the user's language.

**Detect:** a date shown to the user without `DateFormat` from `intl`: raw `.toString()`, string interpolation of a `DateTime`, or hand-built patterns like `'$day/$month/$year'`. One is enough to count the rule as not adhered.

**Hunt:** grep `DateFormat` first; a shared helper (like an extension on `DateTime`) used for display means it's handled. Otherwise grep `DateTime` near `Text(` and interpolations of `.day|.month|.year|.hour|.minute`; a date rendered raw is a match.

---

## Keep statusbar tap scrolling to top on iOS

**Link:** https://flutterpro.design/details/md/statusbar-tap-scroll

**Why:** tapping the statusbar scrolls the page to the top; it's native to iOS and users expect it. It works out of the box, but a custom `ScrollController` silently breaks it.

**Detect:** a scrollable with its own `ScrollController` inside a `Scaffold` (or `CupertinoPageScaffold`) that isn't wrapped in a `PrimaryScrollController` providing that controller. Scrollables without a custom controller are fine; Flutter handles those.

**Hunt:** grep `ScrollController(` for custom controllers; none means the rule doesn't apply. For each one attached to a vertical scrollable in a `Scaffold`, check for a `PrimaryScrollController` above it with the same controller. Stop at the first one without; that's a match.

---

## Limit text scaling so layouts don't break

**Link:** https://flutterpro.design/details/md/text-scale-factor

**Why:** some users increase their device's text size for accessibility, and at high scales layouts overflow and break. After the fix, run the app at the capped scale yourself to confirm nothing breaks.

**Detect:** the app doesn't limit `textScaler` app-wide (in `MaterialApp.builder` or equivalent): no clamp and no fixed override like x1.0 or x1.1. Nothing limiting the scale means not adhered.

**Hunt:** grep `textScaler|textScaleFactor`; a clamp or a fixed override applied app-wide means it's handled. Nothing found means the app's missing it.

---

## Fade scrolling content under the status bar

**Link:** https://flutterpro.design/details/md/progressive-fade

**Why:** on pages with no app bar, scrolling content runs behind the status bar and collides with the clock and battery. A progressive fade at the top dissolves it as it slides under.

**Detect:** a `Scaffold` without an `appBar` whose scrollable body has no top fade (like a `ShaderMask` with a top-down gradient). One such page is enough to count the rule as not adhered.

**Hunt:** grep `Scaffold(` and pick ones without `appBar:`; skip pages whose body doesn't scroll. On a scrolling one, check for `ShaderMask|ProgressiveFade` around the body. Stop at the first without; that's a match.

---

## Don't show the iOS date picker on Android

**Link:** https://flutterpro.design/details/md/adaptive-date-picker

**Why:** `CupertinoDatePicker` is iOS-native; Android users don't expect a scroll-wheel picker, and an iOS widget on Android reads as cheap. Show the Material picker on Android and the Cupertino one on iOS.

**Detect:** a `CupertinoDatePicker` shown without a platform check: Android gets the iOS picker instead of something Android-appropriate (usually `showDatePicker`). One is enough to count the rule as not adhered.

**Hunt:** grep `CupertinoDatePicker`; none means the rule doesn't apply. For each, look for a platform branch (`defaultTargetPlatform|Platform.isIOS`) that shows Android something else. No branch means the app's missing it.

---

## Let horizontal lists reach the screen edges

**Link:** https://flutterpro.design/details/md/horizontal-list-padding

**Why:** when the page's scrollable has horizontal padding, horizontal lists inside get clipped at the edges instead of sliding past them. The list should reach the edges and carry its own padding.

**Detect:** a horizontal list nested in a page scrollable that has horizontal padding, so items clip at the padding line instead of the screen edge. One is enough to count the rule as not adhered.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`; none means the rule doesn't apply. For each, check the enclosing page scrollable for horizontal padding around it. Found one means it's a match.

---

## Strip Material tap effects from custom designs

**Link:** https://flutterpro.design/details/md/strip-material-tap-effects

**Why:** Material widgets ripple and glow on tap by default, and in an app with a custom design language those effects clash with it.

**Detect:** first decide what the app is. A Material design app (Material 3 or Expressive, mostly Material widgets) needs its ripples; the rule doesn't apply. An app with its own custom buttons that still uses some Material widgets here and there (a `TextButton`, a `Switch`) should strip their tap effects at both levels: global (`splashFactory: NoSplash.splashFactory`, transparent `splashColor`, `highlightColor`, `hoverColor`, `focusColor`) and each used widget's component theme (`overlayColor`), since M3 widgets draw their press overlay from their own component theme. Only widgets the app actually uses matter.

**Hunt:** judge the design first: mostly Material widgets means the rule doesn't apply. Otherwise in `ThemeData` check the globals (`NoSplash|splashColor|highlightColor|hoverColor|focusColor`) and `overlayColor` in the component themes of the Material widgets in use. A used widget whose tap effect isn't stripped at either level is a match.

---

## Reserve space for images before they load

**Link:** https://flutterpro.design/details/md/reserve-image-space

**Why:** an image widget has no size until its bytes arrive, so when it loads it snaps to its real height and pushes everything below it down. Its box should be sized before the image arrives, so the layout doesn't jump.

**Detect:** a network image with no size decided before load: no `AspectRatio`, no width-and-height box, no parent that fixes its height. `fit:` doesn't count, it only paints inside an already-sized box; width alone doesn't count either, the height still snaps.

**Hunt:** grep `Image.network|CachedNetworkImage|NetworkImage`; none means the rule doesn't apply. For each, check for an `AspectRatio`, an explicit width and height, or a height-bounding parent. Stop at the first without; that's a match.

---

## Update the browser tab title per page

**Link:** https://flutterpro.design/details/md/browser-tab-title

**Why:** the tab title shows in the browser tab, history and bookmarks, and when it's the same on every page, a user with several tabs open can't tell them apart. Each page should say what it is.

**Detect:** the app has a `web/` folder and nothing updates the tab title per page. Handled looks like either `Title` widgets on individual pages, or a wrapper in the app's `builder` that listens to the router (whatever the routing setup: Navigator 1.0/2.0, go_router, auto_route or other) and rebuilds a `Title` on navigation. A static app title alone, or `onGenerateTitle` (it only reruns on app rebuild, not navigation), doesn't count.

**Hunt:** check the `web/` folder exists; if not, the rule doesn't apply. If it does, grep `Title(`; nothing beyond a static app title means the app's missing it.

---

## Unfocus the text field before opening a modal

**Link:** https://flutterpro.design/details/md/unfocus-before-modal

**Why:** opening a modal while a text field is focused brings the keyboard back when the modal closes, even though the user was done typing. Unfocus before opening and it stays away.

**Detect:** a page where a text field can be focused while something somewhere in the page opens a modal (a picker, dropdown, bottom sheet, dialog, anything), and the opening handler doesn't call `FocusManager.instance.primaryFocus?.unfocus()` first. `FocusScope.of(context).unfocus()` counts as the wrong tool: it finds the nearest scope, not the actually focused field.

**Hunt:** grep `showModalBottomSheet|showDialog|showDatePicker|showCupertinoModalPopup` in files that also have `TextField|TextFormField`; no overlap means the rule doesn't apply. Check each opener for `FocusManager` unfocus before it. Stop at the first without; that's a match.

---

## Show a friendly view when a widget breaks

**Link:** https://flutterpro.design/details/md/friendly-error-view

**Why:** when a widget fails to build, users see an empty grey box in release. Show a friendly "Something went wrong" in the app's own colors instead.

**Detect:** the app doesn't set a custom `ErrorWidget.builder`. Not set means not adhered.

**Hunt:** grep `ErrorWidget.builder`; nothing found means the app's missing it.

---

## Format phone numbers

**Link:** https://flutterpro.design/details/md/format-phone-numbers

**Why:** a phone number shown or typed as one long run of digits is hard to read and easy to mistype. It should be formatted everywhere the user sees one, including while they type to a text field.

**Detect:** a phone number shown or typed unformatted: a page displaying a raw phone value, or a phone text field with no formatting `inputFormatters`. Either one is enough to count the rule as not adhered.

**Hunt:** grep `phone|phoneNumber|msisdn`; nothing means the rule doesn't apply. Where a phone value hits a `Text(` or a `TextField`, check for a mask or formatter (`MaskTextInputFormatter|inputFormatters|maskPhoneNumber|phone_numbers_parser`). Stop at the first raw one; that's a match.

---

## Give every text field the right keyboard action

**Link:** https://flutterpro.design/details/md/text-input-action

**Why:** users should be able to fill a form and submit it with the keyboard's action key alone: it moves them to the next field, and on the last one, submits. No tapping each field by hand.

**Detect:** a page with two or more single-line text fields where fields don't set `textInputAction`: the non-last ones `.next`, the last one `.done` (or `.send`/`.go`/`.search` when that's the form's action) with a submit in `onFieldSubmitted`. Multiline fields are exempt: their return key should insert a new line, which is the default. One form without this is enough to count the rule as not adhered.

**Hunt:** find files with two or more `TextField|TextFormField`; none means the rule doesn't apply. In each, check for `textInputAction`. A multi-field form without it is a match.

---

## Make bottom sheets smooth and draggable

**Link:** https://flutterpro.design/details/md/smooth-draggable-bottom-sheets

**Why:** Material's `showModalBottomSheet` opens and closes with a mechanical, soulless motion. Bottom sheets should respond to drags smoothly.

**Detect:** the app opens small non-scrolling bottom sheets with `showModalBottomSheet`. Full-screen scrollable sheets are out of scope.

**Hunt:** grep Material's `showModalBottomSheet`; nothing means the rule doesn't apply. Found means the app's missing it.

---

## Keep the dial code from doubling in phone fields

**Link:** https://flutterpro.design/details/md/autofill-dial-code

**Why:** the phone number field has dial code (+XXX) as a static prefix, but paste and autofill insert the full number with the dial code again. The user ends up with the code twice and has to fix it by hand.

**Detect:** a phone field that shows the dial code as a separate leading part (`prefixText`, a prefix widget, or a country code picker) with nothing in its `inputFormatters` stripping the dial code from pasted or autofilled input. No such field means the rule doesn't apply.

**Hunt:** grep `TextInputType.phone|AutofillHints.telephoneNumber|prefixText` and find a phone field with a separated dial code; none means the rule doesn't apply. Check its `inputFormatters` for a formatter that strips the dial code; missing is a match.

---

## Give full-screen modals the modern sheet look

**Link:** https://flutterpro.design/details/md/adaptive-sheet-route

**Why:** full-screen modals that open from the bottom either use the iOS 13 sheet style, which looks dated now, or just slide up mechanically. Make them modern and smooth, closing with a pull down from the top.

**Detect:** a full-screen modal opened from the bottom with `CupertinoSheetRoute` or with `showModalBottomSheet(isScrollControlled: true)`. One is enough to count the rule as not adhered. Small non-scrolling sheets belong to the bottom sheet rule, not this one. No full-screen bottom modals means the rule doesn't apply.

**Hunt:** grep `CupertinoSheetRoute|isScrollControlled`; a full-screen modal behind either is a match. Nothing found means the rule doesn't apply.

---

## Don't let amount fields go over the limit

**Link:** https://flutterpro.design/details/md/max-amount-formatter

**Why:** when a text field has an amount limit, letting the user type past it and then showing an error is needless friction. A field that simply never accepts more than the limit is better.

**Detect:** an amount field with a hard maximum (a balance, a limit, an order total) that accepts values past it and complains afterwards, anywhere: a field error, a snackbar, a dialog, or a rejected submit. Blocking the edit at input counts as handled. No amount field with a hard maximum means the rule doesn't apply.

**Hunt:** grep `numberWithOptions|TextInputType.number` for amount fields, then look for a max nearby: any path comparing the value to a balance or limit and reacting with an error (`insufficient|exceeds|too (much|high)|maxAmount|balance`), whether in the field, a snackbar, or a dialog. A field that errors after typing instead of blocking the edit is a match. No capped amount field means the rule doesn't apply.

---

## Report real screen names to analytics

**Link:** https://flutterpro.design/details/md/analytics-screen-names

**Why:** analytics sees a Flutter app as one native view, so every page gets reported as `FlutterViewController`. You can't tell which screens users spend time.

**Detect:** the app reports screens through an analytics navigator observer, and a piece is missing: native automatic screen reporting still on (meaning that nothing disables it on iOS and Android), or routes pushed without names (plain `Navigator.push` with no `RouteSettings` name; named routes and go_router routes carry names already). One missing piece is enough to count the rule as not adhered. No analytics observer means the app doesn't track screens at all, so the rule doesn't apply.

**Hunt:** grep `AnalyticsObserver|navigatorObservers|observers:` for an analytics observer; none means the rule doesn't apply. Then two checks: look in `ios/` and `android/` for whatever disables that SDK's native automatic screen reporting (for Firebase it's `FirebaseAutomaticScreenReportingEnabled` and `automatic_screen_reporting`; other SDKs have their own switch, and some report by session capture instead); grep `Navigator.of|MaterialPageRoute` for pushes without a `RouteSettings` name. The first missing one is a match.

---

## Animate exits faster than entrances

**Link:** https://flutterpro.design/details/md/enter-slow-exit-fast

**Why:** an element that's leaving is already dismissed in the user's head, so every extra frame of its exit is waiting. Entrances can take their time so they get noticed; exits should be faster, and never bounce.

**Detect:** a custom enter and exit animation that runs both directions on the same timing: an `AnimationController` played forward and reversed with no `reverseDuration`, a `CurvedAnimation` with no `reverseCurve`, a custom route with `transitionDuration` and no `reverseTransitionDuration`, or the same spring (or a bouncy one) used for both the entrance and the exit. Flutter's built-in page and dialog transitions and one-way animations (a loading pulse, a progress fill) are out of scope. One is enough to count the rule as not adhered.

**Hunt:** grep `AnimationController(|PageRouteBuilder|transitionDuration|SpringDescription|SpringSimulation|MaterialSpringMotion`; none means the rule doesn't apply. For a controller, check it's also `.reverse()`d and whether it sets `reverseDuration` and its curve a `reverseCurve`; for a route, `reverseTransitionDuration`; for springs, whether the exit uses a faster one without bounce. Stop at the first that shares one timing; that's a match.

---

## Let password managers fill the login form

**Link:** https://flutterpro.design/details/md/password-autofill

**Why:** password managers can fill a login screen with one tap, but only when the app tells the system what each field is. Without it, users who saved their password still copy it from the manager by hand, and sign up gets no generated strong password.

**Detect:** a login, sign up or password reset form where the fields have no `autofillHints`, or have them but aren't wrapped together in one `AutofillGroup`. Also counts: a sign up or reset field using `AutofillHints.password` instead of `AutofillHints.newPassword`, or `AutofillHints.email` on a field without `TextInputType.emailAddress`. One is enough to count the rule as not adhered.

**Hunt:** grep `obscureText` for password fields; none means the rule doesn't apply. In each of those files, check the fields for `autofillHints` and the form for an `AutofillGroup` around them. Stop at the first form missing either; that's a match.

---

## Offer to save the password after a successful sign in

**Link:** https://flutterpro.design/details/md/trigger-password-save

**Why:** after the user types their login, the password manager should offer to save it, so next time it's one tap. Without a save at the right moment it's either never offered, or offered after a failed sign in and saves the wrong password.

**Detect:** a login form where nothing calls `TextInput.finishAutofillContext()` after the sign in succeeds, or whose `AutofillGroup` keeps the default `onDisposeAction` (`commit`), which saves whenever the screen is left, even after a failed sign in. A successful password reset with no `finishAutofillContext()` counts too. One is enough to count the rule as not adhered.

**Hunt:** grep `obscureText` for the login form; none means the rule doesn't apply. Then grep `finishAutofillContext` on the sign in and reset success paths, and `onDisposeAction` on the group. The first one missing is a match.

---

## Let urgent notifications through Focus on iOS

**Link:** https://flutterpro.design/details/md/time-sensitive-notifications

**Why:** a courier at the door or a login code that expires in minutes lands silently in a Focus mode or a scheduled summary, and the user sees it too late. Marked as time sensitive, iOS delivers it right away and keeps it on screen until it's opened.

**Detect:** the app shows notifications on iOS for events that expire in minutes (a courier or ride arriving, a login code, a reservation about to be released, a time that's about to pass) and either doesn't mark them time sensitive (`InterruptionLevel.timeSensitive` for local ones) or marks them without the Time Sensitive Notifications capability (`com.apple.developer.usernotifications.time-sensitive` in the `ios/Runner` entitlements), which makes iOS ignore the level. Push payloads are built on the server, outside this repo, so judge push by the capability alone. "Order shipped", promos and likes aren't urgent; nothing to flag for apps that only send those.

**Hunt:** check `ios/` exists and pubspec has `flutter_local_notifications|firebase_messaging|onesignal`; if not, the rule doesn't apply. Read what the notifications say to find an urgent one; none means the rule doesn't apply. Then grep `interruptionLevel|timeSensitive` in `lib/` and `time-sensitive` in `ios/`. An urgent notification without the level, or the level without the capability, is a match.

---

## Cache network images on disk

**Link:** https://flutterpro.design/details/md/cache-network-images

**Why:** Flutter keeps network images in memory only. Every time the app restarts, every image downloads again: screens fill in slower and users pay for the same data twice. Cache them on disk.

**Detect:** network images loaded with no disk cache: `Image.network`, `NetworkImage`, `ExtendedNetworkImageProvider` without `cache: true`, or `ExtendedImage.network` with `cache: false`. `CachedNetworkImage` and `ExtendedImage.network` with its default cache count as handled. One is enough to count the rule as not adhered.

**Hunt:** grep `Image.network|NetworkImage(|ExtendedNetworkImageProvider|ExtendedImage.network|CachedNetworkImage`; none means the rule doesn't apply. Stop at the first image without a disk cache; that's a match.

---

## Carry the typed number from login to reset password and register

**Link:** https://flutterpro.design/details/md/prefill-from-login

**Why:** users usually open "Forgot password" right after a "Wrong credentials" error, to reset the account they just typed, and make them type it again. A user who taps "Register" from login just realized they don't have an account. Both pages should open with the number already filled in.

**Detect:** a login page with a phone number (or email) field that links to forgot password or registration, where the navigation doesn't pass the typed value along or the next page's field doesn't start with it. One of the two is enough to count the rule as not adhered.

**Hunt:** grep `forgot|Forgot|reset_password|ResetPassword|register|Register|sign_up|SignUp` in the login page; no login page with those links means the rule doesn't apply. Check the push to each for an argument carrying the login field's text, and the target page's controller for an initial `text:` from it. The first missing is a match.

---

## Force an update or freeze the app from the server

**Link:** https://flutterpro.design/details/md/forced-update

**Why:** when a released version has a critical bug, or the backend goes down, there's no way to reach users already on that version. A config on the server with minimum versions and a maintenance flag lets you force an update, show a soft one, or freeze the app with a message, without shipping a new release.

**Detect:** nothing checks a server config for a minimum version or a maintenance mode. If one exists, it still counts as not adhered when the check blocks the app from opening (awaited before the first screen shows), when store URLs are hardcoded in the app, or when one minimum version or flag covers both Android and iOS.

**Hunt:** grep `versionarte|upgrader|minVersion|min_version|minimumVersion|forceUpdate|force_update|forcedUpdate|maintenance|Maintenance`; nothing found means the app's missing it. If found, check where it's awaited, where the store URLs come from, and whether versions are set per platform.

---

## Keep Terms links translatable inside one sentence

**Link:** https://flutterpro.design/details/md/linked-text

**Why:** sign up screens link to Terms of Use and Privacy Policy from inside a sentence, and each language puts those links in a different spot. Glued together from separate pieces in code, the links can't move, and the sentence reads broken in any language with a different word order.

**Detect:** a sentence with tappable links built from separately translated pieces (a `Row`/`Wrap` of `Text`s and buttons, or `TextSpan`s each with their own translation key), instead of one translated string with the links marked inside. One is enough to count the rule as not adhered. An app without localization is out of scope.

**Hunt:** grep `AppLocalizations|easy_localization|\.tr\(|intl_utils|slang` for localization; none means the rule doesn't apply. Then grep `TapGestureRecognizer|Terms|terms|Privacy|privacy` near `TextSpan|Text.rich|RichText|Wrap|Row`. A linked sentence assembled from several translation keys is a match.

---

## Keep the screen awake only where users watch it

**Link:** https://flutterpro.design/details/md/keep-screen-awake

**Why:** on a video player, a call, navigation or a reading page, users look at the screen for minutes without touching it, and it turns off in the middle. Keep it awake there, and only there: a phone that never locks drains the battery and stays open for anyone to see.

**Detect:** a page users watch without touching (video, live call, navigation, compass, reading, workout or cooking timer) with no wakelock, or a wakelock that's turned on app-wide (in `main` or the root widget) or never turned off when the page is left, like a tab in an `IndexedStack` that never disposes. One is enough to count the rule as not adhered.

**Hunt:** grep `wakelock|WakelockPlus` first; if found, check it's enabled per page and disabled when that page is left, not in `main.dart`. If not found, grep `video_player|VideoPlayer|chewie|better_player|agora|livekit|webrtc|jitsi|compass|reader` for a watch-without-touching page. One without a wakelock is a match; no such page means the rule doesn't apply.

---

## Pick photos without a permission prompt

**Link:** https://flutterpro.design/details/md/native-photo-picker

**Why:** asking for the whole photo library just to pick one photo puts users off, and a deny sends them to Settings. The system photo picker gives the app only the photo they pick, with no permission dialog at all.

**Detect:** a photo pick that asks for gallery permission first (`Permission.photos`, `Permission.storage`, `PhotoManager.requestPermissionExtend`), or `image_picker` used without `useAndroidPhotoPicker = true` on Android. Apps whose feature is the whole library (gallery, backup) are out of scope.

**Hunt:** grep `image_picker|ImagePicker|photo_manager|wechat_assets_picker|Permission.photos|Permission.storage`; none means the rule doesn't apply. A permission request on the pick path, or `ImagePicker` with no `useAndroidPhotoPicker` set at startup, is a match.

---

## Remove photo and video permissions the app doesn't need

**Link:** https://raw.githubusercontent.com/kamranbekirovyz/skills/main/flutter-improve-design/articles/remove-media-permissions.md

**Why:** some photo packages quietly add access to the whole photo and video library to the Android app. Google Play only allows that for apps whose core feature is the library, and holds the rest in review. An app that only picks a photo now and then should strip them and use the system photo picker.

**Detect:** the release build declares `READ_MEDIA_IMAGES` or `READ_MEDIA_VIDEO`, in the app's own `AndroidManifest.xml` or merged in from a package, with no `tools:node="remove"` for them, and the app's core feature isn't browsing the whole library. Gallery, backup and photo editing apps are out of scope.

**Hunt:** check `android/` exists; if not, the rule doesn't apply. Grep `READ_MEDIA_IMAGES|READ_MEDIA_VIDEO` in `android/app/src/main/AndroidManifest.xml`: declared without `tools:node="remove"` is a match, and both removed means it's handled. If neither, grep the same in each Android plugin's `android/src/main/AndroidManifest.xml` in the pub cache, at the versions in `pubspec.lock`. A plugin that declares one is a match.

---

## Offer the copied code on fields people paste into

**Link:** https://flutterpro.design/details/md/clipboard-suggestion

**Why:** promo codes, referral codes, card and tracking numbers are almost always copied from somewhere else, and on iOS pasting means a long press, a wait and a tap. Show what's on the clipboard under the field when it's tapped, and one tap fills it in.

**Detect:** a text field for a value users usually paste (promo, coupon, voucher, referral or gift code, tracking number, IBAN, card number, wallet address) with nothing that reads the clipboard and offers it. One is enough to count the rule as not adhered. OTP fields are out of scope; the system fills those from the SMS.

**Hunt:** grep `promo|coupon|voucher|referral|giftCard|gift_card|tracking|iban|IBAN|cardNumber|card_number|wallet` near `TextField|TextFormField`; none means the rule doesn't apply. Check that page for `Clipboard.getData`; missing is a match.

---

## Turn on "Save" only after something changes

**Link:** https://flutterpro.design/details/md/disable-unchanged-save

**Why:** on an edit form, a "Save" that's always on doesn't tell the user whether there's anything to save, and tapping it sends a request that changes nothing. Off until an edit, it lights up when the change counts, and goes back off if the edit is undone.

**Detect:** an edit form prefilled from saved data (profile, address, settings) whose save button is enabled before anything changes, or is disabled by hiding it or fading it with opacity instead of a solid grey, or tracks "was touched" instead of comparing the current values with the initial ones. Login and sign up are out of scope: their button stays enabled. One is enough to count the rule as not adhered.

**Hunt:** grep `TextEditingController(text:|initialValue:` for prefilled forms; none means the rule doesn't apply. Check the save button's `onPressed` for a has-changes condition, and look for `Opacity|AnimatedOpacity|Visibility` around it. The first always-on or faded one is a match.

---

## Ask "Discard changes?" before going back

**Link:** https://flutterpro.design/details/md/discard-changes-prompt

**Why:** one accidental back on an edit form and everything the user typed is gone. Ask before throwing it away, but only when something actually changed.

**Detect:** an edit or compose page (a prefilled edit form, a new post, review or message) that lets the user go back with unsaved changes and no prompt: no `PopScope` (or old `WillPopScope`) guarding it. Also counts: a prompt that shows even when nothing changed, buttons labeled Yes/No/Cancel instead of "Discard changes" and "Keep editing", or a custom back button calling `Navigator.pop` instead of `maybePop`, which skips the guard. Pages that save drafts on their own are fine without a prompt. One is enough to count the rule as not adhered.

**Hunt:** grep `TextEditingController(text:|initialValue:` for edit forms and `maxLines: null|minLines:` for compose fields; none means the rule doesn't apply. Check those pages for `PopScope|WillPopScope`. Stop at the first without; that's a match.

---

## Don't ask for notifications at app open

**Link:** https://flutterpro.design/details/md/notification-permission-priming

**Why:** asked at app open, users have no reason to say yes yet, and the system dialog is a one-shot: after "Don't Allow" on iOS it never shows again. Ask when the benefit is obvious, behind your own screen explaining it, so a "Skip" costs nothing.

**Detect:** the system notification dialog shows with no context: requested from `main`, the splash, the first screen's `initState` or app startup, or requested anywhere without the app's own page or modal explaining the benefit first. `flutter_local_notifications` initialized on iOS with `requestAlertPermission` left at its default `true` counts too, since it prompts at init. One is enough to count the rule as not adhered.

**Hunt:** grep `requestPermission|Permission.notification|requestNotificationsPermission|DarwinInitializationSettings|OneSignal.Notifications`; none means the rule doesn't apply. Trace each request to its caller. One called at startup, or with no explaining screen before it, is a match.
