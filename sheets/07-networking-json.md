# 07 · Networking + JSON

[⬅ Index](../README.md)

## Steps
1. Manifest: `<uses-permission android:name="android.permission.INTERNET"/>`
2. Run network calls on a **background thread**, or you get `NetworkOnMainThreadException`.
3. Update the UI on the main thread.
4. Plain `http://`: add `android:usesCleartextTraffic="true"`.

## HttpURLConnection GET
```java
executor.execute(() -> {
    HttpURLConnection conn = null;
    try {
        conn = (HttpURLConnection) new URL(urlStr).openConnection();
        conn.setRequestMethod("GET");
        conn.setConnectTimeout(10000);
        conn.setReadTimeout(10000);
        if (conn.getResponseCode() != HttpURLConnection.HTTP_OK) throw new IOException("HTTP " + conn.getResponseCode());

        BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream(), StandardCharsets.UTF_8));
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
```

## Parse JSON (org.json is built in)
```java
try {
    JSONObject root = new JSONObject(json);
    String city = root.getString("city");
    double temp = root.getDouble("temp");
    String opt = root.optString("missing", "N/A");
    JSONArray days = root.getJSONArray("days");
    for (int i = 0; i < days.length(); i++) {
        JSONObject d = days.getJSONObject(i);
        String day = d.getString("d");
    }
} catch (JSONException e) { e.printStackTrace(); }
```
If the top level is an array `[...]`, use `new JSONArray(json)`.

## Build JSON (also handy for saving a list into SharedPreferences)
```java
JSONArray arr = new JSONArray();
JSONObject o = new JSONObject();
o.put("name", "Alice");
o.put("count", 3);
arr.put(o);
String s = arr.toString();
```

## Kotlin
```kotlin
lifecycleScope.launch {
    val json = withContext(Dispatchers.IO) { URL(urlStr).readText() }
    resultTV.text = JSONObject(json).getString("city")
}
```

Test API (no key needed): `https://jsonplaceholder.typicode.com/todos/1`
