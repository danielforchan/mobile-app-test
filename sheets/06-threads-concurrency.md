# 06 · Threads, Handler, Executor, AsyncTask, Coroutines

[⬅ Index](../README.md) · Full example with picture: [T4](../templates/T4-progress-handler.md)

**Two rules:**
1. Never block the UI thread (no sleep, network, or DB on main).
2. Never touch Views from a background thread. You get `CalledFromWrongThreadException`.

## Ways to get back to the UI thread
```java
view.post(() -> imageView.setImageBitmap(bmp));
runOnUiThread(() -> statusTV.setText("Done"));
new Handler(Looper.getMainLooper()).post(() -> statusTV.setText("Done"));
```

## 1. Plain Thread + post
```java
loadBtn.setOnClickListener(v -> new Thread(() -> {
    try { Thread.sleep(5000); } catch (InterruptedException e) { e.printStackTrace(); }
    Bitmap bmp = BitmapFactory.decodeResource(getResources(), R.drawable.painter);
    imageView.post(() -> imageView.setImageBitmap(bmp));
}).start());
```
⚠️ `thread.run()` does NOT start a new thread. Use `start()`.

## 2. Executor + Handler (recommended Java combo)
```java
private final ExecutorService executor = Executors.newSingleThreadExecutor();
private final Handler mainHandler = new Handler(Looper.getMainLooper());

private void loadIcon() {
    progressBar.setVisibility(View.VISIBLE);
    executor.execute(() -> {
        for (int i = 1; i <= 10; i++) {
            try { Thread.sleep(500); } catch (InterruptedException e) { return; }
            final int p = i * 10;
            mainHandler.post(() -> progressBar.setProgress(p));   // like publishProgress
        }
        Bitmap bmp = BitmapFactory.decodeResource(getResources(), R.drawable.painter);
        mainHandler.post(() -> {                                    // like onPostExecute
            imageView.setImageBitmap(bmp);
            progressBar.setVisibility(View.INVISIBLE);
        });
    });
}

@Override protected void onDestroy() {
    super.onDestroy();
    mainHandler.removeCallbacksAndMessages(null);
    executor.shutdownNow();
}
```

## 3. Handler postDelayed: timers / repeating tasks
```java
private final Handler handler = new Handler(Looper.getMainLooper());
private int seconds = 0;
private boolean running = false;

private final Runnable tick = new Runnable() {
    @Override public void run() {
        seconds++;
        timeTV.setText(String.format(Locale.US, "%02d:%02d", seconds / 60, seconds % 60));
        handler.postDelayed(this, 1000);
    }
};

startBtn.setOnClickListener(v -> { if (!running) { running = true; handler.postDelayed(tick, 1000); } });
stopBtn.setOnClickListener(v -> { running = false; handler.removeCallbacks(tick); });
resetBtn.setOnClickListener(v -> { running = false; handler.removeCallbacks(tick); seconds = 0; timeTV.setText("00:00"); });
```
Countdown alternative:
```java
new CountDownTimer(30000, 1000) {
    public void onTick(long ms) { timeTV.setText("" + ms / 1000); }
    public void onFinish() { timeTV.setText("Done!"); }
}.start();
```

## 4. AsyncTask (deprecated, but in the slides)
```java
private class LoadIconTask extends AsyncTask<Integer, Integer, Bitmap> {
    @Override protected void onPreExecute() { progressBar.setVisibility(View.VISIBLE); }
    @Override protected Bitmap doInBackground(Integer... resId) {           // background
        for (int i = 1; i <= 10; i++) { sleep(500); publishProgress(i * 10); }
        return BitmapFactory.decodeResource(getResources(), resId[0]);
    }
    @Override protected void onProgressUpdate(Integer... v) { progressBar.setProgress(v[0]); } // UI
    @Override protected void onPostExecute(Bitmap b) {                        // UI
        progressBar.setVisibility(View.INVISIBLE); imageView.setImageBitmap(b);
    }
    private void sleep(long ms) { try { Thread.sleep(ms); } catch (InterruptedException ignored) {} }
}
// start
new LoadIconTask().execute(R.drawable.painter);
```

## 5. Kotlin Coroutines
Deps: `implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")` and `implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.6.2")`.
```kotlin
loadBtn.setOnClickListener {
    lifecycleScope.launch {                 // Main, auto-cancelled on destroy
        progressBar.visibility = View.VISIBLE
        val bmp = withContext(Dispatchers.IO) {        // background
            for (i in 1..10) {
                delay(500)
                withContext(Dispatchers.Main) { progressBar.progress = i * 10 }
            }
            BitmapFactory.decodeResource(resources, R.drawable.painter)
        }
        imageView.setImageBitmap(bmp)        // back on Main
        progressBar.visibility = View.INVISIBLE
    }
}
```
- Dispatchers: **Main** (UI), **IO** (network/file/DB), **Default** (CPU)
- Builders: `launch` (fire & forget), `async {}` + `await()` (returns value), `runBlocking` (tests only)
- In ViewModel: `viewModelScope.launch { }`

## Singleton pattern (thread-safe)
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
