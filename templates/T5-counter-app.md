# T5 · Counter App: +/−/Reset, Step, Limit, Saved Count, Landscape, Multi-Counter List

[⬅ Index](../README.md)

![Counter screens](../images/T5-counter.svg)
![Counter landscape](../images/T5-counter-land.svg)

**Features:**
- **+ / −** change the count by the step value. Tapping the big number adds 1. Long-pressing **+** adds 10.
- Range is 0–99. At the limit the number turns red and a Toast appears.
- **Reset** asks for confirmation.
- Tracks total taps and the highest value.
- Saved in SharedPreferences, so the count survives rotation and app restart.
- Separate landscape layout using the same IDs.

---
## A. Single counter (Java)

### res/layout/activity_main.xml (portrait)
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="24dp"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Count"
        android:textSize="20sp" />

    <TextView
        android:id="@+id/countTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="0"
        android:textSize="96sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/msgTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Tap + to count"
        android:textColor="#79747E" />

    <!-- − Reset + : equal-width buttons -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:orientation="horizontal">

        <Button
            android:id="@+id/minusBtn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="−"
            android:textSize="24sp" />

        <Button
            android:id="@+id/resetBtn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="8dp"
            android:layout_marginEnd="8dp"
            android:text="Reset" />

        <Button
            android:id="@+id/plusBtn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="+"
            android:textSize="24sp" />
    </LinearLayout>

    <!-- Step: [ 1 ] -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Step:"
            android:textSize="16sp" />

        <EditText
            android:id="@+id/stepET"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="8dp"
            android:inputType="number"
            android:text="1" />
    </LinearLayout>

    <TextView
        android:id="@+id/totalTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Total taps: 0" />

    <TextView
        android:id="@+id/bestTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Highest: 0" />
</LinearLayout>
```

### res/layout-land/activity_main.xml (landscape, **same IDs**)
The number goes on the left half and the controls on the right half.
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal"
    android:padding="16dp">

    <!-- LEFT: number -->
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:gravity="center"
        android:orientation="vertical">

        <TextView
            android:id="@+id/countTV"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0"
            android:textSize="96sp"
            android:textStyle="bold" />

        <TextView
            android:id="@+id/msgTV"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Tap + to count"
            android:textColor="#79747E" />
    </LinearLayout>

    <!-- RIGHT: controls -->
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:gravity="center_vertical"
        android:orientation="vertical">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">
            <Button android:id="@+id/minusBtn" android:layout_width="0dp" android:layout_height="wrap_content"
                android:layout_weight="1" android:text="−" android:textSize="24sp" />
            <Button android:id="@+id/resetBtn" android:layout_width="0dp" android:layout_height="wrap_content"
                android:layout_weight="1" android:layout_marginStart="8dp" android:layout_marginEnd="8dp"
                android:text="Reset" />
            <Button android:id="@+id/plusBtn" android:layout_width="0dp" android:layout_height="wrap_content"
                android:layout_weight="1" android:text="+" android:textSize="24sp" />
        </LinearLayout>

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:orientation="horizontal">
            <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
                android:text="Step:" android:textSize="16sp" />
            <EditText android:id="@+id/stepET" android:layout_width="0dp" android:layout_height="wrap_content"
                android:layout_weight="1" android:layout_marginStart="8dp" android:inputType="number"
                android:text="1" />
        </LinearLayout>

        <TextView android:id="@+id/totalTV" android:layout_width="wrap_content"
            android:layout_height="wrap_content" android:text="Total taps: 0" />
        <TextView android:id="@+id/bestTV" android:layout_width="wrap_content"
            android:layout_height="wrap_content" android:text="Highest: 0" />
    </LinearLayout>
</LinearLayout>
```
> To create it: right-click `res` → **New → Android Resource Directory** → `layout-land`, then copy `activity_main.xml` into it.

### MainActivity.java
```java
package com.example.counter;

import android.content.SharedPreferences;
import android.graphics.Color;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    private static final int MIN = 0;
    private static final int MAX = 99;
    private static final String PREF = "COUNTER";

    private int count = 0;     // current value
    private int taps = 0;      // how many times the user changed it
    private int best = 0;      // highest value reached

    private TextView countTV, msgTV, totalTV, bestTV;
    private EditText stepET;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        countTV = findViewById(R.id.countTV);
        msgTV   = findViewById(R.id.msgTV);
        totalTV = findViewById(R.id.totalTV);
        bestTV  = findViewById(R.id.bestTV);
        stepET  = findViewById(R.id.stepET);
        Button plusBtn  = findViewById(R.id.plusBtn);
        Button minusBtn = findViewById(R.id.minusBtn);
        Button resetBtn = findViewById(R.id.resetBtn);

        // load saved values (also restores after rotation, because onPause saved them)
        SharedPreferences p = getSharedPreferences(PREF, MODE_PRIVATE);
        count = p.getInt("count", 0);
        taps  = p.getInt("taps", 0);
        best  = p.getInt("best", 0);

        plusBtn.setOnClickListener(v -> change(+getStep()));
        minusBtn.setOnClickListener(v -> change(-getStep()));
        countTV.setOnClickListener(v -> change(+1));                          // tap number = +1
        plusBtn.setOnLongClickListener(v -> { change(+10); return true; });  // long press = +10

        resetBtn.setOnClickListener(v -> new AlertDialog.Builder(this)
                .setTitle("Reset counter?")
                .setMessage("Current count " + count + " will be set to 0.")
                .setPositiveButton("Reset", (d, w) -> { count = 0; updateUI(); })
                .setNegativeButton("Cancel", null)
                .show());

        updateUI();
    }

    /** Reads the step EditText. Falls back to 1 if empty/invalid. */
    private int getStep() {
        String s = stepET.getText().toString().trim();
        try {
            int step = Integer.parseInt(s);
            if (step <= 0) throw new NumberFormatException();
            return step;
        } catch (NumberFormatException e) {
            stepET.setError("Step must be a positive number");
            stepET.setText("1");
            return 1;
        }
    }

    private void change(int delta) {
        int next = count + delta;
        if (next > MAX) {
            next = MAX;
            Toast.makeText(this, "Maximum " + MAX + " reached!", Toast.LENGTH_SHORT).show();
        } else if (next < MIN) {
            next = MIN;
            Toast.makeText(this, "Cannot go below " + MIN, Toast.LENGTH_SHORT).show();
        }
        count = next;
        taps++;
        if (count > best) best = count;
        updateUI();
        bounce();
    }

    private void updateUI() {
        countTV.setText(String.valueOf(count));
        totalTV.setText("Total taps: " + taps);
        bestTV.setText("Highest: " + best);

        if (count >= MAX) {
            countTV.setTextColor(Color.RED);
            msgTV.setText("Limit reached");
        } else if (count == MIN) {
            countTV.setTextColor(Color.GRAY);
            msgTV.setText("Tap + to count");
        } else {
            countTV.setTextColor(Color.parseColor("#6750A4"));
            msgTV.setText(count % 10 == 0 ? "Nice, a multiple of 10!" : "");
        }
    }

    private void bounce() {
        countTV.animate().scaleX(1.3f).scaleY(1.3f).setDuration(100)
               .withEndAction(() -> countTV.animate().scaleX(1f).scaleY(1f).setDuration(100));
    }

    @Override
    protected void onPause() {          // runs on rotation, Home and Back, so it always saves
        super.onPause();
        getSharedPreferences(PREF, MODE_PRIVATE).edit()
                .putInt("count", count)
                .putInt("taps", taps)
                .putInt("best", best)
                .apply();
    }
}
```

**If the task says the count should survive rotation only, not closing the app:** remove the prefs code and use this instead:
```java
@Override protected void onSaveInstanceState(Bundle out) {
    super.onSaveInstanceState(out);
    out.putInt("count", count); out.putInt("taps", taps); out.putInt("best", best);
}
// in onCreate, after findViewById:
if (savedInstanceState != null) {
    count = savedInstanceState.getInt("count");
    taps  = savedInstanceState.getInt("taps");
    best  = savedInstanceState.getInt("best");
}
```

### Quick modifications
| Requirement | Change |
|---|---|
| Different max/min | edit the `MIN` / `MAX` constants |
| Allow negative numbers | `MIN = Integer.MIN_VALUE` (and remove the gray colour rule) |
| Start at 10 and count down | initial `count = 10`, and swap `+`/`−` |
| Buttons in a column | button row `orientation="vertical"`, buttons `layout_width="match_parent"` instead of `0dp + weight` |
| Bigger number | `android:textSize="120sp"` |
| Number in a circle | `android:background="@drawable/circle"` (sheet 11) + `200dp` width/height + `android:gravity="center"` |

---
## B. MVVM version (ViewModel + LiveData)
```java
public class CounterViewModel extends ViewModel {
    private final MutableLiveData<Integer> count = new MutableLiveData<>(0);
    public LiveData<Integer> getCount() { return count; }
    public void add(int d) {
        int n = count.getValue() == null ? 0 : count.getValue();
        n = Math.max(0, Math.min(99, n + d));
        count.setValue(n);
    }
    public void reset() { count.setValue(0); }
}
```
```java
// MainActivity.onCreate
CounterViewModel vm = new ViewModelProvider(this).get(CounterViewModel.class);
vm.getCount().observe(this, c -> countTV.setText(String.valueOf(c)));   // survives rotation
plusBtn.setOnClickListener(v -> vm.add(1));
minusBtn.setOnClickListener(v -> vm.add(-1));
resetBtn.setOnClickListener(v -> vm.reset());
```

---
## C. Kotlin version (single counter)
```kotlin
class MainActivity : AppCompatActivity() {
    private var count = 0
    private lateinit var countTV: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        countTV = findViewById(R.id.countTV)
        count = getSharedPreferences("COUNTER", MODE_PRIVATE).getInt("count", 0)

        findViewById<Button>(R.id.plusBtn).setOnClickListener { change(1) }
        findViewById<Button>(R.id.minusBtn).setOnClickListener { change(-1) }
        findViewById<Button>(R.id.resetBtn).setOnClickListener { count = 0; show() }
        show()
    }
    private fun change(d: Int) {
        count = (count + d).coerceIn(0, 99)
        if (count == 99) Toast.makeText(this, "Max reached", Toast.LENGTH_SHORT).show()
        show()
    }
    private fun show() {
        countTV.text = count.toString()
        countTV.setTextColor(if (count >= 99) Color.RED else Color.BLACK)
    }
    override fun onPause() {
        super.onPause()
        getSharedPreferences("COUNTER", MODE_PRIVATE).edit().putInt("count", count).apply()
    }
}
```

---
## D. Multi-counter list (RecyclerView tally counter)

![Multi counter](../images/T5-multi-counter.svg)

Each row has a name, −, the count and +, with the total shown on top. You add counters by name and long-press a row to delete it. The list is saved as JSON in SharedPreferences.

### Counter.java
```java
public class Counter {
    public String name;
    public int count;
    public Counter(String name, int count) { this.name = name; this.count = count; }
}
```

### res/layout/item_counter.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:orientation="horizontal"
    android:padding="8dp">

    <TextView
        android:id="@+id/nameTV"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:textSize="18sp" />

    <Button
        android:id="@+id/minusBtn"
        android:layout_width="56dp"
        android:layout_height="wrap_content"
        android:text="−" />

    <TextView
        android:id="@+id/countTV"
        android:layout_width="48dp"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:textSize="20sp"
        android:textStyle="bold" />

    <Button
        android:id="@+id/plusBtn"
        android:layout_width="56dp"
        android:layout_height="wrap_content"
        android:text="+" />
</LinearLayout>
```

### res/layout/activity_multi.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/totalTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Total: 0"
        android:textSize="22sp"
        android:textStyle="bold" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <EditText
            android:id="@+id/nameET"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Counter name" />
        <Button
            android:id="@+id/addBtn"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Add" />
    </LinearLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rv"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
</LinearLayout>
```

### CounterAdapter.java
```java
public class CounterAdapter extends RecyclerView.Adapter<CounterAdapter.VH> {

    public interface Listener {
        void onCountChanged();          // update total
        void onDelete(int position);    // long press
    }

    private final List<Counter> data;
    private final Listener listener;

    public CounterAdapter(List<Counter> data, Listener listener) {
        this.data = data; this.listener = listener;
    }

    static class VH extends RecyclerView.ViewHolder {
        TextView name, count; Button plus, minus;
        VH(View v) {
            super(v);
            name  = v.findViewById(R.id.nameTV);
            count = v.findViewById(R.id.countTV);
            plus  = v.findViewById(R.id.plusBtn);
            minus = v.findViewById(R.id.minusBtn);
        }
    }

    @NonNull @Override
    public VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_counter, parent, false);
        return new VH(v);
    }

    @Override
    public void onBindViewHolder(@NonNull VH h, int position) {
        Counter c = data.get(position);
        h.name.setText(c.name);
        h.count.setText(String.valueOf(c.count));

        h.plus.setOnClickListener(v -> {
            int p = h.getAdapterPosition();
            if (p == RecyclerView.NO_POSITION) return;
            data.get(p).count++;
            notifyItemChanged(p);
            listener.onCountChanged();
        });
        h.minus.setOnClickListener(v -> {
            int p = h.getAdapterPosition();
            if (p == RecyclerView.NO_POSITION || data.get(p).count == 0) return;
            data.get(p).count--;
            notifyItemChanged(p);
            listener.onCountChanged();
        });
        h.itemView.setOnLongClickListener(v -> {
            listener.onDelete(h.getAdapterPosition());
            return true;
        });
    }

    @Override public int getItemCount() { return data.size(); }
}
```

### MultiCounterActivity.java
```java
public class MultiCounterActivity extends AppCompatActivity {

    private final List<Counter> counters = new ArrayList<>();
    private CounterAdapter adapter;
    private TextView totalTV;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_multi);

        totalTV = findViewById(R.id.totalTV);
        EditText nameET = findViewById(R.id.nameET);
        RecyclerView rv = findViewById(R.id.rv);

        load();

        adapter = new CounterAdapter(counters, new CounterAdapter.Listener() {
            @Override public void onCountChanged() { updateTotal(); }
            @Override public void onDelete(int pos) {
                if (pos == RecyclerView.NO_POSITION) return;
                new AlertDialog.Builder(MultiCounterActivity.this)
                    .setTitle("Delete " + counters.get(pos).name + "?")
                    .setPositiveButton("Delete", (d, w) -> {
                        counters.remove(pos);
                        adapter.notifyItemRemoved(pos);
                        updateTotal();
                    })
                    .setNegativeButton("Cancel", null).show();
            }
        });
        rv.setLayoutManager(new LinearLayoutManager(this));
        rv.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));
        rv.setAdapter(adapter);

        findViewById(R.id.addBtn).setOnClickListener(v -> {
            String name = nameET.getText().toString().trim();
            if (name.isEmpty()) { nameET.setError("Enter a name"); return; }
            counters.add(new Counter(name, 0));
            adapter.notifyItemInserted(counters.size() - 1);
            nameET.setText("");
            updateTotal();
        });

        updateTotal();
    }

    private void updateTotal() {
        int sum = 0;
        for (Counter c : counters) sum += c.count;
        totalTV.setText("Total: " + sum);
    }

    // ---- save / load the list as JSON in SharedPreferences ----
    @Override
    protected void onPause() {
        super.onPause();
        JSONArray arr = new JSONArray();
        try {
            for (Counter c : counters) {
                JSONObject o = new JSONObject();
                o.put("name", c.name);
                o.put("count", c.count);
                arr.put(o);
            }
        } catch (JSONException e) { e.printStackTrace(); }
        getSharedPreferences("MULTI", MODE_PRIVATE).edit().putString("list", arr.toString()).apply();
    }

    private void load() {
        String json = getSharedPreferences("MULTI", MODE_PRIVATE).getString("list", null);
        if (json == null) {                       // first launch: sample data
            counters.add(new Counter("Apples", 0));
            counters.add(new Counter("Oranges", 0));
            return;
        }
        try {
            JSONArray arr = new JSONArray(json);
            for (int i = 0; i < arr.length(); i++) {
                JSONObject o = arr.getJSONObject(i);
                counters.add(new Counter(o.getString("name"), o.getInt("count")));
            }
        } catch (JSONException e) { e.printStackTrace(); }
    }
}
```
In the manifest, add `<activity android:name=".MultiCounterActivity" />`. To make it the start screen, move the `MAIN/LAUNCHER` `<intent-filter>` into it.
To open it from the single counter: `startActivity(new Intent(this, MultiCounterActivity.class));`
