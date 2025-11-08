# Copilot Instructions

## Big Picture
- The entire app lives in `index.html` and renders a Tailwind-styled single-page JLPT kanji trainer with no build tooling; every change should keep the file self-contained and CDN-friendly.
- State is managed via module-level variables (`currentJLPTLevel`, `currentView`, `currentMode`, `selectedLessons`, `studyKanjiList`, etc.); all user interactions end by calling `renderApp()` to refresh the DOM.
- Three study flows share the same data: list browsing, flip-style flashcards, and an automated quiz that cycles cards with timers and progress feedback.

## Data Model
- `KANJI_DATA` is the canonical source: each level (currently `N5`, `N4`) holds ordered `lesson` objects whose `kanji` entries include `kanji`, `meaning`, `onyomi`, `kunyomi`, and `usage` (array of strings).
- `ALL_KANJI` derives flattened lists for features like bookmarks; if you add new levels or alter the structure, update both `KANJI_DATA` and the derived aggregations.
- Bookmarking relies on `localStorage` (`jlptKanjiBookmarks`); ensure new kanji objects keep stable `.kanji` values so saved bookmarks resolve correctly.

## Rendering & Navigation
- `renderApp()` orchestrates view changes via `currentView` (`level-select`, `mode-select`, `lesson-select`, `study`) and injects HTML strings into shared containers; any new views should follow this pattern and reset timers as needed.
- Shared navigation buttons (`prevButton`, `nextButton`) are wired once on `DOMContentLoaded`; mode-specific behavior is handled inside `renderStudyView()` based on `currentMode`.
- Inline `onclick="handle…"` hooks require their handlers to be attached to `window` or otherwise remain in scope after re-renders.

## Mode Behaviors
- Use `compileStudyList()` to rebuild `studyKanjiList`; it shuffles automatically for flashcards and quizzes while preserving order for the list view.
- Flashcards respect `isMeaningHidden` and `isCardFlipped`; when adding card controls, update these flags and re-render instead of manipulating the DOM directly.
- Quiz mode is timer-driven (`startRevealTimer()`, `startReviewTimer()`); reuse `startQuizInterval()` for new countdown phases so pause/resume keeps working.

## List View Patterns
- Column visibility toggles read from `listVisibility`; to add new columns, extend this object, the header toggle markup, and the cell generator helpers in `renderListView()`.
- Cell-level hiding uses `.hidden-cell-content` / `.visible-cell-content` classes and placeholder spans; preserve these conventions so reveal interactions stay consistent.

## Settings & Bookmarks
- Lesson selection UI is generated in `renderLessonSelectPage()`; its mutual exclusivity logic for bookmarks lives in `updateStartButtonCount()` and `handleSelectAll()`—keep these helpers in sync when extending lesson filters.
- Quiz timings are stored in `userSettings` (seconds) and edited through the modal rendered by `renderSettingsModal()`; validate inputs before persisting.

## Styling & UX
- Tailwind is loaded via CDN (`tailwindcss.com`); utility classes must be hard-coded strings so they survive minification and avoid dynamic class name generation.
- Dark mode toggles by swapping container classes and updating the SVG icon; any new components should include both light and dark variants for readability.
- Reuse `showToast()` for short-lived user feedback instead of adding new notification mechanisms.

## Developer Workflow
- No build step is required; open `index.html` directly or serve locally with `python3 -m http.server` (recommended for reliable `localStorage`).
- Manual testing is the norm: exercise each mode (list, flashcard, quiz) after stateful changes to ensure `renderApp()` paths, timers, and bookmarks behave as expected.
- Keep additions ASCII-only unless the dataset explicitly needs kana/kanji samples; existing kanji entries already include necessary Unicode.
