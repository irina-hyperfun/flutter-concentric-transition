# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Flutter package for creating concentric transition effects, inspired by Cuberto's Animated Onboarding Screens. The package provides three main components:
- **ConcentricPageView**: A PageView widget with concentric transition animations
- **ConcentricPageRoute**: A custom PageRoute for navigation with concentric transitions
- **ConcentricClipper**: A CustomClipper for creating concentric clipping effects

## Project Structure

```
lib/
├── concentric_transition.dart  # Main export file
├── page_view.dart             # ConcentricPageView widget implementation
├── page_route.dart            # ConcentricPageRoute for navigation
└── clipper.dart               # ConcentricClipper for custom clipping

example/
├── lib/
│   ├── main.dart              # Entry point (launches OnboardingExample)
│   ├── onboarding_example.dart # Full onboarding flow example
│   └── route_example.dart     # Navigation transition example
└── pubspec.yaml               # Example app dependencies

test/
└── flutter_concentric_test.dart # Widget tests
```

## Core Architecture

### ConcentricClipper (lib/clipper.dart)
The foundation of all transitions. Creates circular clipping paths that grow/shrink based on progress (0.0-1.0):
- **Growing phase** (0.0-0.5): Circle expands exponentially from a small radius
- **Shrinking phase** (0.5-1.0): Circle contracts, revealing content behind
- Uses exponential growth (`pow(2, progress)`) for smooth animations
- Key parameters: `progress`, `radius`, `verticalPosition`, `reverse`

### ConcentricPageView (lib/page_view.dart:8-203)
A stateful widget wrapping Flutter's PageView with concentric transitions:
- **State management**: Tracks `_progress`, `_prevPage`, `_prevColor`, `_nextColor`
- **Animation system**: Listens to PageController to update clipper progress
- **Color transitions**: Interpolates between page colors using the clipper
- **Next button**: Positioned at `verticalPosition`, fades during page transitions
- **Scale/Opacity**: Optional transformations on page content during scroll

Key implementation details:
- `_onScroll()` (line 178): Calculates progress and updates colors based on scroll direction
- `_buildClipper()` (line 155): Creates the animated clipping layer
- `_Button` (line 205): Handles next/finish actions with opacity animations

### ConcentricPageRoute (lib/page_route.dart:4-76)
Custom PageRoute using ConcentricClipper for navigation transitions:
- 700ms transition duration
- Non-opaque to show previous route during transition
- Uses `_FadeInPageTransition` wrapper that applies ConcentricClipper to route animation

## Development Commands

### Testing
```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/flutter_concentric_test.dart

# Run tests with coverage
flutter test --coverage
```

### Linting
```bash
# Analyze the package
flutter analyze

# Format code
dart format lib/ test/
```

### Example App
```bash
# Run the example app
cd example
flutter run

# To switch between examples, modify example/lib/main.dart:
# - OnboardingExample() for full onboarding flow
# - RouteExample() for navigation transitions
```

### Package Management
```bash
# Get dependencies
flutter pub get

# Check for outdated dependencies
flutter pub outdated

# Publish (maintainers only)
flutter pub publish --dry-run
```

## Implementation Notes

### ConcentricPageView Usage Patterns
- Always provide at least 2 colors for transitions
- `itemCount`: null for infinite scrolling, set value for fixed pages
- `verticalPosition` (default 0.75): Controls where transition circle appears (0.0-1.0)
- `radius` (default 40.0): Base size of the transition circle
- `scaleFactor` (default 0.3): How much pages scale during transitions
- `opacityFactor` (default 0.0): How much pages fade during transitions

### Next Button Behavior (lib/page_view.dart:224-237)
The built-in next button:
- Automatically positioned at `verticalPosition`
- On last page: calls `onFinish()` callback
- On other pages: animates to next page
- Button opacity controlled by scroll progress (line 254)
- Custom button can be provided via `nextButtonBuilder`

### Bug Fix Context (commit 3010d61)
Fixed issue where next button wouldn't work after user swiped past the limit. The button's page index calculation uses `pageController.page?.round()` and checks against `pagesCount` to determine if on last page.

## Testing Guidelines

- Widget tests use `WidgetTester` from flutter_test
- Always pump frames after building widgets with animations
- Use `skipOffstage: false` when finding widgets that may be off-screen in PageView
- Controller-based tests need sufficient pump durations for animations to complete

## Version Information

- Current version: 1.0.3
- Dart SDK: >=2.12.0 <3.0.0 (null safety enabled)
- Flutter: Uses stable channel features only
- Linting: Uses flutter_lints ^1.0.4
