# 10 · ViewBinding & DataBinding

[⬅ Index](../README.md)

## ViewBinding
Gradle: `buildFeatures { viewBinding = true }`, then Sync. `activity_main.xml` generates `ActivityMainBinding`, and IDs become camelCase (`bmi_msg` → `bmiMsg`).

```java
private ActivityMainBinding binding;

@Override protected void onCreate(Bundle s) {
    super.onCreate(s);
    binding = ActivityMainBinding.inflate(getLayoutInflater());
    setContentView(binding.getRoot());
    binding.submitButton.setOnClickListener(v -> binding.msgTV.setText("Hi"));
}
```
Kotlin:
```kotlin
private lateinit var binding: ActivityMainBinding
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = ActivityMainBinding.inflate(layoutInflater)
    setContentView(binding.root)
    binding.submitButton.setOnClickListener { binding.msgTV.text = "Hi" }
}
```
Fragment:
```java
private FragmentHomeBinding binding;
@Override public View onCreateView(@NonNull LayoutInflater inf, ViewGroup c, Bundle s) {
    binding = FragmentHomeBinding.inflate(inf, c, false);
    return binding.getRoot();
}
@Override public void onDestroyView() { super.onDestroyView(); binding = null; }
```
ViewHolder:
```java
static class VH extends RecyclerView.ViewHolder {
    final ItemViewBinding b;
    VH(ItemViewBinding b) { super(b.getRoot()); this.b = b; }
}
// onCreateViewHolder: new VH(ItemViewBinding.inflate(LayoutInflater.from(p.getContext()), p, false));
```

## DataBinding
Gradle: `buildFeatures { dataBinding = true }`
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="viewModel" type="com.example.app.UserViewModel"/>
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="@{viewModel.name}"/>                    <!-- one-way -->
        <EditText android:layout_width="match_parent" android:layout_height="wrap_content"
            android:text="@={viewModel.name}"/>                   <!-- two-way (MutableLiveData<String>) -->
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
            android:text="Go" android:onClick="@{() -> viewModel.onGo()}"/>
    </LinearLayout>
</layout>
```
```java
ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_main);
binding.setViewModel(new ViewModelProvider(this).get(UserViewModel.class));
binding.setLifecycleOwner(this);   // REQUIRED for LiveData to update the UI
```

| | ViewBinding | DataBinding |
|---|---|---|
| Setup | flag only | flag + `<layout>` tag |
| Auto UI update | ❌ | ✅ |
| Best for | simple apps (exam) | MVVM |
