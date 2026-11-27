# 05 · Architecture: MVP / MVVM / MVI + LiveData & ViewModel

[⬅ Index](../README.md)

| Pattern | Who updates the View? | Key class |
|---|---|---|
| MVP | Presenter calls `view.showX()` (imperative) | `Contract.View` interface |
| MVVM | View **observes** LiveData (reactive) | `ViewModel` + `LiveData` |
| MVI | View observes a single immutable **State**; user actions = **Intents** | `State` class, `Intent` class |

ViewModel **survives rotation**. LiveData is lifecycle-aware.

---
## MVVM (Java)
### Model
```java
public class User {
    private final String name, email;
    public User(String name, String email) { this.name = name; this.email = email; }
    public String getName() { return name; }
    public String getEmail() { return email; }
}
```
### ViewModel
```java
public class UserViewModel extends ViewModel {
    private final MutableLiveData<User> user = new MutableLiveData<>();
    private final MutableLiveData<Integer> counter = new MutableLiveData<>(0);

    public LiveData<User> getUser() { return user; }          // expose immutable
    public LiveData<Integer> getCounter() { return counter; }

    public void loadUser() {
        user.setValue(new User("Alice", "alice@cityu.edu.hk"));   // main thread
        // from a background thread use: user.postValue(...)
    }
    public void increment() {
        Integer c = counter.getValue();
        counter.setValue(c == null ? 1 : c + 1);
    }
}
```
### View (Activity)
```java
public class MainActivity extends AppCompatActivity {
    private UserViewModel vm;

    @Override protected void onCreate(Bundle s) {
        super.onCreate(s);
        setContentView(R.layout.activity_main);
        TextView nameTV = findViewById(R.id.nameTV);
        TextView countTV = findViewById(R.id.countTV);

        vm = new ViewModelProvider(this).get(UserViewModel.class);

        vm.getUser().observe(this, u -> nameTV.setText(u.getName() + " / " + u.getEmail()));
        vm.getCounter().observe(this, c -> countTV.setText(String.valueOf(c)));

        findViewById(R.id.loadBtn).setOnClickListener(v -> vm.loadUser());
        findViewById(R.id.plusBtn).setOnClickListener(v -> vm.increment());
    }
}
```
Imports: `androidx.lifecycle.ViewModel`, `ViewModelProvider`, `LiveData`, `MutableLiveData`.

### AndroidViewModel (needs Context, e.g. Room/Prefs)
```java
public class NoteViewModel extends AndroidViewModel {
    public NoteViewModel(@NonNull Application app) { super(app); /* AppDatabase.getInstance(app) */ }
}
```

### Kotlin MVVM
```kotlin
class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>()
    val user: LiveData<User> = _user
    fun loadUser() { _user.value = User("Alice", "alice@cityu.edu.hk") }
}
// Activity
private val vm: UserViewModel by viewModels()      // needs androidx.activity:activity-ktx
// or: val vm = ViewModelProvider(this)[UserViewModel::class.java]
vm.user.observe(this) { u -> nameTV.text = u.name }
```

---
## MVP (Java)
```java
public interface UserContract {
    interface View { void showUser(String name, String email); void showError(String msg); }
    interface Presenter { void onLoadClicked(); }
}

public class UserPresenter implements UserContract.Presenter {
    private final UserContract.View view;
    private final UserModel model;
    public UserPresenter(UserContract.View v, UserModel m) { view = v; model = m; }
    @Override public void onLoadClicked() {
        User u = model.getUser();
        if (u != null) view.showUser(u.getName(), u.getEmail()); else view.showError("No user");
    }
}

public class MainActivity extends AppCompatActivity implements UserContract.View {
    private UserPresenter presenter;
    private TextView nameTV, emailTV;
    @Override protected void onCreate(Bundle s) {
        super.onCreate(s); setContentView(R.layout.activity_main);
        nameTV = findViewById(R.id.nameTV); emailTV = findViewById(R.id.emailTV);
        presenter = new UserPresenter(this, new UserModel());
        findViewById(R.id.loadBtn).setOnClickListener(v -> presenter.onLoadClicked());
    }
    @Override public void showUser(String n, String e) { nameTV.setText(n); emailTV.setText(e); }
    @Override public void showError(String m) { Toast.makeText(this, m, Toast.LENGTH_SHORT).show(); }
}
```

---
## MVI (Java, minimal)
```java
// Intent
public abstract class UserIntent { public static class LoadUser extends UserIntent {} }

// State (immutable)
public class UserState {
    public final boolean loading; public final User user; public final String error;
    public UserState(boolean l, User u, String e) { loading = l; user = u; error = e; }
    public static UserState idle() { return new UserState(false, null, null); }
}

// ViewModel
public class UserMviViewModel extends ViewModel {
    private final MutableLiveData<UserState> state = new MutableLiveData<>(UserState.idle());
    public LiveData<UserState> getState() { return state; }
    public void process(UserIntent intent) {
        if (intent instanceof UserIntent.LoadUser) {
            state.setValue(new UserState(true, null, null));
            state.setValue(new UserState(false, new User("Alice","a@x.com"), null));
        }
    }
}

// View
vm.getState().observe(this, st -> {
    progress.setVisibility(st.loading ? View.VISIBLE : View.GONE);
    if (st.user != null) nameTV.setText(st.user.getName());
    if (st.error != null) Toast.makeText(this, st.error, Toast.LENGTH_SHORT).show();
});
loadBtn.setOnClickListener(v -> vm.process(new UserIntent.LoadUser()));
```

---
## Repository pattern (UI → ViewModel → Repository → Room / Network)
```java
public class ColorRepository {
    private final ColorDao dao;
    private final ExecutorService io = Executors.newSingleThreadExecutor();
    public ColorRepository(Application app) { dao = ColorDatabase.getInstance(app).colorDao(); }
    public LiveData<List<ColorItem>> getAll() { return dao.getAllLive(); }
    public void insert(ColorItem c) { io.execute(() -> dao.insert(c)); }
}
```
