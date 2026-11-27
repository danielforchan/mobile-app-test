# 07 · Networking + JSON

[⬅ Index](../README.md)

## Steps
1. Manifest: `<uses-permission android:name="android.permission.INTERNET"/>`
2. Run the network call on a **background thread**, or you get `NetworkOnMainThreadException`.
3. Update the UI on the main thread.
4. Plain `http://` URL? Add `android:usesCleartextTraffic="true"` to `<application>`.
5. From the emulator, your PC's localhost is `http://10.0.2.2:PORT/`.

## HttpURLConnection GET (Java, no library)
```java
private final ExecutorService executor = Executors.newSingleThreadExecutor();
private final Handler main = new Handler(Looper.getMainLooper());

private void fetch(String urlStr) {
    executor.execute(() -> {
        HttpURLConnection conn = null;
        try {
            URL url = new URL(urlStr);
            conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            conn.setConnectTimeout(10000);
            conn.setReadTimeout(10000);

            int code = conn.getResponseCode();
            if (code != HttpURLConnection.HTTP_OK) throw new IOException("HTTP " + code);

            BufferedReader br = new BufferedReader(
                    new InputStreamReader(conn.getInputStream(), StandardCharsets.UTF_8));
            StringBuilder sb = new StringBuilder();
            String line;
            while ((line = br.readLine()) != null) sb.append(line);
            br.close();

            String json = sb.toString();
            main.post(() -> showResult(json));
        } catch (Exception e) {
            main.post(() -> Toast.makeText(this, "Error: " + e.getMessage(), Toast.LENGTH_LONG).show());
        } finally {
            if (conn != null) conn.disconnect();
        }
    });
}
```

## POST JSON
```java
conn.setRequestMethod("POST");
conn.setDoOutput(true);
conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
JSONObject body = new JSONObject();
body.put("name", "Daniel");
try (OutputStream os = conn.getOutputStream()) {
    os.write(body.toString().getBytes(StandardCharsets.UTF_8));
}
int code = conn.getResponseCode();
```

## Parse JSON (`org.json`, built into Android)
Example: `{"city":"HK","temp":23.5,"days":[{"d":"Mon","t":24},{"d":"Tue","t":22}]}`
```java
try {
    JSONObject root = new JSONObject(json);
    String city = root.getString("city");
    double temp = root.getDouble("temp");
    String opt  = root.optString("missing", "N/A");   // safe default

    JSONArray days = root.getJSONArray("days");
    List<String> out = new ArrayList<>();
    for (int i = 0; i < days.length(); i++) {
        JSONObject d = days.getJSONObject(i);
        out.add(d.getString("d") + ": " + d.getInt("t") + "°C");
    }
} catch (JSONException e) { e.printStackTrace(); }
```
Top-level array `[...]` → `new JSONArray(json)`.

## Build JSON
```java
JSONObject o = new JSONObject();
o.put("name", "Alice");
o.put("age", 20);
JSONArray arr = new JSONArray(); arr.put("a"); arr.put("b");
o.put("tags", arr);
String s = o.toString();
```

## Check connectivity (optional, needs ACCESS_NETWORK_STATE)
```java
ConnectivityManager cm = (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkCapabilities nc = cm.getNetworkCapabilities(cm.getActiveNetwork());
boolean online = nc != null && nc.hasCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET);
```

## Kotlin + coroutines
```kotlin
lifecycleScope.launch {
    val json = withContext(Dispatchers.IO) { URL(urlStr).readText() }
    val obj = JSONObject(json)
    resultTV.text = obj.getString("city")
}
```

## Load image from URL
```java
executor.execute(() -> {
    try (InputStream in = new URL(imgUrl).openStream()) {
        Bitmap b = BitmapFactory.decodeStream(in);
        main.post(() -> imageView.setImageBitmap(b));
    } catch (IOException e) { e.printStackTrace(); }
});
```

## Public test APIs (no key)
- `https://jsonplaceholder.typicode.com/todos/1`
- `https://jsonplaceholder.typicode.com/users`
