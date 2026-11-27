# 12 · Kotlin Quick Syntax (Java ↔ Kotlin)

[⬅ Index](../README.md)

| Java | Kotlin |
|---|---|
| `final String n = "Peter";` | `val n = "Peter"` |
| `int age = 24;` | `var age = 24` |
| `String s = null;` | `var s: String? = null` |
| `s != null ? s.length() : 0` | `s?.length ?: 0` |
| `(cond) ? a : b` | `if (cond) a else b` |
| `switch` | `when (x) { 1 -> ...; 2, 3 -> ...; else -> ... }` |
| `for (int i=0;i<n;i++)` | `for (i in 0 until n)` |
| `for (String x : list)` | `for (x in list)` |
| `new Intent(this, B.class)` | `Intent(this, B::class.java)` |
| `MainActivity.this` | `this@MainActivity` |
| `static` | `companion object { }` |
| `instanceof` | `is` (smart cast) |
| `String.format("%.2f", d)` | `"%.2f".format(d)` |
| `"Hi " + name` | `"Hi $name, ${age + 1}"` |
| `Integer.parseInt(s)` + try/catch | `s.toIntOrNull()` |
| `Math.max(lo, Math.min(hi, x))` | `x.coerceIn(lo, hi)` |

## Views
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    val stepsET = findViewById<EditText>(R.id.stepsET)
    findViewById<Button>(R.id.saveBtn).setOnClickListener {
        val steps = stepsET.text.toString().toIntOrNull()
        if (steps == null || steps <= 0) { stepsET.error = "Invalid"; return@setOnClickListener }
        Toast.makeText(this, "Saved: $steps", Toast.LENGTH_SHORT).show()
    }
}
```

## Classes
```kotlin
data class User(val id: Int, val name: String)    // equals/toString/copy
class Person(val name: String, var age: Int)
enum class Mood { ENERGETIC, NORMAL, TIRED }
object Singleton { var count = 0 }
sealed class UiIntent { object Load : UiIntent(); data class Add(val n: Int) : UiIntent() }
```

## Collections
```kotlin
val list = listOf(1, 2, 3); val ml = mutableListOf<String>()
val map = mapOf("a" to 1); val mm = mutableMapOf<String, Int>()
list.filter { it > 1 }.map { it * 2 }.sum()
list.sortedByDescending { it }
```

## Null safety & scope functions
```kotlin
name?.let { println(it) }
val len = name?.length ?: 0
val tv = TextView(this).apply { text = "Hi"; textSize = 20f }
getSharedPreferences("p", MODE_PRIVATE).edit().putInt("goal", 8000).apply()
```

## Anonymous object
```kotlin
seekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
    override fun onProgressChanged(s: SeekBar?, p: Int, fromUser: Boolean) { tv.text = "$p" }
    override fun onStartTrackingTouch(s: SeekBar?) {}
    override fun onStopTrackingTouch(s: SeekBar?) {}
})
```
**Code → Convert Java File to Kotlin File** is a built-in IDE feature, not AI.
