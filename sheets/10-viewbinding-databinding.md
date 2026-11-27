# 10 · ViewBinding & DataBinding

[⬅ Index](../README.md)

## ViewBinding (recommended, simple)
Gradle: `buildFeatures { viewBinding = true }` (kts) or `viewBinding true` (groovy), then Sync.
`activity_main.xml` → `ActivityMainBinding`, `item_view.xml` → `ItemViewBinding`. IDs become camelCase (`bmi_msg` → `bmiMsg`).

### Activity (Java)
```java
private ActivityMainBinding binding;

@Override protected void onCreate(Bundle s) {
    super.onCreate(s);
    binding = ActivityMainBinding.inflate(getLayoutInflater());
    setContentView(binding.getRoot());

    binding.submitButton.setOnClickListener(v -> {
        String h = binding.heightInput.getText().toString();
        binding.bmiMsgTextView.setText("You look great");
    });
}
```
### Activity (Kotlin)
```kotlin
private lateinit var binding: ActivityMainBinding
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)
    binding.submitButton.setOnClickListener { binding.bmiMsgTextView.text = "Hi" }
}
```
### Fragment (Java)
```java
private FragmentHomeBinding binding;
@Override public View onCreateView(@NonNull LayoutInflater inf, ViewGroup c, Bundle s) {
    binding = FragmentHomeBinding.inflate(inf, c, false);
    return binding.getRoot();
}
@Override public void onDestroyView() { super.onDestroyView(); binding = null; }
```
### RecyclerView ViewHolder
```java
static class VH extends RecyclerView.ViewHolder {
    final ItemViewBinding b;
    VH(ItemViewBinding b) { super(b.getRoot()); this.b = b; }
}
@NonNull @Override public VH onCreateViewHolder(@NonNull ViewGroup p, int t) {
    return new VH(ItemViewBinding.inflate(LayoutInflater.from(p.getContext()), p, false));
}
@Override public void onBindViewHolder(@NonNull VH h, int pos) { h.b.titleTV.setText(data.get(pos)); }
```

---
## DataBinding
Gradle: `buildFeatures { dataBinding = true }`. Kotlin projects also need the `kotlin-kapt` plugin.

### Layout wrapped in `<layout>`
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="name" type="String"/>
        <variable name="viewModel" type="com.example.app.UserViewModel"/>
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView android:id="@+id/nameTV"
            android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="@{name.toUpperCase()}"/>              <!-- one-way -->
        <EditText android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="@={viewModel.name}"/>                 <!-- two-way, name is MutableLiveData<String> -->
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="Go" android:onClick="@{() -> viewModel.onGo()}"/>
    </LinearLayout>
</layout>
```
### Activity
```java
ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
binding.setName("John");
UserViewModel vm = new ViewModelProvider(this).get(UserViewModel.class);
binding.setViewModel(vm);
binding.setLifecycleOwner(this);   // REQUIRED for LiveData to auto-update UI
```
### In RecyclerView
```java
binding.setNumber(number);
binding.executePendingBindings();   // apply immediately
```
Expressions like `@{String.valueOf(number)}` and `@{Integer.toBinaryString(number)}` work in XML.

| | ViewBinding | DataBinding |
|---|---|---|
| Setup | flag only | flag + `<layout>` tag |
| Auto UI update | ❌ manual | ✅ observes LiveData |
| Best for | simple apps (exam!) | MVVM |
