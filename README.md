# 📱 EE5415 Android Programming Test – Cheat Sheets

Personal quick reference for the EE5415 Week 13 programming test (3 hrs, Lab PC, B7701). **All app code is Java.** Blocks labelled `kotlin` are optional; skip them.

> ⚠️ Browse in a web browser only. No cloning or downloading. No AI tools. Type everything yourself.

## 🔎 How to find anything in 5 seconds
1. Press **Ctrl + F** on this page.
2. Type a keyword in English or 中文, e.g. `date 日期`, `save 儲存`, `list 列表`, `menu 選單`, `toast`, `rotate 轉屏`, `guest`, `clear 清除`.
3. Most-used code is **right on this page** in [⚡ Quick code](#-quick-code-type-directly). For full examples, click the link in the [🧭 Feature index](#-feature-index).

| Jump to | |
|---|---|
| [🚨 Exam start](#-exam-start) | first 15 minutes |
| [⚡ Quick code](#-quick-code-type-directly) | 20 snippets, no clicking needed |
| [🧭 Feature index](#-feature-index) | "I need to…" → link |
| [🧩 Templates](#-full-app-templates) | full apps (T1–T7) |
| [🐞 Debug](#-debug-除錯) | crash / error fixes |

---

## 🚨 Exam start
| Step | Go to |
|---|---|
| 1. New project: **File → New → Empty Views Activity**, Java, API 24 → Run once | [00 Checklist](sheets/00-exam-day-checklist.md) |
| 2. Read last year's paper solution (most likely format) | [⭐ T7 Wellness Tracker](templates/T7-wellness-tracker-past-paper.md) |
| 3. Add dependencies (Room, RecyclerView) | [01 Gradle](sheets/01-project-setup-gradle-manifest.md#gradle-module-app-kotlin-dsl-buildgradlekts) |
| 4. Register every new Activity in the Manifest | [01 Manifest](sheets/01-project-setup-gradle-manifest.md#androidmanifestxml) |

---

## ⚡ Quick code (type directly)
Click a title to expand it.

<details>
<summary><b>1. Gradle: Room + RecyclerView</b> · dependency 依賴 build.gradle</summary>

```kotlin
// build.gradle.kts (Module: app) → dependencies { }   then "Sync Now"
implementation("androidx.room:room-runtime:2.6.1")
annotationProcessor("androidx.room:room-compiler:2.6.1")
implementation("androidx.recyclerview:recyclerview:1.3.2")
```
Manifest, inside `<application>`:
```xml
<activity android:name=".HistoryActivity" android:parentActivityName=".MainActivity" />
```
</details>

<details>
<summary><b>2. Toast</b> · message 提示</summary>

```java
Toast.makeText(this, "Saved: " + steps + " steps", Toast.LENGTH_SHORT).show();
```
</details>

<details>
<summary><b>3. Read + validate a number from EditText</b> · input 輸入 validate 驗證 empty 空</summary>

```java
String s = stepsET.getText().toString().trim();
if (s.isEmpty()) { stepsET.setError("Required"); return; }
int steps;
try { steps = Integer.parseInt(s); }
catch (NumberFormatException e) { stepsET.setError("Invalid number"); return; }
if (steps <= 0) { stepsET.setError("Must be > 0"); return; }
```
</details>

<details>
<summary><b>4. RadioGroup selected value</b> · radio 單選 mood 心情</summary>

```java
int id = moodRG.getCheckedRadioButtonId();          // -1 if none
String mood = (id == R.id.normalRB) ? "Normal" : (id == R.id.tiredRB) ? "Tired" : "Energetic";
```
</details>

<details>
<summary><b>5. Spinner selected value</b> · dropdown 下拉</summary>

```java
String level = levelSP.getSelectedItem().toString();
```
XML: `<Spinner android:id="@+id/levelSP" android:entries="@array/levels" .../>`
</details>

<details>
<summary><b>6. DatePicker (no future dates)</b> · date 日期 calendar</summary>

```java
Calendar c = Calendar.getInstance();
DatePickerDialog dlg = new DatePickerDialog(this, (view, y, m, d) -> {
    selectedDate = String.format(Locale.US, "%04d-%02d-%02d", y, m + 1, d);  // month 0-based!
    dateBtn.setText(selectedDate);
}, c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DAY_OF_MONTH));
dlg.getDatePicker().setMaxDate(System.currentTimeMillis());   // today or earlier only
dlg.show();
```
Today as a string: `String.format(Locale.US, "%04d-%02d-%02d", c.get(Calendar.YEAR), c.get(Calendar.MONTH) + 1, c.get(Calendar.DAY_OF_MONTH))`
</details>

<details>
<summary><b>7. Open another screen + pass data / go back</b> · intent 跳頁 putExtra finish back 返回</summary>

```java
Intent i = new Intent(this, HistoryActivity.class);
i.putExtra("GOAL", 8000);
startActivity(i);

// in HistoryActivity
int goal = getIntent().getIntExtra("GOAL", 8000);
findViewById(R.id.backBtn).setOnClickListener(v -> finish());   // back
```
</details>

<details>
<summary><b>8. SharedPreferences save / load with default ("Guest", 8000)</b> · prefs 儲存 nickname goal default 預設</summary>

```java
SharedPreferences p = getSharedPreferences("app_prefs", MODE_PRIVATE);

// SAVE
p.edit().putString("nickname", nickname).putInt("dailyStepGoal", goal).apply();

// LOAD (2nd argument = default when nothing saved)
String nickname = p.getString("nickname", "Guest");
int goal = p.getInt("dailyStepGoal", 8000);

// RESET everything to default
p.edit().clear().apply();
```
</details>

<details>
<summary><b>9. Refresh screen when coming back (after Settings)</b> · onResume 更新 refresh</summary>

```java
@Override
protected void onResume() {
    super.onResume();
    helloTV.setText("Hello, " + getSharedPreferences("app_prefs", MODE_PRIVATE)
            .getString("nickname", "Guest") + "!");
}
```
</details>

<details>
<summary><b>10. Options menu ⋮ (Settings / Clear All)</b> · menu 選單 settings 設定</summary>

`res/menu/main_menu.xml`
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/action_settings" android:title="Settings" />
    <item android:id="@+id/action_clear" android:title="Clear All Data" />
</menu>
```
```java
@Override public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_menu, menu); return true;
}
@Override public boolean onOptionsItemSelected(@NonNull MenuItem item) {
    if (item.getItemId() == R.id.action_settings) { startActivity(new Intent(this, SettingsActivity.class)); return true; }
    if (item.getItemId() == R.id.action_clear) { confirmClear(); return true; }
    return super.onOptionsItemSelected(item);
}
```
Menu not showing? The theme parent must not be `NoActionBar` (`res/values/themes.xml`).
</details>

<details>
<summary><b>11. Confirm dialog / info dialog</b> · AlertDialog 對話框 confirm 確認</summary>

```java
new AlertDialog.Builder(this)                 // androidx.appcompat.app.AlertDialog
    .setTitle("Clear All Data?")
    .setMessage("This cannot be undone.")
    .setPositiveButton("OK", (d, w) -> clearAll())
    .setNegativeButton("Cancel", null)
    .show();
```
</details>

<details>
<summary><b>12. Room: Entity + DAO + Database</b> · database 資料庫 sqlite</summary>

```java
@Entity(tableName = "records")
public class WellnessRecord {
    @PrimaryKey @NonNull public String date;     // one row per date
    public int steps; public String mood; public String note;
    public WellnessRecord(@NonNull String date, int steps, String mood, String note) {
        this.date = date; this.steps = steps; this.mood = mood; this.note = note; }
}

@Dao
public interface RecordDao {
    @Query("SELECT * FROM records WHERE date = :d LIMIT 1") WellnessRecord findByDate(String d);
    @Query("SELECT * FROM records ORDER BY date DESC") List<WellnessRecord> getAll();   // newest first
    @Query("SELECT COUNT(*) FROM records WHERE date BETWEEN :s AND :e AND steps >= :goal")
    int countGoalReached(String s, String e, int goal);
    @Insert(onConflict = OnConflictStrategy.REPLACE) void insert(WellnessRecord r);
    @Update void update(WellnessRecord r);
    @Delete void delete(WellnessRecord r);
    @Query("DELETE FROM records") void deleteAll();
}

@Database(entities = {WellnessRecord.class}, version = 1, exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    public abstract RecordDao recordDao();
    private static volatile AppDatabase INSTANCE;
    public static AppDatabase getInstance(Context ctx) {
        if (INSTANCE == null) { synchronized (AppDatabase.class) { if (INSTANCE == null)
            INSTANCE = Room.databaseBuilder(ctx.getApplicationContext(), AppDatabase.class, "app_db")
                    .fallbackToDestructiveMigration().build(); } }
        return INSTANCE;
    }
}
```
</details>

<details>
<summary><b>13. Room in background thread → update UI</b> · executor handler main thread 背景</summary>

```java
private final ExecutorService io = Executors.newSingleThreadExecutor();
private final Handler main = new Handler(Looper.getMainLooper());

io.execute(() -> {
    List<WellnessRecord> list = dao.getAll();        // background
    main.post(() -> adapter.setData(list));          // UI
});
```
Time shortcut: add `.allowMainThreadQueries()` to the Room builder, then call `dao` directly.
</details>

<details>
<summary><b>14. Insert or merge same date (add steps, replace mood, append note)</b> · merge 合併 update</summary>

```java
io.execute(() -> {
    WellnessRecord old = dao.findByDate(date);
    if (old == null) dao.insert(new WellnessRecord(date, steps, mood, remark));
    else {
        old.steps += steps;
        old.mood = mood;
        if (!remark.isEmpty()) old.note = (old.note == null || old.note.isEmpty()) ? remark : old.note + "; " + remark;
        dao.update(old);
    }
    main.post(() -> Toast.makeText(this, "Saved: " + steps + " steps, mood: " + mood, Toast.LENGTH_SHORT).show());
});
```
</details>

<details>
<summary><b>15. This week Sunday → Saturday</b> · week 週 星期日 range</summary>

```java
Calendar c = Calendar.getInstance();
c.add(Calendar.DAY_OF_MONTH, -(c.get(Calendar.DAY_OF_WEEK) - Calendar.SUNDAY));
String start = format(c);                 // Sunday
c.add(Calendar.DAY_OF_MONTH, 6);
String end = format(c);                   // Saturday
// Monday start: int back = (c.get(Calendar.DAY_OF_WEEK) + 5) % 7;
```
</details>

<details>
<summary><b>16. RecyclerView adapter + row click</b> · list 列表 recycler click 點擊</summary>

```java
public class RecordAdapter extends RecyclerView.Adapter<RecordAdapter.VH> {
    public interface OnItemClick { void onClick(WellnessRecord r); }
    private final List<WellnessRecord> data = new ArrayList<>();
    private final OnItemClick listener;
    public RecordAdapter(OnItemClick l) { listener = l; }
    public void setData(List<WellnessRecord> list) { data.clear(); data.addAll(list); notifyDataSetChanged(); }

    static class VH extends RecyclerView.ViewHolder {
        TextView dateTV, stepsTV;
        VH(View v) { super(v); dateTV = v.findViewById(R.id.dateTV); stepsTV = v.findViewById(R.id.stepsTV); }
    }
    @NonNull @Override public VH onCreateViewHolder(@NonNull ViewGroup p, int t) {
        return new VH(LayoutInflater.from(p.getContext()).inflate(R.layout.item_record, p, false));
    }
    @Override public void onBindViewHolder(@NonNull VH h, int pos) {
        WellnessRecord r = data.get(pos);
        h.dateTV.setText(r.date);
        h.stepsTV.setText(r.steps + " steps");
        h.itemView.setOnClickListener(v -> listener.onClick(r));
    }
    @Override public int getItemCount() { return data.size(); }
}
```
Activity:
```java
RecyclerView rv = findViewById(R.id.recordRV);
rv.setLayoutManager(new LinearLayoutManager(this));
adapter = new RecordAdapter(r -> new AlertDialog.Builder(this).setTitle(r.date)
        .setMessage(r.note == null || r.note.isEmpty() ? "No remark" : r.note)
        .setPositiveButton("OK", null).show());
rv.setAdapter(adapter);
```
Item layout root height must be `wrap_content`.
</details>

<details>
<summary><b>17. Keep data on rotation</b> · rotate 轉屏 onSaveInstanceState</summary>

```java
@Override protected void onSaveInstanceState(@NonNull Bundle out) {
    super.onSaveInstanceState(out);
    out.putInt("COUNT", count);
}
// onCreate: if (savedInstanceState != null) count = savedInstanceState.getInt("COUNT", 0);
```
</details>

<details>
<summary><b>18. ViewModel + LiveData (MVVM)</b> · mvvm viewmodel livedata observe</summary>

```java
public class CounterViewModel extends ViewModel {
    private final MutableLiveData<Integer> count = new MutableLiveData<>(0);
    public LiveData<Integer> getCount() { return count; }
    public void add(int d) { count.setValue(Math.max(0, count.getValue() + d)); }
}
// Activity
CounterViewModel vm = new ViewModelProvider(this).get(CounterViewModel.class);
vm.getCount().observe(this, c -> countTV.setText(String.valueOf(c)));
plusBtn.setOnClickListener(v -> vm.add(1));
```
</details>

<details>
<summary><b>19. Timer every second</b> · timer 計時 stopwatch postDelayed</summary>

```java
private final Handler handler = new Handler(Looper.getMainLooper());
private final Runnable tick = new Runnable() { @Override public void run() {
    seconds++; timeTV.setText(String.valueOf(seconds)); handler.postDelayed(this, 1000); } };
// start: handler.postDelayed(tick, 1000);   stop: handler.removeCallbacks(tick);
```
</details>

<details>
<summary><b>20. Change UI: visibility / colour / progress / animation</b> · hide 隱藏 color 顏色 progress animate</summary>

```java
view.setVisibility(View.GONE);                 // VISIBLE / INVISIBLE / GONE
tv.setTextColor(Color.RED);
progressBar.setProgress(Math.min(100, steps * 100 / goal));
img.animate().scaleX(1.5f).scaleY(1.5f).setDuration(500);
```
</details>

---

## 🧭 Feature index
Each row: **I need to…** → link → keywords (for Ctrl+F)

### 🎨 Layout 版面
| I need to… | Go to | Keywords |
|---|---|---|
| Vertical form, buttons side by side | [02 LinearLayout](sheets/02-layouts-widgets.md#linearlayout-fastest-to-write) | linear weight 排版 並排 |
| ConstraintLayout | [02](sheets/02-layouts-widgets.md#constraintlayout) | constraint 約束 |
| Label + input table | [02 TableLayout](sheets/02-layouts-widgets.md#tablelayout) | table 表格 |
| 3×3 grid of buttons | [T3](templates/T3-tictactoe.md) | grid 格仔 board |
| **Rearrange / change layout** | [14 Recipes](sheets/14-screen-gallery.md#12-layout-change-recipes) | 改layout centre 置中 bottom 底部 spacer |
| **Landscape** | [02](sheets/02-layouts-widgets.md#landscape-layout) · [T5](templates/T5-counter-app.md#reslayout-landactivity_mainxml-same-ids) | landscape 橫屏 layout-land |
| See pictures of layouts | [14 Gallery](sheets/14-screen-gallery.md) | picture 圖 |
| size / padding / margin / gravity | [02 Size rules](sheets/02-layouts-widgets.md#size-rules) | match_parent wrap_content 間距 |
| Rounded / circle background | [02 Shape](sheets/02-layouts-widgets.md#shape-background) | shape 圓角 circle |

### ✍️ Input & widgets 輸入
| I need to… | Go to | Keywords |
|---|---|---|
| Read + validate EditText | [⚡3](#-quick-code-type-directly) · [13](sheets/13-errors-troubleshooting.md#input-validation-template) | input validate 驗證 |
| RadioGroup / CheckBox / Switch | [02](sheets/02-layouts-widgets.md#radiogroup--radiobutton) | radio 單選 checkbox 剔 |
| Spinner | [02](sheets/02-layouts-widgets.md#spinner-dropdown) | spinner 下拉 |
| DatePicker / TimePicker | [02](sheets/02-layouts-widgets.md#datepickerdialog) | date 日期 time 時間 |
| SeekBar / RatingBar / ProgressBar | [02](sheets/02-layouts-widgets.md#ratingbar--seekbar) | seek rating 星 progress 進度 |

### 🔀 Screens & navigation 多畫面
| I need to… | Go to | Keywords |
|---|---|---|
| Open screen + pass data | [03](sheets/03-activity-intent-lifecycle.md#explicit-intent--extras) · [T1](templates/T1-bmi-two-activities.md) | intent 跳頁 extra |
| Return a result | [03](sheets/03-activity-intent-lifecycle.md#get-a-result-back-activityresultlauncher) | result 回傳 |
| Lifecycle / onResume | [03](sheets/03-activity-intent-lifecycle.md#lifecycle-cheat) | lifecycle 生命週期 |
| Rotation keep data | [03](sheets/03-activity-intent-lifecycle.md#save-state-on-rotation) | rotate 轉屏 |
| Web / dial / share | [03](sheets/03-activity-intent-lifecycle.md#implicit-intents) | share 分享 url |
| Fragments | [04](sheets/04-fragments.md) | fragment 碎片 list-detail |

### 💬 Dialog & menu 對話框 / 選單
| I need to… | Go to | Keywords |
|---|---|---|
| Options menu | [03](sheets/03-activity-intent-lifecycle.md#options-menu-) | menu 選單 ⋮ |
| AlertDialog (confirm / list / input) | [03](sheets/03-activity-intent-lifecycle.md#alertdialog) | dialog 彈窗 |
| Settings screen | [T7](templates/T7-wellness-tracker-past-paper.md#4-settings-20-marks) | settings 設定 nickname |
| Clear All Data | [T7](templates/T7-wellness-tracker-past-paper.md#mainactivityjava) | clear 清除 reset |

### 💾 Save data 儲存
| I need to… | Go to | Keywords |
|---|---|---|
| SharedPreferences | [08](sheets/08-storage-prefs-files-room.md#sharedpreferences) · [T7 Prefs](templates/T7-wellness-tracker-past-paper.md#prefsjava) | prefs 儲存 default Guest |
| Room database | [08](sheets/08-storage-prefs-files-room.md#room-java) · [T7](templates/T7-wellness-tracker-past-paper.md#1-data-layer) | room 資料庫 dao entity |
| Merge record on same date | [T7 saveRecord](templates/T7-wellness-tracker-past-paper.md#mainactivityjava) | merge 合併 |
| Count / sum / date range query | [08 DAO](sheets/08-storage-prefs-files-room.md#dao) | count sum between 統計 |
| Today / this week dates | [T7 DateUtil](templates/T7-wellness-tracker-past-paper.md#dateutiljava) | week 週 today 今日 |
| List saved as JSON in prefs | [T5](templates/T5-counter-app.md#b-multi-counter-list-recyclerview) | json |
| Text file | [08](sheets/08-storage-prefs-files-room.md#internal-file) | file 檔案 |

### 📋 Lists 列表
| I need to… | Go to | Keywords |
|---|---|---|
| RecyclerView | [09](sheets/09-recyclerview.md) · [T7](templates/T7-wellness-tracker-past-paper.md#recordadapterjava) | recycler 列表 adapter |
| Row tap → dialog | [09](sheets/09-recyclerview.md#click--dialog-with-details-exam-bonus-pattern) | click 點擊 bonus |
| Long-press delete / edit | [T2](templates/T2-room-recyclerview-crud.md) | delete 刪除 edit 修改 |
| Swipe delete | [09](sheets/09-recyclerview.md#swipe-to-delete) | swipe 滑 |
| Grid / horizontal | [09](sheets/09-recyclerview.md#grid--horizontal) | grid 橫向 |
| Different row types | [09](sheets/09-recyclerview.md#multiple-view-types) | viewtype |

### 🔢 Counter & games 計數 / 遊戲
| I need to… | Go to | Keywords |
|---|---|---|
| +/−/Reset counter | [T5](templates/T5-counter-app.md) | counter 計數 count |
| Counter in MVP/MVVM/MVI | [T6](templates/T6-architecture-counter.md) | architecture |
| Board game + score | [T3](templates/T3-tictactoe.md) | game 遊戲 score 分數 |

### ⏱️ Background & timer 背景 / 計時
| I need to… | Go to | Keywords |
|---|---|---|
| Executor + Handler | [06](sheets/06-threads-concurrency.md#executor--handler-recommended) | thread 線程 background |
| Timer / countdown | [06](sheets/06-threads-concurrency.md#handler-postdelayed-timer) | timer 計時 倒數 |
| AsyncTask | [06](sheets/06-threads-concurrency.md#asynctask-deprecated-but-in-the-slides) | asynctask |
| Progress bar loading | [T4](templates/T4-progress-handler.md) | progress 進度 |

### 🌐 Network 網絡
| I need to… | Go to | Keywords |
|---|---|---|
| HTTP GET + JSON | [07](sheets/07-networking-json.md) | http 網絡 json api |

### 🏛️ Architecture 架構
| I need to… | Go to | Keywords |
|---|---|---|
| MVP / MVVM / MVI explained | [05](sheets/05-architecture-mvvm.md) | mvp mvvm mvi 架構 |
| Convert app to architecture | [05 Recipe](sheets/05-architecture-mvvm.md#7-exam-recipe-convert-any-app) | convert 轉換 |
| Comparison table / Q&A | [05](sheets/05-architecture-mvvm.md#6-comparison) | compare 比較 theory |
| ViewBinding / DataBinding | [10](sheets/10-viewbinding-databinding.md) | binding |

### ✨ Animation 動畫
| I need to… | Go to | Keywords |
|---|---|---|
| animate / tween / Canvas | [11](sheets/11-graphics-animation.md) | animation 動畫 draw 畫 |

### 🐞 Debug 除錯
| Problem | Fix |
|---|---|
| App crashes | [13 Errors](sheets/13-errors-troubleshooting.md), Logcat → `FATAL EXCEPTION` |
| ⋮ menu missing | theme parent without `NoActionBar` → [01](sheets/01-project-setup-gradle-manifest.md#theme-make-the--options-menu-visible) |
| Screen not updated after Settings | load data in `onResume()` ([⚡9](#-quick-code-type-directly)) |
| "Cannot access database on the main thread" | Executor ([⚡13](#-quick-code-type-directly)) or `.allowMainThreadQueries()` |
| `ActivityNotFoundException` | add `<activity>` to the Manifest |
| NPE at `findViewById` | ID missing in this layout (check `layout-land` too) |
| DatePicker month wrong | `month + 1` |

---

## 🧩 Full app templates
| Template | Contains | Use when the question is… |
|---|---|---|
| [⭐ **T7 Wellness Tracker** (past paper)](templates/T7-wellness-tracker-past-paper.md) | Room · Prefs · DatePicker · History · week rate · Menu · Settings · Clear All | any **tracker / record / log** app |
| [T1 BMI](templates/T1-bmi-two-activities.md) | form · validation · 2 screens · prefs · menu | calculator → result |
| [T2 Notes](templates/T2-room-recyclerview-crud.md) | Room · RecyclerView · add/edit/delete | CRUD list |
| [T3 Tic Tac Toe](templates/T3-tictactoe.md) | button grid · game logic · score | board game |
| [T4 Progress & Stopwatch](templates/T4-progress-handler.md) | Executor · Handler · timer | background / timer |
| [T5 Counter](templates/T5-counter-app.md) | +/−/Reset · limit · landscape · multi-counter | counting app |
| [T6 Counter × MVP/MVVM/MVI](templates/T6-architecture-counter.md) | same app 4 ways | "use architecture X" |

## 📚 All sheets
| # | Sheet | # | Sheet |
|---|---|---|---|
| 00 | [Exam-day checklist](sheets/00-exam-day-checklist.md) | 08 | [Storage](sheets/08-storage-prefs-files-room.md) |
| 01 | [Setup, Gradle, Manifest](sheets/01-project-setup-gradle-manifest.md) | 09 | [RecyclerView](sheets/09-recyclerview.md) |
| 02 | [Layouts & widgets](sheets/02-layouts-widgets.md) | 10 | [View/DataBinding](sheets/10-viewbinding-databinding.md) |
| 03 | [Activity, Intent, Dialog, Menu](sheets/03-activity-intent-lifecycle.md) | 11 | [Animation](sheets/11-graphics-animation.md) |
| 04 | [Fragments](sheets/04-fragments.md) | 12 | [Kotlin (skip if Java)](sheets/12-kotlin-syntax.md) |
| 05 | [MVP / MVVM / MVI](sheets/05-architecture-mvvm.md) | 13 | [Errors & fixes](sheets/13-errors-troubleshooting.md) |
| 06 | [Threads & timers](sheets/06-threads-concurrency.md) | 14 | [Screen gallery](sheets/14-screen-gallery.md) |
| 07 | [Networking](sheets/07-networking-json.md) | | |

## 🔗 Official docs
- API reference: https://developer.android.com/reference
- Room: https://developer.android.com/training/data-storage/room
- SharedPreferences: https://developer.android.com/training/data-storage/shared-preferences
- RecyclerView: https://developer.android.com/develop/ui/views/layout/recyclerview
- Menus: https://developer.android.com/develop/ui/views/components/menus
