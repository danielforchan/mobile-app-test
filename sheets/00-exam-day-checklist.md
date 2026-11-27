# 00 · Exam-Day Checklist & 3-Hour Plan

[⬅ Index](../README.md) · Past paper practice: [T7 Wellness Tracker](../templates/T7-wellness-tracker-past-paper.md)

## Before 09:10 (10-min setup)
- [ ] Log in to the Lab PC. Open Android Studio and check the SDK and emulator start.
- [ ] Open Canvas and download **your own** past assignment submissions only.
- [ ] Open this repo in the browser. **Don't clone or download it.**
- [ ] Start the emulator **now**. It takes a few minutes to boot.
- [ ] Phone off and in your bag.

## New project (2 min)
1. **File → New → New Project → Empty Views Activity** (XML + Java/Kotlin, *not* "Empty Activity", which is Compose).
2. Language: **Java** (or Kotlin). Minimum SDK: **API 24** is safe.
3. Build config language: Kotlin DSL (`build.gradle.kts`) is the default. See sheet 01.
4. Click **Run** once *before* coding to make sure the empty app builds.

## Suggested 3-hour time plan
| Time | Task |
|------|------|
| 0:00–0:15 | Read the whole question. List screens, data and buttons. Sketch the UI on paper. |
| 0:15–0:30 | Project + Gradle deps + Manifest (activities, permissions). Build once. |
| 0:30–1:15 | Layout XML for every screen. Build. |
| 1:15–2:15 | Logic: listeners, validation, Intent extras, storage, list. **Run after every feature.** |
| 2:15–2:40 | Extra requirements (menu, settings, landscape, bonus). |
| 2:40–2:55 | Test edge cases: empty input, rotation, back button, reopen app. |
| 2:55–3:00 | Submit as instructed. **Don't leave this to the last minute.** |

## Golden rules
- **Functionality first, UI second** (last year's paper said so).
- **Build and run often.** Fixing one error is easier than fixing 20.
- Validate input: empty `EditText` → `setError` or Toast, don't crash.
- If Gradle sync breaks: undo the last dependency change, or **File → Invalidate Caches**.
- Use `Log.d("TAG", "msg")` and the **Logcat** tab to debug.

## Requirement → which sheet
| Requirement | Sheet |
|---|---|
| Room + SharedPreferences + list + menu + settings | **T7** |
| Input form, buttons, checkboxes, radio buttons | 02 |
| Counter / count clicks / score | T5 |
| Change / rearrange layout, landscape | 14, T5 landscape |
| Second screen showing a result | 03 + T1 |
| Remember last input / high score | 08 SharedPreferences |
| List of items, click an item | 09 + T2 |
| Save a list permanently | 08 Room + T2 / T7 |
| About dialog / menu | 03 |
| Background task with a progress bar | 06 + T4 |
| Fragment list-detail | 04 |
| MVP / MVVM / MVI, ViewModel / LiveData | 05 + T6 |
| Animation | 11 |

## Submission sanity check
- App launches without crashing on a clean install.
- Rotation doesn't lose important data.
- If an APK is required: **Build → Build App Bundle(s)/APK(s) → Build APK(s)** → `app/build/outputs/apk/debug/app-debug.apk`.
