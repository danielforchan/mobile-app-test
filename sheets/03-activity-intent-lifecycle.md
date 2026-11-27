# 03 · Activity, Intent, Lifecycle, Dialogs, Menu

[⬅ Index](../README.md)

## Lifecycle cheat
```
onCreate → onStart → onResume → [RUNNING] → onPause → onStop → onDestroy
                                      ↑ onRestart ←───────┘
```
| Scenario | Calls |
|---|---|
| First launch | onCreate → onStart → onResume |
| Open another Activity, then Back | onPause → onStop → *(back)* onRestart → onStart → **onResume** |
| **Rotate screen** | onPause → onStop → onDestroy → onCreate → onStart → onResume ⚠️ data lost |
| Home button | onPause → onStop |

> **Refresh data in `onResume()`.** It runs on first launch AND every time you return from another screen (e.g. after Settings).

## Basic Activity (Java)
```java
public class MainActivity extends AppCompatActivity {
    private EditText heightET;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        heightET = findViewById(R.id.heightET);
        findViewById(R.id.reportBtn).setOnClickListener(v -> openReport());
    }
}
```
> New templates may add `EdgeToEdge.enable(this)` and `ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), ...)`. If you remove `android:id="@+id/main"` from the root layout, delete that block too, or the app crashes with a NullPointerException.

## Explicit Intent + extras
```java
// Sender
Intent i = new Intent(MainActivity.this, ReportActivity.class);
i.putExtra("HEIGHT", 170.0);
i.putExtra("NAME", "Daniel");
startActivity(i);

// Receiver (ReportActivity.onCreate)
double h = getIntent().getDoubleExtra("HEIGHT", 0.0);
String name = getIntent().getStringExtra("NAME");
```
To go back: `finish();`

## Get a result back (ActivityResultLauncher)
```java
private final ActivityResultLauncher<Intent> launcher =
    registerForActivityResult(new ActivityResultContracts.StartActivityForResult(), result -> {
        if (result.getResultCode() == RESULT_OK && result.getData() != null) {
            replyTV.setText(result.getData().getStringExtra("REPLY"));
        }
    });
launcher.launch(new Intent(this, SecondActivity.class));

// SecondActivity
Intent data = new Intent();
data.putExtra("REPLY", "hello back");
setResult(RESULT_OK, data);
finish();
```

## Implicit intents
```java
startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("https://www.cityu.edu.hk")));
startActivity(new Intent(Intent.ACTION_DIAL, Uri.parse("tel:12345678")));
Intent s = new Intent(Intent.ACTION_SEND);
s.setType("text/plain");
s.putExtra(Intent.EXTRA_TEXT, "My BMI is 22");
startActivity(Intent.createChooser(s, "Share via"));
```

## Save state on rotation
```java
@Override
protected void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putInt("SCORE", score);
}
// in onCreate:
if (savedInstanceState != null) score = savedInstanceState.getInt("SCORE", 0);
```

## AlertDialog
![dialogs](../images/03-dialogs-menu.svg)

```java
new AlertDialog.Builder(this)          // androidx.appcompat.app.AlertDialog
    .setTitle("Clear all?")
    .setMessage("This cannot be undone.")
    .setPositiveButton("OK", (d, which) -> clearAll())
    .setNegativeButton("Cancel", null)
    .show();
```
List-choice dialog:
```java
String[] items = {"Easy","Medium","Hard"};
new AlertDialog.Builder(this).setTitle("Level")
    .setItems(items, (d, which) -> levelTV.setText(items[which])).show();
```
Input dialog:
```java
EditText input = new EditText(this);
new AlertDialog.Builder(this).setTitle("Name").setView(input)
    .setPositiveButton("OK", (d, w) -> nameTV.setText(input.getText().toString()))
    .show();
```

## Options menu (⋮)
`res/menu/main_menu.xml`
```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/action_settings" android:title="Settings"/>
    <item android:id="@+id/action_clear" android:title="Clear All Data"/>
</menu>
```
```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_menu, menu);
    return true;
}
@Override
public boolean onOptionsItemSelected(@NonNull MenuItem item) {
    int id = item.getItemId();
    if (id == R.id.action_settings) { startActivity(new Intent(this, SettingsActivity.class)); return true; }
    if (id == R.id.action_clear) { confirmClear(); return true; }
    return super.onOptionsItemSelected(item);
}
```
> The menu needs an ActionBar, so the theme must not be `NoActionBar` (sheet 01).

Back arrow on the second activity: `getSupportActionBar().setDisplayHomeAsUpEnabled(true);` + `android:parentActivityName` in the manifest, or:
```java
@Override public boolean onSupportNavigateUp() { finish(); return true; }
```

## Back stack
- `startActivity` pushes. **Back** / `finish()` pops.
- Clear the stack: `i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK | Intent.FLAG_ACTIVITY_CLEAR_TASK);`
