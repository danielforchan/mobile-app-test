# 04 · Fragments

[⬅ Index](../README.md)

![fragments](../images/04-fragments.svg)

**Fragment lifecycle:** onAttach → onCreate → **onCreateView** (inflate) → **onViewCreated** (findViewById, listeners) → onStart → onResume → onPause → onStop → onDestroyView → onDestroy → onDetach

## 1. Fragment class
```java
public class ItemListFragment extends Fragment {

    // Interface for talking to the host Activity
    public interface OnItemSelectedListener { void onItemSelected(String item); }
    private OnItemSelectedListener listener;

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        if (context instanceof OnItemSelectedListener) {
            listener = (OnItemSelectedListener) context;
        } else {
            throw new ClassCastException(context + " must implement OnItemSelectedListener");
        }
    }

    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.item_list_fragment, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        view.findViewById(R.id.item1).setOnClickListener(v -> listener.onItemSelected("Item 1 Details"));
        view.findViewById(R.id.item2).setOnClickListener(v -> listener.onItemSelected("Item 2 Details"));
    }
}
```
```java
public class ItemDetailFragment extends Fragment {
    private TextView detailTV;

    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, ViewGroup container, Bundle b) {
        View view = inflater.inflate(R.layout.item_detail_fragment, container, false);
        detailTV = view.findViewById(R.id.item_detail_text);
        return view;
    }
    public void updateDetail(String detail) {
        if (detailTV != null) detailTV.setText(detail);
    }
}
```
Import `androidx.fragment.app.Fragment` (NOT `android.app.Fragment`).

## 2a. Static (in XML)
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent" android:layout_height="match_parent"
    android:orientation="horizontal">
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragment_list"
        android:name="com.example.myapp.ItemListFragment"
        android:layout_width="0dp" android:layout_height="match_parent" android:layout_weight="1"/>
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragment_detail"
        android:name="com.example.myapp.ItemDetailFragment"
        android:layout_width="0dp" android:layout_height="match_parent" android:layout_weight="2"/>
</LinearLayout>
```
(`<fragment>` tag also works. `android:name` = **full package name**.)

```java
public class MainActivity extends AppCompatActivity implements ItemListFragment.OnItemSelectedListener {
    @Override protected void onCreate(Bundle s) {
        super.onCreate(s);
        setContentView(R.layout.activity_main);
    }
    @Override public void onItemSelected(String item) {
        ItemDetailFragment f = (ItemDetailFragment) getSupportFragmentManager()
                .findFragmentById(R.id.fragment_detail);
        if (f != null) f.updateDetail(item);
    }
}
```

## 2b. Dynamic (FragmentTransaction)
Layout: empty `FrameLayout`s as containers.
```xml
<FrameLayout android:id="@+id/container_list"   android:layout_width="0dp" android:layout_height="match_parent" android:layout_weight="1"/>
<FrameLayout android:id="@+id/container_detail" android:layout_width="0dp" android:layout_height="match_parent" android:layout_weight="2"/>
```
```java
private ItemDetailFragment detailFragment;

@Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    if (savedInstanceState == null) {               // avoid duplicates on rotation
        detailFragment = new ItemDetailFragment();
        getSupportFragmentManager().beginTransaction()
            .replace(R.id.container_list, new ItemListFragment())
            .replace(R.id.container_detail, detailFragment, "DETAIL")
            .commit();
    } else {
        detailFragment = (ItemDetailFragment) getSupportFragmentManager().findFragmentByTag("DETAIL");
    }
}
```

## Swap fragments in ONE container + back stack
```java
private void show(Fragment f) {
    getSupportFragmentManager().beginTransaction()
        .replace(R.id.container, f)
        .addToBackStack(null)        // Back returns to previous fragment
        .commit();
}
btnA.setOnClickListener(v -> show(new FragmentA()));
btnB.setOnClickListener(v -> show(new FragmentB()));
```
Pop manually: `getSupportFragmentManager().popBackStack();`

## Pass arguments to a fragment
```java
public static DetailFragment newInstance(String title) {
    DetailFragment f = new DetailFragment();
    Bundle args = new Bundle();
    args.putString("TITLE", title);
    f.setArguments(args);
    return f;
}
// inside fragment (onViewCreated)
String title = getArguments() != null ? getArguments().getString("TITLE") : "";
```

## Context inside a fragment
`requireContext()`, `requireActivity()`. Example: `Toast.makeText(requireContext(), "hi", Toast.LENGTH_SHORT).show();`

## Fragment state on rotation
```java
@Override public void onSaveInstanceState(@NonNull Bundle out) {
    super.onSaveInstanceState(out); out.putString("TEXT", detailTV.getText().toString());
}
// restore in onViewCreated(view, savedInstanceState)
```

## Shared ViewModel between fragments
```java
SharedVM vm = new ViewModelProvider(requireActivity()).get(SharedVM.class);
vm.getSelected().observe(getViewLifecycleOwner(), item -> detailTV.setText(item));
```

## Custom back behaviour (replaces onBackPressed)
```java
getOnBackPressedDispatcher().addCallback(this, new OnBackPressedCallback(true) {
    @Override public void handleOnBackPressed() {
        getSupportFragmentManager().popBackStack(null, FragmentManager.POP_BACK_STACK_INCLUSIVE);
    }
});
```
