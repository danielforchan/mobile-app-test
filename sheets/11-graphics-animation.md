# 11 · Graphics & Animation

[⬅ Index](../README.md)

## Shape drawable (circle): `res/drawable/circle.xml`
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="oval">
    <solid android:color="#7F00FFFF"/>   <!-- ARGB: 7F = 50% alpha -->
    <size android:width="200dp" android:height="200dp"/>
</shape>
```
Use it with `android:src="@drawable/circle"`, or as a TextView background with `android:background="@drawable/circle" android:gravity="center"`.

## Custom View + Canvas
```java
public class DrawView extends View {
    private final Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private float cx = 200, cy = 200;
    public DrawView(Context c) { super(c); }
    public DrawView(Context c, AttributeSet a) { super(c, a); }

    @Override protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.WHITE);
        paint.setColor(Color.RED);
        canvas.drawCircle(cx, cy, 60, paint);
        paint.setColor(Color.BLACK); paint.setTextSize(64);
        canvas.drawText("ANDROID", 50, 700, paint);
        canvas.drawRect(300, 300, 500, 450, paint);
        canvas.drawLine(0, 100, getWidth(), 100, paint);
    }

    @Override public boolean onTouchEvent(MotionEvent e) {
        if (e.getAction() == MotionEvent.ACTION_DOWN || e.getAction() == MotionEvent.ACTION_MOVE) {
            cx = e.getX(); cy = e.getY(); invalidate(); return true;
        }
        return super.onTouchEvent(e);
    }
}
```

## ViewPropertyAnimator (easiest)
```java
image.animate().scaleX(1.5f).scaleY(1.5f).setDuration(1000);
image.animate().rotation(360f).setDuration(1000);
image.animate().alpha(0f).setDuration(1000).withEndAction(() -> image.setVisibility(View.GONE));
image.animate().translationX(200f).setDuration(800);
// bounce
tv.animate().scaleX(1.3f).scaleY(1.3f).setDuration(100)
  .withEndAction(() -> tv.animate().scaleX(1f).scaleY(1f).setDuration(100));
```

## ObjectAnimator
```java
ObjectAnimator a = ObjectAnimator.ofFloat(view, "rotation", 0f, 360f);
a.setDuration(1000); a.setRepeatCount(ValueAnimator.INFINITE); a.start();
```

## Tween XML: `res/anim/view_animation.xml`
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <alpha android:fromAlpha="0.0" android:toAlpha="1.0" android:duration="1000"/>
    <rotate android:fromDegrees="0" android:toDegrees="360" android:pivotX="50%" android:pivotY="50%"
        android:startOffset="1000" android:duration="1000"/>
    <scale android:fromXScale="1" android:toXScale="2" android:fromYScale="1" android:toYScale="2"
        android:pivotX="50%" android:pivotY="50%" android:startOffset="2000" android:duration="1000"/>
</set>
```
```java
image.startAnimation(AnimationUtils.loadAnimation(this, R.anim.view_animation));
```

## Frame animation: `res/drawable/frames.xml`
```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android" android:oneshot="false">
    <item android:drawable="@drawable/f1" android:duration="300"/>
    <item android:drawable="@drawable/f2" android:duration="300"/>
</animation-list>
```
```java
image.setBackgroundResource(R.drawable.frames);
((AnimationDrawable) image.getBackground()).start();
```
