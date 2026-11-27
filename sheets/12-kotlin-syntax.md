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
| `for (int i=10;i>=0;i-=2)` | `for (i in 10 downTo 0 step 2)` |
| `for (String x : list)` | `for (x in list)` / `list.forEach { }` |
| `new Intent(this, B.class)` | `Intent(this, B::class.java)` |
| `MainActivity.this` | `this@MainActivity` |
| `v -> foo()` | `{ foo() }` |
| `static` | `companion object { }` |
| `instanceof` + cast | `if (x is Foo) x.bar()` (smart cast) |
| `String.format("%.2f", d)` | `"%.2f".format(d)` |
| `"Hi " + name` | `"Hi $name, age ${age + 1}"` |
| `Math.max(lo, Math.min(hi, x))` | `x.coerceIn(lo, hi)` |

## Views
```kotlin
lateinit var heightET: EditText
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    heightET = findViewById(R.id.heightET)
    val btn = findViewById<Button>(R.id.reportBtn)
    btn.setOnClickListener {
        val h = heightET.text.toString().toDoubleOrNull()
        if (h == null) { heightET.error = "Invalid"; return@setOnClickListener }
        val i = Intent(this, ReportActivity::class.java).apply { putExtra("H", h) }
        startActivity(i)
    }
}
// receive
val h = intent.getDoubleExtra("H", 0.0)
Toast.makeText(this, "BMI %.2f".format(bmi), Toast.LENGTH_SHORT).show()
```

## Functions & classes
```kotlin
fun bmi(w: Double, hCm: Double): Double { val m = hCm / 100; return w / (m * m) }
fun square(x: Int) = x * x                       // expression body
fun greet(name: String = "World") = "Hello $name" // default arg

class Person(val name: String, var age: Int) {
    fun introduce() = println("I'm $name, $age")
}
data class User(val id: Int, val name: String)    // equals/hashCode/toString/copy
enum class ItemType { NUMBER, COLOR }
interface OnItemClick { fun onClick(pos: Int) }
object Singleton { var count = 0 }                // singleton
```

## Collections
```kotlin
val list = listOf(1, 2, 3)               // immutable
val ml = mutableListOf<String>()         // ml.add("a"); ml.removeAt(0)
val set = setOf("a", "b")
val map = mapOf("a" to 1, "b" to 2);  val mm = mutableMapOf<String, Int>(); mm["c"] = 3
val arr = intArrayOf(0, 0, 0); val board = Array(9) { "" }

list.filter { it > 1 }.map { it * 2 }.sum()
list.sortedBy { it } ; list.size ; list.isEmpty() ; list.indices ; list.first()
for ((i, v) in list.withIndex()) println("$i=$v")
```

## Null safety
```kotlin
val name: String? = null
name?.length            // null
name!!.length           // throws NPE
name?.let { println(it) }   // runs only if non-null
val len = name?.length ?: 0
```

## Scope functions
```kotlin
val tv = TextView(this).apply { text = "Hi"; textSize = 20f }
prefs.edit().apply { putInt("best", 10); apply() }
```

## Anonymous objects
```kotlin
seekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
    override fun onProgressChanged(s: SeekBar?, p: Int, fromUser: Boolean) { tv.text = "$p" }
    override fun onStartTrackingTouch(s: SeekBar?) {}
    override fun onStopTrackingTouch(s: SeekBar?) {}
})
```
Tip: **Code → Convert Java File to Kotlin File** is a built-in IDE feature, not AI.
