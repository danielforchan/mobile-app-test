# 06 · Threads, Handler, Executor, AsyncTask, Coroutines

[⬅ Index](../README.md) · Full example: [T4](../templates/T4-progress-handler.md)

**Two rules:**
1. Never block the UI thread (no sleep, network or DB work on main).
2. Never touch Views from a background thread. That throws `CalledFromWrongThreadException`.

## Back to the UI thread
```java
view.post(() -> imageView.setImageBitmap(bmp));
runOnUiThread(() -> statusTV.setText("Done"));
new Handler(Looper.getMainLooper()).post(() -> statusTV.setText("Done"));
```

## Executor + Handler (recommended)
```java
private final ExecutorService executor = Executors.newSingleThreadExecutor();
private final Handler main = new Handler(Looper.getMainLooper());

executor.execute(() -> {
    for (int i = 1; i <= 10; i++) {
        try { Thread.sleep(500); } catch (InterruptedException e) { return; }
        final int p = i * 10;
        main.post(() -> progressBar.setProgress(p));
    }
    main.post(() -> statusTV.setText("Done"));
});

@Override protected void onDestroy() {
    super.onDestroy();
    main.removeCallbacksAndMessages(null);
    executor.shutdownNow();
}
```

## Plain Thread
```java
new Thread(() -> {
    // background work
    runOnUiThread(() -> tv.setText("done"));
}).start();      // start(), NOT run()
```

## Handler postDelayed: timer
```java
private final Handler handler = new Handler(Looper.getMainLooper());
private int seconds = 0;
private final Runnable tick = new Runnable() {
    @Override public void run() {
        seconds++;
        timeTV.setText(String.format(Locale.US, "%02d:%02d", seconds / 60, seconds % 60));
        handler.postDelayed(this, 1000);
    }
};
// start: handler.postDelayed(tick, 1000);   stop: handler.removeCallbacks(tick);
```
Countdown:
```java
new CountDownTimer(30000, 1000) {
    public void onTick(long ms) { timeTV.setText("" + ms / 1000); }
    public void onFinish() { timeTV.setText("Done!"); }
}.start();
```

## AsyncTask (deprecated, but in the slides)
```java
private class LoadTask extends AsyncTask<Integer, Integer, Bitmap> {
    @Override protected void onPreExecute() { progressBar.setVisibility(View.VISIBLE); }
    @Override protected Bitmap doInBackground(Integer... resId) {
        for (int i = 1; i <= 10; i++) {
            try { Thread.sleep(500); } catch (InterruptedException ignored) {}
            publishProgress(i * 10);
        }
        return BitmapFactory.decodeResource(getResources(), resId[0]);
    }
    @Override protected void onProgressUpdate(Integer... v) { progressBar.setProgress(v[0]); }
    @Override protected void onPostExecute(Bitmap b) { progressBar.setVisibility(View.INVISIBLE); imageView.setImageBitmap(b); }
}
// new LoadTask().execute(R.drawable.painter);
```

## Kotlin Coroutines
```kotlin
lifecycleScope.launch {
    progressBar.visibility = View.VISIBLE
    val data = withContext(Dispatchers.IO) { dao.getAll() }   // background
    adapter.setData(data)                                     // back on Main
    progressBar.visibility = View.GONE
}
```
- **Dispatchers:** Main (UI), IO (network/file/DB), Default (CPU-heavy work)
- **In a ViewModel:** `viewModelScope.launch { }`

## Singleton (thread-safe)
```java
public class Counter {
    private static volatile Counter instance;
    private Counter() {}
    public static Counter getInstance() {
        if (instance == null) {
            synchronized (Counter.class) {
                if (instance == null) instance = new Counter();
            }
        }
        return instance;
    }
}
```
