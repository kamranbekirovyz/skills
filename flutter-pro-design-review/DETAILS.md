# FlutterPro Design — Detail Catalog

The rules behind `flutter-pro-design-review`. Each rule is one detail from [FlutterPro.Design](https://flutterpro.design). Findings cite rules by their heading slug. The Fix code here is canonical: copy it into plans and adapt names to the codebase, never invent a different solution.

Each rule reads: **Detect** (what to look for in the code), **Why** (what users feel), **Fix** (the exact pattern).

---

## safe-area-replacement

Typical impact: Noticeable · Effort: S · [article](https://flutterpro.design/details/safe-area-replacement)

**Detect:** a scrollable (`ListView`, `SingleChildScrollView`, `CustomScrollView`) wrapped in `SafeArea`, or a scrollable whose padding has no bottom inset handling (`viewPadding` / `SafeArea` / `MediaQuery` nowhere in sight). Both fail: `SafeArea` clips scrolled content at the bottom edge, no handling makes the last item stick under the system bar.

**Why:** scroll to the end and the last item is either cut off mid-render or pinned under the home indicator with zero breathing room. The screen feels unfinished.

**Fix:** remove `SafeArea` around the scrollable and add dynamic bottom padding instead:

```dart
class BottomPadding extends StatelessWidget {
  const BottomPadding({super.key});

  static double of(BuildContext context, {double minimum = 16}) {
    final double viewPadding = MediaQuery.viewPaddingOf(context).bottom;
    return viewPadding > minimum ? viewPadding : minimum;
  }

  @override
  Widget build(BuildContext context) {
    return SizedBox(height: of(context));
  }
}
```

Use it as the last child of the scrolled `Column`, or as padding:

```dart
ListView(
  padding: EdgeInsets.only(bottom: BottomPadding.of(context)),
)
```

`SafeArea` shrinks the viewport. `BottomPadding` adds breathing room.

---

## shader-mask

Typical impact: Subtle · Effort: S · [article](https://flutterpro.design/details/shader-mask)

**Detect:** a scrollable list that fills its viewport with no scroll affordance: no edge fade, no visible scrollbar, content hard-clipped at the boundary. Common spots: pickers, horizontal chip rows, tall menus.

**Why:** long lists aren't obviously scrollable. Items just sit there and users don't know there's more.

**Fix:** fade the edge with `ShaderMask`, no package needed. Bottom-only usually works best (once they've scrolled, they know there's a top):

```dart
ShaderMask(
  shaderCallback: (Rect bounds) {
    return const LinearGradient(
      begin: Alignment.topCenter,
      end: Alignment.bottomCenter,
      colors: [Colors.white, Colors.white, Colors.white, Colors.transparent],
      stops: [0.0, 0.0, 0.88, 1.0],
    ).createShader(bounds);
  },
  blendMode: BlendMode.dstIn,
  child: /* the scrollable */,
)
```

For horizontal lists use `centerLeft` → `centerRight`. Adjust `stops` for fade size: `0.9` subtle, `0.7` aggressive. The mask is alpha-only, so it works the same in light and dark mode.

---

## in-app-changelog

Typical impact: Subtle · Effort: L · [article](https://flutterpro.design/details/in-app-changelog)

**Detect:** the app ships updates (version in `pubspec.yaml` above 1.0.x, or a store presence) but has no changelog surface: no "what's new" screen, no last-seen-version tracking (`grep` for `changelog`, `last_seen_version`, `whats_new` returns nothing). Flag as an opportunity, not a defect.

**Why:** without a changelog, shipped features and fixed bugs stay in the shadow. And when changes came from user feedback, users get to see they were heard.

**Fix:** a versioned map plus last-seen tracking. Needs `pub_semver`, `package_info_plus`, `shared_preferences` (declare in the plan header):

```dart
const Map<String, List<String>> _changelog = {
  '1.3.0': ['export transactions to csv.'],
  '1.2.0': ['reorder categories with drag.', 'unlimited accounts.'],
};

List<String> unseenChanges({required String lastSeen, required String current}) {
  final Version lastSeenVersion = Version.parse(lastSeen);
  final Version currentVersion = Version.parse(current);

  final List<String> result = [];
  for (final entry in _changelog.entries) {
    final Version entryVersion = Version.parse(entry.key);
    if (entryVersion > lastSeenVersion && entryVersion <= currentVersion) {
      result.addAll(entry.value);
    }
  }
  return result;
}
```

On app open: read the current version, compare with the persisted last-seen version, and show unseen changes in a modal. Persist the current version on dismiss. Don't show on first install: there's nothing to catch up on, save and skip. If the user jumps from `1.2.0` to `1.5.0`, show everything in between. Full state-management example in the article.
