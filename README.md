# EE5415 Android Programming Test – Cheat Sheets

My personal quick-reference sheets for the EE5415 Week 13 programming test (3 hrs, Lab PC, B7701).
Built from the course lecture slides (L01–L09, Storage, RecyclerView). **Java first, Kotlin second.**

> Exam rules reminder: browse in a web browser only. No cloning or downloading. No AI tools. Type everything yourself.

## 🎯 Start here: last year's paper
[**T7 Wellness Tracker (25/26A)**: full solution, 3-hr plan, test checklist, likely variations](templates/T7-wellness-tracker-past-paper.md)

![Wellness Tracker](images/T7-wellness.svg)

## 🗺️ Index

| # | Sheet | Use it when… |
|---|-------|--------------|
| 00 | [Exam-day checklist & 3-hr plan](sheets/00-exam-day-checklist.md) | Before you start / when stuck |
| 01 | [Project setup, Gradle, Manifest](sheets/01-project-setup-gradle-manifest.md) | Adding dependencies, permissions, activities |
| 02 | [XML layouts & widgets (with pictures)](sheets/02-layouts-widgets.md) | Building any screen |
| 03 | [Activity, Intent, Dialogs, Menu, Lifecycle](sheets/03-activity-intent-lifecycle.md) | Multi-screen apps, passing data |
| 04 | [Fragments](sheets/04-fragments.md) | Static/dynamic fragments, list-detail |
| 05 | [🏛️ **MVP / MVVM / MVI + LiveData & ViewModel**](sheets/05-architecture-mvvm.md) | "Use MVP / MVVM / MVI" requirements |
| 06 | [Threads, Handler, Executor, Coroutines](sheets/06-threads-concurrency.md) | Background work, progress bar, timer |
| 07 | [Networking + JSON](sheets/07-networking-json.md) | HTTP GET, parse JSON |
| 08 | [Storage: SharedPreferences, Files, Room](sheets/08-storage-prefs-files-room.md) | Save/load data |
| 09 | [RecyclerView](sheets/09-recyclerview.md) | Any list |
| 10 | [ViewBinding & DataBinding](sheets/10-viewbinding-databinding.md) | Replace findViewById |
| 11 | [Graphics & Animation](sheets/11-graphics-animation.md) | Shapes, Canvas, animate() |
| 12 | [Kotlin quick syntax](sheets/12-kotlin-syntax.md) | If writing Kotlin |
| 13 | [Common errors & fixes](sheets/13-errors-troubleshooting.md) | Crashes, Gradle errors |
| 14 | [🖼️ Screen gallery + layout change recipes](sheets/14-screen-gallery.md) | See / rearrange a layout |

## 🧩 Full app templates

| Template | Covers |
|----------|--------|
| [T1 BMI app](templates/T1-bmi-two-activities.md) | Form → result screen, prefs, menu |
| [T2 Notes app (Room + RecyclerView CRUD)](templates/T2-room-recyclerview-crud.md) | Database list add/edit/delete |
| [T3 Tic Tac Toe](templates/T3-tictactoe.md) | Grid of buttons + game logic |
| [T4 ProgressBar + Stopwatch](templates/T4-progress-handler.md) | Background thread → UI |
| [T5 Counter app](templates/T5-counter-app.md) | Counting, landscape, multi-counter list |
| [T6 Counter in Plain → MVP → MVVM → MVI](templates/T6-architecture-counter.md) | Same app in every architecture |
| [⭐ **T7 Wellness Tracker (past paper 25/26A)**](templates/T7-wellness-tracker-past-paper.md) | **Room + Prefs + DatePicker + History + Menu + Settings** |

## ⚡ Quick lookup
| Requirement | Go to |
|---|---|
| Room record "insert or merge if same date" | [T7 saveRecord()](templates/T7-wellness-tracker-past-paper.md#mainactivityjava) |
| DatePicker (no future dates) | [T7 showDatePicker()](templates/T7-wellness-tracker-past-paper.md#mainactivityjava) |
| This week (Sun–Sat) range | [T7 DateUtil](templates/T7-wellness-tracker-past-paper.md#dateutiljava) |
| Options menu → Settings / Clear all | [T7](templates/T7-wellness-tracker-past-paper.md#4-settings-20-marks), [03](sheets/03-activity-intent-lifecycle.md) |
| Count clicks / +/− buttons | [T5](templates/T5-counter-app.md) |
| "Implement using MVP / MVVM / MVI" | [05](sheets/05-architecture-mvvm.md) + [T6](templates/T6-architecture-counter.md) |
| Change or rearrange the layout | [14 recipes](sheets/14-screen-gallery.md#12-layout-change-recipes) |
| List of items | [09](sheets/09-recyclerview.md), T2, T7 |
| App crashes | [13](sheets/13-errors-troubleshooting.md) |

## 🔗 Official docs
- API reference: https://developer.android.com/reference
- Room: https://developer.android.com/training/data-storage/room
- Pickers (DatePicker): https://developer.android.com/develop/ui/views/components/pickers
- Menus: https://developer.android.com/develop/ui/views/components/menus
- RecyclerView: https://developer.android.com/develop/ui/views/layout/recyclerview
- ViewModel: https://developer.android.com/topic/libraries/architecture/viewmodel
