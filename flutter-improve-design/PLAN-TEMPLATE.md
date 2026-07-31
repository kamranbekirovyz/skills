# Plan Template

Every plan written by `flutter-improve-design` follows this structure. The executor may be a less capable model with zero context and zero taste. The plan must contain everything, exactly: no references to "the review above" or "the rule we discussed", and every value spelled out. Never "make it tappable properly"; always the literal code.

```markdown
# Fix: <the symptom in user language, e.g. "Taps on row padding do nothing">

> Follow the steps in order. Run every check. If anything in "STOP if"
> happens, stop and report instead of improvising.

- **Link**: <the rule's article link, verbatim from its **Link** line in DETAILS.md>
- **Needs new dependency**: none | <package name and why>

## Why

2-3 sentences: what users feel, and why it matters. Taken from the
catalog rule, grounded in this app.

## Where

Every affected location, with the current code quoted verbatim:

​```dart
// lib/screens/home_screen.dart:42 — current
GestureDetector(
  onTap: () => _openDetails(item),
  child: Row(...),
)
​```

## The fix

The article's fix, applied to this codebase. The exact end state,
every value spelled out:

​```dart
// target
GestureDetector(
  behavior: HitTestBehavior.opaque,
  onTap: () => _openDetails(item),
  child: Row(...),
)
​```

Match how this codebase already writes widgets; imitate <one exemplar
file that does it right, if one exists>.

## Steps

1. <One concrete edit per step: file, what changes, resulting code.>
2. …

## Check it

`dart analyze` exits clean. <Plus greps that prove the steps happened,
each scoped to one file the steps changed, never `lib` or the whole app:
the old pattern gone from that file, the new pattern present in it, e.g.
`grep -c "SmoothImage" lib/features/home/widgets/filter_tabs.dart` → 1.
A check must only be able to fail when a step wasn't done; if it could
fail for any other reason, rewrite or drop it.>

## Don't touch

- <files/widgets that look related but are out of scope, and why>
- No new dependencies unless declared in the header.
- No refactors, renames, or cleanups beyond the fix.

## STOP if

- The code at any location in "Where" doesn't match the quoted excerpt
  (the codebase moved on since this plan was written).
- The fix seems to require touching something in "Don't touch".
- A check fails twice.

## When you're done

Tell the developer, in two or three sentences, what's different in the app
now and one place to open to see it. Not a list of files, the steps already
have those.
```

## Notes for the plan author

- The fix comes from the rule's article (`curl -s <the URL on the rule's **Link** line in DETAILS.md>`), never from memory. Copy the pattern, then adapt names and types to this codebase. Carry that same link into the plan's **Link** header, so a session that hits an edge case the plan didn't cover can read the source instead of improvising.
- Quote current code from your own reads during this session, at the moment of writing the plan.
