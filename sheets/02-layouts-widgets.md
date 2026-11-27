# 02 · XML Layouts & Widgets

[⬅ Index](../README.md) · More pictures + layout change recipes: [sheet 14](14-screen-gallery.md)

## Size rules
![size](../images/02-size.svg)

| Value | Meaning |
|---|---|
| `match_parent` | fill the parent |
| `wrap_content` | as big as the content |
| `0dp` | in ConstraintLayout = *match constraints*. In LinearLayout, use it with `layout_weight` |
| `padding` / `margin` | space inside the border / outside the border |

![padding vs margin](../images/02-padding-margin.svg)

![gravity](../images/02-gravity.svg)

## LinearLayout (fastest to write)
![layouts](../images/02-layouts.svg)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:id="@+id/titleTV"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="My App"
        android:textSize="24sp"
        android:textStyle="bold" />

    <EditText
        android:id="@+id/heightET"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Height (cm)"
        android:inputType="numberDecimal" />

    <!-- Two buttons side by side, equal width -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <Button
            android:id="@+id/okBtn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="OK" />
        <Button
            android:id="@+id/clearBtn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_marginStart="8dp"
            android:text="Clear" />
    </LinearLayout>
</LinearLayout>
```
For a long form, wrap the content in a `<ScrollView>`, which takes exactly **one** child.

## ConstraintLayout
![constraint](../images/02-constraint.svg)

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/label"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="Hello"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <EditText
        android:id="@+id/input"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        app:layout_constraintTop_toBottomOf="@id/label"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="16dp"
        android:text="Go"
        app:layout_constraintTop_toBottomOf="@id/input"
        app:layout_constraintEnd_toEndOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
Every view needs at least one horizontal **and** one vertical constraint, or it jumps to the top-left at runtime.

## TableLayout
```xml
<TableLayout android:layout_width="match_parent" android:layout_height="wrap_content"
    android:stretchColumns="1">
    <TableRow>
        <TextView android:text="Height:" />
        <EditText android:id="@+id/h" android:inputType="numberDecimal" />
    </TableRow>
    <TableRow>
        <TextView android:text="Weight:" />
        <EditText android:id="@+id/w" android:inputType="numberDecimal" />
    </TableRow>
</TableLayout>
```

## GridLayout (3×3 board)
```xml
<GridLayout
    android:id="@+id/board"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:columnCount="3"
    android:rowCount="3">
    <Button android:id="@+id/b0" android:layout_width="80dp" android:layout_height="80dp" android:textSize="32sp"/>
    <!-- ... b1..b8 -->
</GridLayout>
```

## Widgets cheat-table (XML + Java)
![widgets](../images/02-widgets.svg)

### EditText
```xml
<EditText android:id="@+id/nameET" android:inputType="text" android:hint="Name"/>
<!-- inputType: number, numberDecimal, numberSigned, textPassword, textEmailAddress, phone -->
```
```java
String s = nameET.getText().toString().trim();
if (s.isEmpty()) { nameET.setError("Required"); return; }
int n = Integer.parseInt(s);   // wrap in try/catch NumberFormatException
```

### Button
```java
btn.setOnClickListener(v -> doSomething());
btn.setOnLongClickListener(v -> { doOther(); return true; });
```

### CheckBox
```java
boolean checked = agreeCB.isChecked();
agreeCB.setOnCheckedChangeListener((b, isChecked) -> { });
```

### RadioGroup / RadioButton
```xml
<RadioGroup android:id="@+id/moodRG" android:layout_width="match_parent"
    android:layout_height="wrap_content" android:orientation="horizontal">
    <RadioButton android:id="@+id/happyRB" android:layout_width="wrap_content"
        android:layout_height="wrap_content" android:text="Happy" android:checked="true"/>
    <RadioButton android:id="@+id/sadRB" android:layout_width="wrap_content"
        android:layout_height="wrap_content" android:text="Sad"/>
</RadioGroup>
```
```java
int id = moodRG.getCheckedRadioButtonId();   // -1 if none
String mood = (id == R.id.sadRB) ? "Sad" : "Happy";
// or: RadioButton rb = findViewById(id); String text = rb.getText().toString();
moodRG.check(R.id.happyRB);                  // select in code
```

### Switch / ToggleButton
```java
darkSW.setOnCheckedChangeListener((b, on) -> { });
```

### RatingBar / SeekBar
```java
rb.setOnRatingBarChangeListener((bar, rating, fromUser) -> { });
seek.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
    public void onProgressChanged(SeekBar s, int p, boolean fromUser) { valueTV.setText("" + p); }
    public void onStartTrackingTouch(SeekBar s) {}
    public void onStopTrackingTouch(SeekBar s) {}
});
```

### Spinner (dropdown)
```xml
<Spinner android:id="@+id/levelSP" android:entries="@array/levels"
    android:layout_width="match_parent" android:layout_height="wrap_content"/>
```
```java
String level = levelSP.getSelectedItem().toString();
ArrayAdapter<String> ad = new ArrayAdapter<>(this,
        android.R.layout.simple_spinner_item, new String[]{"A","B","C"});
ad.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
levelSP.setAdapter(ad);
```

### DatePickerDialog
```java
Calendar c = Calendar.getInstance();
DatePickerDialog dlg = new DatePickerDialog(this, (view, y, m, d) -> {
    String date = String.format(Locale.US, "%04d-%02d-%02d", y, m + 1, d);   // month is 0-based!
    dateBtn.setText(date);
}, c.get(Calendar.YEAR), c.get(Calendar.MONTH), c.get(Calendar.DAY_OF_MONTH));
dlg.getDatePicker().setMaxDate(System.currentTimeMillis());   // no future dates
// dlg.getDatePicker().setMinDate(System.currentTimeMillis() - 1000); // no past dates
dlg.show();
```

### TimePickerDialog
```java
new TimePickerDialog(this, (view, h, min) ->
    timeBtn.setText(String.format(Locale.US, "%02d:%02d", h, min)), 9, 0, true).show();
```

### ImageView / ProgressBar / TextView
```java
img.setImageResource(R.drawable.bot_fat);
pb.setProgress(50);  pb.setVisibility(View.GONE);
tv.setText(String.format(Locale.US, "%.2f", bmi));
tv.setTextColor(Color.RED);
tv.setVisibility(View.VISIBLE);   // VISIBLE / INVISIBLE (keeps space) / GONE
Toast.makeText(this, "Saved!", Toast.LENGTH_SHORT).show();
```
```xml
<ProgressBar android:id="@+id/pb" style="?android:attr/progressBarStyleHorizontal"
    android:layout_width="match_parent" android:layout_height="wrap_content" android:max="100"/>
```

## Landscape layout
1. Right-click `res` → **New → Android Resource Directory** → name it `layout-land`, then copy `activity_main.xml` into it.
2. **Keep the same view IDs** in both files.

Example: [T5 landscape](../templates/T5-counter-app.md)

## Shape background
`res/drawable/rounded_bg.xml`
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle">
    <solid android:color="#E3F2FD"/>
    <corners android:radius="12dp"/>
    <stroke android:width="2dp" android:color="#1565C0"/>
</shape>
```
Apply it with `android:background="@drawable/rounded_bg"`. On Material buttons, use `app:backgroundTint` instead.
