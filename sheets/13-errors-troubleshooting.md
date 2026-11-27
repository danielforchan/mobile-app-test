# 13 · Common Errors & Fixes

[⬅ Index](../README.md)

Read **Logcat** (bottom panel): filter by your package, then find `FATAL EXCEPTION` and the first line that names **your** class.

| Error / Symptom | Cause | Fix |
|---|---|---|
| `ActivityNotFoundException` | Activity not in manifest | Add `<activity android:name=".XActivity"/>` |
| `NullPointerException` on `findViewById(...)` | Wrong ID / view in another layout / before `setContentView` | Check `R.id` matches the XML **in this layout (and layout-land!)** |
| NPE at `findViewById(R.id.main)` insets code | Removed `android:id="@+id/main"` from root | Re-add the id or delete the EdgeToEdge/insets block |
| Crash only after rotation | `layout-land` is missing an ID used in Java | Same IDs in both layout files |
| `NumberFormatException` | `parseInt("")` | Check `isEmpty()` first + try/catch |
| `CalledFromWrongThreadException` | UI update from background thread | `runOnUiThread` / `handler.post` |
| `NetworkOnMainThreadException` | Network on UI thread | Executor / Thread / coroutine |
| `SecurityException ... INTERNET` | No permission | Add `<uses-permission android:name="android.permission.INTERNET"/>` |
| `Cleartext HTTP traffic not permitted` | `http://` URL | `android:usesCleartextTraffic="true"` or use https |
| `Cannot access database on the main thread` | Room on UI thread | Executor, LiveData, or `.allowMainThreadQueries()` |
| Room: `Cannot find getter for field` | private fields without getters/setters | Make fields `public` or add both |
| Room: `cannot find implementation for ...Database` | Missing compiler dep | Add `room-compiler` with `annotationProcessor`/`kapt` |
| `You need to use a Theme.AppCompat theme` | AppCompatActivity with wrong theme | Keep the template theme |
| `Could not find method onX(View)` | `android:onClick` method missing / private | `public void onX(View v)` |
| `ClassCastException` | ID points to another type | Fix ID/type |
| RecyclerView shows nothing | No LayoutManager / adapter / empty list / height 0 | `setLayoutManager`, `setAdapter`, `notify...` |
| Each RecyclerView row is full screen | item root `match_parent` height | `wrap_content` |
| `IndexOutOfBoundsException` in adapter | list changed without notify | `notifyItemRemoved/Inserted` |
| Data lost on rotation | Activity recreated | `onSaveInstanceState` / prefs / ViewModel |
| Fragment duplicated on rotation | Adding fragment every onCreate | `if (savedInstanceState == null)` |
| Menu not showing | NoActionBar theme | Theme with ActionBar or set Toolbar |
| Red `R` / `R` cannot be resolved | XML error somewhere in res | Fix XML (check Build tab), then **Build → Rebuild Project** |
| Invalid resource name | Capitals/dashes in file names | Only `a-z 0-9 _` |
| Gradle `Could not resolve ...` | Bad version / typo | Use the version the IDE suggests |
| Button colour ignored | Material theme overrides `background` | `app:backgroundTint="#..."` |
| Emulator won't start | AVD issue | Device Manager → Cold Boot / Wipe Data |
| App runs old code | Cache | **Build → Clean Project**, then Run |
| Toast not showing in fragment | Wrong context | `requireContext()` |

## Debug fast
```java
Log.d("DBG", "value = " + value);
```
- Breakpoint: click the gutter, then **Debug** (bug icon).
- **Build → Rebuild Project** for weird `R`/binding issues.

## Input validation template
```java
private Double readDouble(EditText et, String fieldName) {
    String s = et.getText().toString().trim();
    if (s.isEmpty()) { et.setError(fieldName + " is required"); return null; }
    try {
        double v = Double.parseDouble(s);
        if (v <= 0) { et.setError(fieldName + " must be > 0"); return null; }
        return v;
    } catch (NumberFormatException e) {
        et.setError("Invalid number"); return null;
    }
}
// usage
Double h = readDouble(heightET, "Height"); Double w = readDouble(weightET, "Weight");
if (h == null || w == null) return;
```

## Hide keyboard
```java
InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
```
