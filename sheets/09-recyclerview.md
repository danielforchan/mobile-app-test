# 09 · RecyclerView

[⬅ Index](../README.md)

![recyclerview](../images/09-recyclerview.svg)

**4 steps:** ① `RecyclerView` in the layout → ② item layout XML → ③ Adapter + ViewHolder → ④ set LayoutManager + Adapter in the Activity.

## ① activity_main.xml
```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/rv"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    android:layout_weight="1"
    android:scrollbars="vertical" />
```
(Inside a vertical LinearLayout with other views above it, `height=0dp + weight=1` fills the remaining space.)

## ② res/layout/item_view.xml
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="12dp">
    <TextView android:id="@+id/titleTV" android:layout_width="0dp" android:layout_weight="1"
        android:layout_height="wrap_content" android:textSize="18sp"/>
    <TextView android:id="@+id/subTV" android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</LinearLayout>
```
⚠️ The item root height must be **wrap_content**, or each row fills the whole screen.

## ③ Adapter with click + delete (Java)
```java
public class ItemAdapter extends RecyclerView.Adapter<ItemAdapter.VH> {

    public interface OnItemClick { void onClick(int position); }
    public interface OnItemLongClick { void onLongClick(int position); }

    private final List<String> data;
    private final OnItemClick click;
    private final OnItemLongClick longClick;

    public ItemAdapter(List<String> data, OnItemClick click, OnItemLongClick longClick) {
        this.data = data; this.click = click; this.longClick = longClick;
    }

    public static class VH extends RecyclerView.ViewHolder {
        TextView title, sub;
        public VH(@NonNull View row) {
            super(row);
            title = row.findViewById(R.id.titleTV);
            sub = row.findViewById(R.id.subTV);
        }
    }

    @NonNull @Override
    public VH onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_view, parent, false);
        return new VH(v);
    }

    @Override
    public void onBindViewHolder(@NonNull VH h, int position) {
        String item = data.get(position);
        h.title.setText(item);
        h.sub.setText("#" + (position + 1));
        h.itemView.setOnClickListener(v -> click.onClick(h.getAdapterPosition()));
        h.itemView.setOnLongClickListener(v -> { longClick.onLongClick(h.getAdapterPosition()); return true; });
    }

    @Override public int getItemCount() { return data.size(); }
}
```

## ④ Activity
```java
private final List<String> items = new ArrayList<>();
private ItemAdapter adapter;

@Override protected void onCreate(Bundle s) {
    super.onCreate(s);
    setContentView(R.layout.activity_main);

    for (int i = 0; i <= 100; i++) items.add("Item " + i);

    RecyclerView rv = findViewById(R.id.rv);
    EditText inputET = findViewById(R.id.inputET);
    rv.setLayoutManager(new LinearLayoutManager(this));
    rv.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));

    adapter = new ItemAdapter(items,
        pos -> Toast.makeText(this, "Clicked " + items.get(pos), Toast.LENGTH_SHORT).show(),
        pos -> { items.remove(pos); adapter.notifyItemRemoved(pos); });
    rv.setAdapter(adapter);

    findViewById(R.id.addBtn).setOnClickListener(v -> {
        String t = inputET.getText().toString().trim();
        if (t.isEmpty()) return;
        items.add(t);
        adapter.notifyItemInserted(items.size() - 1);
        rv.scrollToPosition(items.size() - 1);
        inputET.setText("");
    });
}
```
Notify methods: `notifyItemInserted(pos)`, `notifyItemRemoved(pos)`, `notifyItemChanged(pos)`, and `notifyDataSetChanged()` to replace everything.

### Replace the whole list (e.g. from Room)
```java
public void setData(List<String> newData) { data.clear(); data.addAll(newData); notifyDataSetChanged(); }
```

## Kotlin Adapter
```kotlin
class IntAdapter(private val data: List<Int>, private val onClick: (Int) -> Unit)
    : RecyclerView.Adapter<IntAdapter.VH>() {
    class VH(row: View) : RecyclerView.ViewHolder(row) {
        val tv: TextView = row.findViewById(R.id.titleTV)
    }
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) =
        VH(LayoutInflater.from(parent.context).inflate(R.layout.item_view, parent, false))
    override fun onBindViewHolder(h: VH, position: Int) {
        h.tv.text = data[position].toString()
        h.itemView.setOnClickListener { onClick(data[position]) }
    }
    override fun getItemCount() = data.size
}
// rv.layoutManager = LinearLayoutManager(this)
// rv.adapter = IntAdapter((0..100).toList()) { n -> Toast.makeText(this, "$n", Toast.LENGTH_SHORT).show() }
```

---
## ListAdapter + DiffUtil (efficient updates)
```java
public class IntListAdapter extends ListAdapter<Integer, IntListAdapter.VH> {
    public IntListAdapter() { super(DIFF); }

    private static final DiffUtil.ItemCallback<Integer> DIFF = new DiffUtil.ItemCallback<Integer>() {
        @Override public boolean areItemsTheSame(@NonNull Integer a, @NonNull Integer b) { return a.equals(b); }
        @Override public boolean areContentsTheSame(@NonNull Integer a, @NonNull Integer b) { return a.equals(b); }
    };

    static class VH extends RecyclerView.ViewHolder {
        TextView tv; VH(View v) { super(v); tv = v.findViewById(R.id.titleTV); }
    }
    @NonNull @Override public VH onCreateViewHolder(@NonNull ViewGroup p, int t) {
        return new VH(LayoutInflater.from(p.getContext()).inflate(R.layout.item_view, p, false));
    }
    @Override public void onBindViewHolder(@NonNull VH h, int pos) {
        h.tv.setText(String.valueOf(getItem(pos)));
    }
}
// Activity
IntListAdapter adapter = new IntListAdapter();
rv.setAdapter(adapter);
adapter.submitList(new ArrayList<>(data));   // submit a NEW list object each time
```
For objects, compare `id` in `areItemsTheSame` and the fields (or `equals`) in `areContentsTheSame`.

---
## Multiple view types (odd = number, even = colour block)
```java
public class MultiAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> {
    private static final int TYPE_NUMBER = 0, TYPE_COLOR = 1;
    private final List<Integer> data;
    public MultiAdapter(List<Integer> d) { data = d; }

    @Override public int getItemViewType(int position) {
        return (position % 2 == 0) ? TYPE_COLOR : TYPE_NUMBER;
    }

    @NonNull @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        LayoutInflater inf = LayoutInflater.from(parent.getContext());
        if (viewType == TYPE_NUMBER)
            return new NumberVH(inf.inflate(R.layout.item_number, parent, false));
        return new ColorVH(inf.inflate(R.layout.item_color, parent, false));
    }

    @Override public void onBindViewHolder(@NonNull RecyclerView.ViewHolder h, int pos) {
        if (h instanceof NumberVH) ((NumberVH) h).bind(data.get(pos));
        else ((ColorVH) h).bind();
    }
    @Override public int getItemCount() { return data.size(); }

    static class NumberVH extends RecyclerView.ViewHolder {
        TextView tv; NumberVH(View v) { super(v); tv = v.findViewById(R.id.text_number); }
        void bind(int n) { tv.setText(String.valueOf(n)); }
    }
    static class ColorVH extends RecyclerView.ViewHolder {
        View block; ColorVH(View v) { super(v); block = v.findViewById(R.id.color_block); }
        void bind() { block.setBackgroundColor(android.graphics.Color.BLUE); }
    }
}
```
For a header + items list, use objects with an `isHeader` flag and return the view type from that flag.

## Grid
```java
rv.setLayoutManager(new GridLayoutManager(this, 2));        // 2 columns

GridLayoutManager gm = new GridLayoutManager(this, 2);
gm.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
    @Override public int getSpanSize(int position) { return position < 3 ? 2 : 1; } // first 3 full width
});
rv.setLayoutManager(gm);
```
Horizontal list: `new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false)`

## Swipe to delete
```java
new ItemTouchHelper(new ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT) {
    @Override public boolean onMove(@NonNull RecyclerView r, @NonNull RecyclerView.ViewHolder a, @NonNull RecyclerView.ViewHolder b) { return false; }
    @Override public void onSwiped(@NonNull RecyclerView.ViewHolder vh, int dir) {
        int pos = vh.getAdapterPosition();
        items.remove(pos); adapter.notifyItemRemoved(pos);
    }
}).attachToRecyclerView(rv);
```

## Simple ListView (if RecyclerView is not required)
```java
ArrayAdapter<String> ad = new ArrayAdapter<>(this, android.R.layout.simple_list_item_1, items);
listView.setAdapter(ad);
listView.setOnItemClickListener((parent, view, pos, id) -> { ... });
ad.notifyDataSetChanged();
```
