# 13 · Common Errors & Fixes

[⬅ Index](../README.md)

In **Logcat**, filter by your package, then look for `FATAL EXCEPTION` and the first line that names **your** class.

| Error / Symptom | Cause | Fix |
|---|---|---|
| `ActivityNotFoundException` | Activity not in manifest | `<activity android:name=".XActivity"/>` |
| `NullPointerException` at `findViewById` | Wrong ID / wrong layout / called before `setContentView` | Check the ID exists **in this layout (and layout-land!)** |
| NPE at `findViewById(R.id.main)` insets code | removed `android:id="@+id/main"` | Re-add the id or delete the insets block |
| `NumberFormatException` | `parseInt("")` | Check `isEmpty()` + try/catch |
| `CalledFromWrongThreadException` | UI update from background thread | `runOnUiThread` / `handler.post` |
| `Cannot access database on the main thread` | Room on UI thread | Executor or `.allowMainThreadQueries()` |
| Room: `Cannot find getter for field` | private fields | make fields `public` or add getters + setters |
| Room: `cannot find implementation for ...Database` | missing compiler dependency | `annotationProcessor("androidx.room:room-compiler:2.6.1")` |
| Room: `@PrimaryKey String` error | missing `@NonNull` | `@PrimaryKey @NonNull public String date;` |
| `NetworkOnMainThreadException` | network on UI thread | Executor / coroutine |
| Options menu not showing | NoActionBar theme | change the theme parent (sheet 01) |
| Main screen doesn't update after Settings | loading only in `onCreate` | load in **`onResume()`** |
| Date picker month off by one | `Calendar.MONTH` is 0-based | `month + 1` when formatting |
| RecyclerView empty | no LayoutManager / adapter / notify | `setLayoutManager`, `setAdapter`, `notifyDataSetChanged` |
| Each row is full screen | item root `match_parent` height | `wrap_content` |
| Data lost on rotation | Activity recreated | `onSaveInstanceState` / ViewModel |
| Fragment duplicated on rotation | added in every onCreate | `if (savedInstanceState == null)` |
| `You need to use a Theme.AppCompat theme` | wrong theme | keep the template theme |
| Red `R` | error in some XML | fix it (Build tab), then **Build → Rebuild Project** |
| Invalid resource name | capitals/dashes | only `a-z 0-9 _` |
| Button colour ignored | Material theme | `app:backgroundTint="#..."` |
| App runs old code | cache | **Build → Clean Project**, then Run |

## Input validation template
```java
private Integer readPositiveInt(EditText et, String field) {
    String s = et.getText().toString().trim();
    if (s.isEmpty()) { et.setError(field + " is required"); return null; }
    try {
        int v = Integer.parseInt(s);
        if (v <= 0) { et.setError(field + " must be > 0"); return null; }
        return v;
    } catch (NumberFormatException e) {
        et.setError("Invalid number"); return null;
    }
}
```

## Hide the keyboard
```java
InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
```
