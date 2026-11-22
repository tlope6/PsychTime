# PsychTime

A behavior-change companion app. When an urge hits, you tap once, your companion
guides you through a two-minute technique, and the app learns which techniques
work best for you.

This is the starter codebase: project structure, theme system and onboarding.

## Run it

This folder holds the Dart code but not the iOS/Android platform folders.
Generate those once, then run:

```bash
cd psychtime_app
flutter create . --project-name psychtime --org com.yourname
flutter pub get
flutter test
flutter run
```

`flutter create .` won't overwrite the files already here.

## Project structure

```
lib/
├── main.dart                     # loads saved settings, starts the app
├── app.dart                      # MaterialApp: theme follows the user's vibe
├── core/                         # app-wide plumbing, no screens
│   ├── prefs/
│   │   ├── user_prefs.dart           # data: companion, vibe, name, onboarded
│   │   ├── prefs_repository.dart     # saves/loads on the device
│   │   └── user_prefs_controller.dart # Riverpod: the single source of truth
│   ├── router/
│   │   └── app_router.dart       # routes + "not onboarded? go to welcome"
│   └── theme/
│       ├── vibe.dart             # the 6 color themes, all colors live here
│       ├── psych_colors.dart     # custom colors as a ThemeExtension
│       └── app_theme.dart        # turns a Vibe into a full ThemeData
├── features/                     # one folder per feature
│   ├── companion/
│   │   ├── companion.dart        # the 6 companions (line art as SVG paths)
│   │   └── companion_avatar.dart # draws a companion + bobbing animation
│   ├── onboarding/
│   │   ├── welcome_screen.dart
│   │   ├── companion_screen.dart # step 1
│   │   ├── vibe_screen.dart      # step 2
│   │   ├── name_screen.dart      # step 3
│   │   └── widgets/
│   │       ├── onboarding_scaffold.dart # shared step layout
│   │       └── step_dots.dart
│   └── home/
│       └── home_screen.dart      # starter home + "make it yours" settings
└── shared/widgets/               # reusable UI pieces
    ├── choice_card.dart
    └── sparkle.dart
test/
└── user_prefs_test.dart
```

Each new feature gets its own folder under `features/`, so the next ones will be:

```
features/check_in/      # urge logging: feeling, place, strength, note
features/intervention/  # urge surf, delay timer, box breathing
features/follow_up/     # "how did it go?" + local notification
features/insights/      # charts and "pattern spotted"
features/feedback/      # in-app feedback form
```

## How the theme system works

1. **`Vibe`** (enum) holds every color for one theme: background, cards,
   borders, text, accent, companion color, warning color.
2. **`buildTheme(vibe)`** turns a Vibe into Material `ThemeData`: buttons,
   inputs and text all styled from those colors.
3. **`PsychColors`** carries the colors Material has no slot for. Because it's
   a `ThemeExtension`, switching vibes animates smoothly.
4. Screens never hardcode colors. They use `Theme.of(context)` or
   `context.psych.deco`, so every screen follows the user's vibe automatically.

**To add a vibe:** add one entry to the `Vibe` enum. That's it.
**To add a companion:** draw it as line art on a 48x48 grid, copy the SVG path's
`d` attribute into a new `Companion` entry.

## How onboarding works

```
Welcome → Companion (1/3) → Vibe (2/3) → Name (3/3) → Home
```

- Every choice saves immediately, so the app recolors live while picking a
  vibe, and nothing is lost if the app closes mid-onboarding.
- Finishing the name step sets `onboarded = true`. The router notices and moves
  to Home on its own, and the user never sees onboarding again.
- The same three screens open from Home in **edit mode** (`?edit=1`): no
  progress dots, "Save" instead of "Continue", and Save goes back to Home.
- Until the user types a name, switching companions switches the suggested name
  too (Jelly → Shelly). After that, their chosen name sticks.

## Design decisions worth talking about

| Decision | Why |
|---|---|
| Riverpod for state | Testable without widgets, and screens rebuild only for the fields they watch |
| Settings stay on the device | Urge data is sensitive. Cloud sync can be added later behind `PrefsRepository` |
| Companions drawn from SVG paths | No image files, recolor with any vibe, easy to add more |
| Respect "reduce motion" | Bobbing and twinkling stop if the system setting is on |
| Theme as one enum | One place to add or audit colors, including contrast checks |