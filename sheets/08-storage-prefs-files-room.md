# 08 · Storage: SharedPreferences, Files, Raw, Room

[⬅ Index](../README.md)

| Need | Use |
|---|---|
| Small key-value (settings, last input, high score, **count**) | **SharedPreferences** |
| Text/blob file private to the app | **Internal file** `openFileOutput` |
| Static bundled read-only data | **res/raw** |
| Structured list of records | **Room (SQLite)** |

---
## SharedPreferences
```java
// WRITE
SharedPreferences pref = getSharedPreferences("BMI", MODE_PRIVATE);
pref.edit()
    .putString("height", h)
    .putInt("highScore", 120)
    .putBoolean("dark", true)
    .apply();          // async. commit() = sync, returns boolean

// READ (always give a default)
SharedPreferences pref = getSharedPreferences("BMI", MODE_PRIVATE);
heightET.setText(pref.getString("height", ""));
int best = pref.getInt("highScore", 0);

// DELETE
pref.edit().remove("highScore").apply();
pref.edit().clear().apply();
```
Single-activity file: `getPreferences(MODE_PRIVATE)`.
Typical places: **save in `onPause()`**, **load in `onCreate()`/`onResume()`**.
File location: `/data/data/<pkg>/shared_prefs/BMI.xml` (View → Tool Windows → **Device Explorer**).

### High-score example
```java
private void checkHighScore(int score) {
    SharedPreferences p = getSharedPreferences("GAME", MODE_PRIVATE);
    int best = p.getInt("best", 0);
    if (score > best) {
        p.edit().putInt("best", score).apply();
        bestTV.setText("Best: " + score);
    }
}
```

### Settings screen with PreferenceFragmentCompat
Dep: `implementation("androidx.preference:preference:1.2.1")`

`res/xml/settings.xml`
```xml
<PreferenceScreen xmlns:app="http://schemas.android.com/apk/res-auto">
    <EditTextPreference app:key="username" app:title="Username" app:useSimpleSummaryProvider="true"/>
    <SwitchPreferenceCompat app:key="send_report" app:title="Send crash report"/>
    <ListPreference app:key="sync_freq" app:title="Sync frequency"
        app:entries="@array/freq_names" app:entryValues="@array/freq_values"
        app:defaultValue="60" app:useSimpleSummaryProvider="true"/>
</PreferenceScreen>
```
```java
public class SettingsFragment extends PreferenceFragmentCompat {
    @Override public void onCreatePreferences(Bundle b, String rootKey) {
        setPreferencesFromResource(R.xml.settings, rootKey);
    }
}
// SettingsActivity.onCreate:
getSupportFragmentManager().beginTransaction()
    .replace(android.R.id.content, new SettingsFragment()).commit();

// Read these values anywhere (default prefs file):
SharedPreferences sp = PreferenceManager.getDefaultSharedPreferences(this); // androidx.preference
String user = sp.getString("username", "");
boolean report = sp.getBoolean("send_report", false);
```

---
## Internal file: write / read
```java
// WRITE (MODE_APPEND to add)
try (FileOutputStream fOut = openFileOutput("textfile.txt", MODE_PRIVATE);
     OutputStreamWriter osw = new OutputStreamWriter(fOut, StandardCharsets.UTF_8)) {
    osw.write(inputET.getText().toString());
    Toast.makeText(this, "Saved", Toast.LENGTH_SHORT).show();
} catch (IOException e) { e.printStackTrace(); }

// READ
StringBuilder sb = new StringBuilder();
try (FileInputStream fIn = openFileInput("textfile.txt");
     BufferedReader reader = new BufferedReader(new InputStreamReader(fIn, StandardCharsets.UTF_8))) {
    String line;
    while ((line = reader.readLine()) != null) sb.append(line).append('\n');
} catch (FileNotFoundException e) {
    Toast.makeText(this, "No file yet", Toast.LENGTH_SHORT).show();
} catch (IOException e) { e.printStackTrace(); }
outputTV.setText(sb.toString());
```
Path: `/data/data/<pkg>/files/textfile.txt`. Other dirs: `getFilesDir()`, `getExternalFilesDir(null)`. Delete with `deleteFile("textfile.txt");`

## Raw resource (read-only): `res/raw/mydata.txt`
```java
try (InputStream is = getResources().openRawResource(R.raw.mydata);
     BufferedReader br = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8))) {
    String line; StringBuilder sb = new StringBuilder();
    while ((line = br.readLine()) != null) sb.append(line).append('\n');
    contentTV.setText(sb.toString());
} catch (IOException e) { e.printStackTrace(); }
```
Resource file names may contain only lowercase letters, digits and `_`.

---
## Room (Java)
Deps (sheet 01): `room-runtime` + `annotationProcessor room-compiler`.

### Entity
```java
@Entity(tableName = "colors")
public class ColorItem {          // don't name it Color (clashes with android.graphics.Color)
    @PrimaryKey(autoGenerate = true)
    public int id;

    @ColumnInfo(name = "hex_color")
    public String hex;

    public String name;

    public ColorItem(String hex, String name) { this.hex = hex; this.name = name; }
}
```
With public fields you don't need getters/setters. If the fields are private, Room needs both getters **and** setters.

### DAO
```java
@Dao
public interface ColorDao {
    @Query("SELECT * FROM colors ORDER BY id DESC")
    List<ColorItem> getAll();

    @Query("SELECT * FROM colors ORDER BY id DESC")
    LiveData<List<ColorItem>> getAllLive();       // auto-updates UI

    @Query("SELECT * FROM colors WHERE name = :name LIMIT 1")
    ColorItem findByName(String name);

    @Query("SELECT COUNT(*) FROM colors")
    int count();

    @Insert  void insert(ColorItem... colors);
    @Update  void update(ColorItem color);
    @Delete  void delete(ColorItem color);

    @Query("DELETE FROM colors")
    void deleteAll();
}
```

### Database (singleton)
```java
@Database(entities = {ColorItem.class}, version = 1, exportSchema = false)
public abstract class ColorDatabase extends RoomDatabase {
    public abstract ColorDao colorDao();

    private static volatile ColorDatabase INSTANCE;

    public static ColorDatabase getInstance(Context context) {
        if (INSTANCE == null) {
            synchronized (ColorDatabase.class) {
                if (INSTANCE == null) {
                    INSTANCE = Room.databaseBuilder(context.getApplicationContext(),
                                    ColorDatabase.class, "color_database")
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

### Use it (off the main thread)
```java
ColorDao dao = ColorDatabase.getInstance(this).colorDao();
ExecutorService io = Executors.newSingleThreadExecutor();
Handler main = new Handler(Looper.getMainLooper());

io.execute(() -> {
    dao.insert(new ColorItem("#FF0000", "red"));
    List<ColorItem> all = dao.getAll();
    main.post(() -> adapter.setData(all));
});
```
Or with LiveData (simplest for lists):
```java
dao.getAllLive().observe(this, list -> adapter.setData(list));
io.execute(() -> dao.insert(new ColorItem(hex, name)));   // observer refreshes automatically
```
⚡ **Time-saver:** `.allowMainThreadQueries()` lets you call the DAO on the UI thread. It's fine for a small exam app; add a comment saying it's for simplicity.

### Room (Kotlin)
```kotlin
@Entity(tableName = "colors")
data class ColorItem(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    @ColumnInfo(name = "hex_color") val hex: String,
    val name: String)

@Dao interface ColorDao {
    @Query("SELECT * FROM colors") fun getAll(): LiveData<List<ColorItem>>
    @Insert suspend fun insert(c: ColorItem)
    @Delete suspend fun delete(c: ColorItem)
}

@Database(entities = [ColorItem::class], version = 1)
abstract class ColorDatabase : RoomDatabase() {
    abstract fun colorDao(): ColorDao
    companion object {
        @Volatile private var INSTANCE: ColorDatabase? = null
        fun getInstance(ctx: Context): ColorDatabase =
            INSTANCE ?: synchronized(this) {
                INSTANCE ?: Room.databaseBuilder(ctx.applicationContext,
                    ColorDatabase::class.java, "color_database")
                    .fallbackToDestructiveMigration().build().also { INSTANCE = it }
            }
    }
}
// lifecycleScope.launch { dao.insert(ColorItem(hex = "#FF0000", name = "red")) }
```

## SQL CRUD reminder
```sql
INSERT INTO colors (hex_color, name) VALUES ('#FF0000', 'red');
SELECT * FROM colors WHERE name LIKE '%re%' ORDER BY name ASC;
UPDATE colors SET hex_color = '#DD0000' WHERE name = 'red';
DELETE FROM colors WHERE name = 'red';
```
