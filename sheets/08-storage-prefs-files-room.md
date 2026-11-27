# 08 · Storage: SharedPreferences, Files, Raw, Room

[⬅ Index](../README.md) · Full exam-style example: [T7 Wellness Tracker](../templates/T7-wellness-tracker-past-paper.md)

| Need | Use |
|---|---|
| Small key-value data (nickname, goal, high score, count) | **SharedPreferences** |
| Private text file | `openFileOutput` / `openFileInput` |
| Read-only bundled data | `res/raw` |
| A list of records | **Room** |

---
## SharedPreferences
```java
// WRITE
SharedPreferences pref = getSharedPreferences("app_prefs", MODE_PRIVATE);
pref.edit()
    .putString("nickname", "Daniel")
    .putInt("dailyStepGoal", 10000)
    .putBoolean("dark", true)
    .apply();

// READ (always give a default)
String nick = pref.getString("nickname", "Guest");
int goal = pref.getInt("dailyStepGoal", 8000);

// DELETE / RESET TO DEFAULT
pref.edit().remove("nickname").apply();
pref.edit().clear().apply();          // every key goes back to its default value on the next read
```
Load in `onCreate()` / **`onResume()`** (so changes made in other screens show up), save in `onPause()` or on button click.

### Helper class (avoids typos in keys)
```java
public class Prefs {
    public static final String FILE = "app_prefs";
    public static final String KEY_NICKNAME = "nickname";
    public static final String KEY_GOAL = "dailyStepGoal";
    public static SharedPreferences get(Context c) { return c.getSharedPreferences(FILE, Context.MODE_PRIVATE); }
    public static String nickname(Context c) { return get(c).getString(KEY_NICKNAME, "Guest"); }
    public static int goal(Context c) { return get(c).getInt(KEY_GOAL, 8000); }
}
```

### PreferenceFragmentCompat settings screen (optional)
Dependency: `implementation("androidx.preference:preference:1.2.1")`
```xml
<!-- res/xml/settings.xml -->
<PreferenceScreen xmlns:app="http://schemas.android.com/apk/res-auto">
    <EditTextPreference app:key="nickname" app:title="Nickname" app:useSimpleSummaryProvider="true"/>
    <SwitchPreferenceCompat app:key="notify" app:title="Notifications"/>
</PreferenceScreen>
```
```java
public class SettingsFragment extends PreferenceFragmentCompat {
    @Override public void onCreatePreferences(Bundle b, String rootKey) { setPreferencesFromResource(R.xml.settings, rootKey); }
}
// Activity: getSupportFragmentManager().beginTransaction().replace(android.R.id.content, new SettingsFragment()).commit();
// read: PreferenceManager.getDefaultSharedPreferences(this).getString("nickname", "Guest");
```
⚠️ `EditTextPreference` always stores a **String**. For an int goal, use your own Settings Activity with `putInt` (as in T7), or parse the string.

---
## Internal file
```java
try (FileOutputStream fOut = openFileOutput("textfile.txt", MODE_PRIVATE);   // MODE_APPEND to add
     OutputStreamWriter osw = new OutputStreamWriter(fOut, StandardCharsets.UTF_8)) {
    osw.write(inputET.getText().toString());
} catch (IOException e) { e.printStackTrace(); }

StringBuilder sb = new StringBuilder();
try (BufferedReader r = new BufferedReader(new InputStreamReader(openFileInput("textfile.txt"), StandardCharsets.UTF_8))) {
    String line;
    while ((line = r.readLine()) != null) sb.append(line).append('\n');
} catch (IOException e) { e.printStackTrace(); }
```
## Raw resource
```java
try (BufferedReader br = new BufferedReader(new InputStreamReader(getResources().openRawResource(R.raw.mydata)))) {
    String line;
    while ((line = br.readLine()) != null) { /* ... */ }
} catch (IOException e) { e.printStackTrace(); }
```

---
## Room (Java)
Dependencies (sheet 01): `room-runtime` + `annotationProcessor room-compiler`.

### Entity
```java
@Entity(tableName = "records")
public class Record {
    @PrimaryKey(autoGenerate = true)
    public int id;
    @ColumnInfo(name = "record_date")
    public String date;
    public int steps;
    public Record(String date, int steps) { this.date = date; this.steps = steps; }
}
```
To allow only one row per date, make the date the key instead: `@PrimaryKey @NonNull public String date;` (see T7).

### DAO
```java
@Dao
public interface RecordDao {
    @Query("SELECT * FROM records ORDER BY record_date DESC")
    List<Record> getAll();

    @Query("SELECT * FROM records ORDER BY record_date DESC")
    LiveData<List<Record>> getAllLive();          // auto-updates the UI

    @Query("SELECT * FROM records WHERE record_date = :d LIMIT 1")
    Record findByDate(String d);

    @Query("SELECT COUNT(*) FROM records WHERE record_date BETWEEN :s AND :e AND steps >= :goal")
    int countReached(String s, String e, int goal);

    @Query("SELECT SUM(steps) FROM records WHERE record_date BETWEEN :s AND :e")
    int sumSteps(String s, String e);

    @Insert(onConflict = OnConflictStrategy.REPLACE) void insert(Record r);
    @Update void update(Record r);
    @Delete void delete(Record r);
    @Query("DELETE FROM records") void deleteAll();
}
```

### Database (singleton)
```java
@Database(entities = {Record.class}, version = 1, exportSchema = false)
public abstract class AppDatabase extends RoomDatabase {
    public abstract RecordDao recordDao();
    private static volatile AppDatabase INSTANCE;
    public static AppDatabase getInstance(Context context) {
        if (INSTANCE == null) {
            synchronized (AppDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(), AppDatabase.class, "app_db")
                            .fallbackToDestructiveMigration()
                            // .allowMainThreadQueries()   // exam shortcut only
                            .build();
                }
            }
        }
        return INSTANCE;
    }
}
```

### Use off the main thread
```java
RecordDao dao = AppDatabase.getInstance(this).recordDao();
ExecutorService io = Executors.newSingleThreadExecutor();
Handler main = new Handler(Looper.getMainLooper());

io.execute(() -> {
    dao.insert(new Record("2026-11-28", 8500));
    List<Record> all = dao.getAll();
    main.post(() -> adapter.setData(all));
});
```
Or observe LiveData: `dao.getAllLive().observe(this, list -> adapter.setData(list));`

⚡ `.allowMainThreadQueries()` lets you call the DAO directly. That's OK for an exam; add a comment saying it's for simplicity.

### Room (Kotlin)
```kotlin
@Entity(tableName = "records")
data class Record(@PrimaryKey val date: String, val steps: Int, val mood: String, val note: String)

@Dao interface RecordDao {
    @Query("SELECT * FROM records ORDER BY date DESC") suspend fun getAll(): List<Record>
    @Insert(onConflict = OnConflictStrategy.REPLACE) suspend fun insert(r: Record)
    @Query("DELETE FROM records") suspend fun deleteAll()
}
// lifecycleScope.launch { dao.insert(Record("2026-11-28", 8500, "Normal", "")) }
```

## Dates as `yyyy-MM-dd` strings
They sort correctly as text, so `ORDER BY date DESC` and `BETWEEN` work. Helpers are in [T7 DateUtil](../templates/T7-wellness-tracker-past-paper.md#dateutiljava).
