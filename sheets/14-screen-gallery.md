# 14 · Screen Gallery: What Each Layout Looks Like + How to Change It

[⬅ Index](../README.md)

Orange dashed boxes show the **layout containers**, so you can see which XML block to edit.

## 1. Size
![size](../images/02-size.svg)

## 2. padding vs margin
![padding margin](../images/02-padding-margin.svg)

## 3. gravity vs layout_gravity
![gravity](../images/02-gravity.svg)

## 4. Linear / Table / Grid
![layouts](../images/02-layouts.svg)

## 5. ConstraintLayout
![constraint](../images/02-constraint.svg)

## 6. Widget gallery
![widgets](../images/02-widgets.svg)

## 7. Menu & dialogs
![dialogs](../images/03-dialogs-menu.svg)

## 8. Fragments
![fragments](../images/04-fragments.svg)

## 9. RecyclerView
![recyclerview](../images/09-recyclerview.svg)

## 10. Architecture diagrams
![MVP](../images/05-mvp.svg)
![MVVM](../images/05-mvvm.svg)
![MVI](../images/05-mvi.svg)

## 11. Templates
![BMI](../images/T1-bmi.svg)
![Notes](../images/T2-notes.svg)
![TicTacToe](../images/T3-tictactoe.svg)
![Progress](../images/T4-progress.svg)
![Counter](../images/T5-counter.svg)
![Counter landscape](../images/T5-counter-land.svg)
![Multi counter](../images/T5-multi-counter.svg)
![Wellness](../images/T7-wellness.svg)

---
## 12. Layout change recipes
| Want | Do this |
|---|---|
| Row of buttons → column | inner LinearLayout `orientation="vertical"`, buttons `layout_width="match_parent"` (remove `0dp` + weight) |
| Column → row | `orientation="horizontal"`, children `layout_width="0dp"` + `layout_weight="1"` |
| Centre everything | root `android:gravity="center"` |
| Push a button to the bottom | spacer above it: `<View android:layout_width="match_parent" android:layout_height="0dp" android:layout_weight="1"/>` |
| Push an item to the right in a row | spacer before it: `<View android:layout_width="0dp" android:layout_height="1dp" android:layout_weight="1"/>` |
| One widget takes 2/3 of the width | weights `2` and `1`, both widths `0dp` |
| Space between widgets | `layout_marginTop="16dp"` / `layout_marginStart="8dp"` |
| Hide but keep the space / remove the space | `android:visibility="invisible"` / `"gone"` |
| Landscape = side by side | `layout-land/` copy, root `horizontal`, two children `0dp` + `weight=1` |
| Too tall in landscape | wrap the root in `<ScrollView>` (one child only) |
| Text centred inside its box | `android:gravity="center"` on the TextView |
| View centred in its parent | `android:layout_gravity="center"` (LinearLayout parent) or constrain all 4 sides (ConstraintLayout) |
| Big round number | `200dp` width and height, `gravity="center"`, `background="@drawable/circle"` |
| Change button colour | `app:backgroundTint="#D32F2F"` |
| Grid of buttons | `GridLayout columnCount="2"` or nested horizontal LinearLayouts |
