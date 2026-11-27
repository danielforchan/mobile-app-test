# 11 · Graphics & Animation

[⬅ Index](../README.md)

## Shape drawable XML (circle, semi-transparent)
`res/drawable/circle.xml`
```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="oval">
    <solid android:color="#7F00FFFF"/>   <!-- ARGB: 7F = 50% alpha -->
    <size android:width="200dp" android:height="200dp"/>
</shape>
```
```xml
<ImageView android:layout_width="200dp" android:layout_height="200dp" android:src="@drawable/circle"/>
<!-- or as background of a TextView: android:background="@drawable/circle" android:gravity="center" -->
```

## ShapeDrawable in code
```java
ShapeDrawable oval = new ShapeDrawable(new OvalShape());   // RectShape(), PathShape(...)
oval.getPaint().setColor(Color.CYAN);
oval.setIntrinsicWidth(200); oval.setIntrinsicHeight(200);
oval.setAlpha(127);
ImageView iv = new ImageView(this);
iv.setImageDrawable(oval);
rootLayout.addView(iv);         // add a view in code
```

## Custom View with Canvas
```java
public class DrawView extends View {
    private final Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
    private float cx = 200, cy = 200;

    public DrawView(Context c) { super(c); }
    public DrawView(Context c, AttributeSet a) { super(c, a); }   // needed for XML use

    @Override protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.WHITE);

        paint.setColor(Color.BLUE); paint.setStrokeWidth(5);
        for (int i = 1; i <= 5; i++) {
            canvas.drawLine(i * 100, 0, i * 100, getHeight(), paint);   // vertical
            canvas.drawLine(0, i * 100, getWidth(), i * 100, paint);    // horizontal
        }
        paint.setColor(Color.RED);
        canvas.drawCircle(cx, cy, 60, paint);

        paint.setColor(Color.BLACK); paint.setTextSize(64);
        canvas.drawText("ANDROID", 50, 700, paint);

        canvas.drawRect(300, 300, 500, 450, paint);
    }

    @Override public boolean onTouchEvent(MotionEvent e) {   // drag circle
        if (e.getAction() == MotionEvent.ACTION_DOWN || e.getAction() == MotionEvent.ACTION_MOVE) {
            cx = e.getX(); cy = e.getY();
            invalidate();               // triggers onDraw again
            return true;
        }
        return super.onTouchEvent(e);
    }
}
```
Use it with `setContentView(new DrawView(this));` or in XML: `<com.example.app.DrawView android:layout_width="match_parent" android:layout_height="match_parent"/>`.

---
## ViewPropertyAnimator (easiest)
```java
image.animate().scaleX(1.5f).scaleY(1.5f).setDuration(4000);          // grow
image.animate().rotation(360f).setDuration(1000);
image.animate().alpha(0f).setDuration(1000).withEndAction(() -> image.setVisibility(View.GONE));
image.animate().translationX(200f).translationY(100f).setDuration(800);
// bounce (e.g. counter number)
tv.animate().scaleX(1.3f).scaleY(1.3f).setDuration(100)
  .withEndAction(() -> tv.animate().scaleX(1f).scaleY(1f).setDuration(100));
```

## ObjectAnimator
```java
ObjectAnimator a = ObjectAnimator.ofFloat(view, "rotation", 0f, 360f);
a.setDuration(1000); a.setRepeatCount(ValueAnimator.INFINITE); a.start();
```

## Tween animation XML: `res/anim/view_animation.xml`
```xml
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:interpolator="@android:anim/accelerate_decelerate_interpolator">
    <alpha android:fromAlpha="0.0" android:toAlpha="1.0" android:duration="2000"/>
    <rotate android:fromDegrees="0" android:toDegrees="360"
        android:pivotX="50%" android:pivotY="50%" android:startOffset="2000" android:duration="2000"/>
    <scale android:fromXScale="1" android:toXScale="2" android:fromYScale="1" android:toYScale="2"
        android:pivotX="50%" android:pivotY="50%" android:startOffset="4000" android:duration="2000"/>
    <translate android:fromXDelta="0" android:toXDelta="100" android:startOffset="6000" android:duration="1000"/>
</set>
```
```java
Animation anim = AnimationUtils.loadAnimation(this, R.anim.view_animation);
image.startAnimation(anim);
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
AnimationDrawable ad = (AnimationDrawable) image.getBackground();
ad.start();     // start from a click or onWindowFocusChanged
```

## TransitionDrawable (fade between 2): `res/drawable/shape_transition.xml`
```xml
<transition xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/cyan_shape"/>
    <item android:drawable="@drawable/magenta_shape"/>
</transition>
```
```java
TransitionDrawable td = (TransitionDrawable) ContextCompat.getDrawable(this, R.drawable.shape_transition);
image.setImageDrawable(td);
td.startTransition(5000);
```
