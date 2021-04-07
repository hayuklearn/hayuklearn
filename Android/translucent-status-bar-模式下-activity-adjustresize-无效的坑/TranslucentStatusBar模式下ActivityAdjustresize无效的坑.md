## 背景

一般情况下通过 AndroidManifest.xml 注册 Activity 时设置 `android:windowSoftInputMode="adjustResize"` 可以让软键盘与 Activity 界面的高度自适应，但是在 translucent status bar 的情况下无效。

参考文章：
[参考一](https://blog.csdn.net/u010255127/article/details/49308775)
[参考二](https://blog.csdn.net/qq_24737357/article/details/51645032)

## 解决

自定义 RelativeLayout
```java
public class TranslucentResizeRelativeLayout extends RelativeLayout {

    private int[] mInsets = new int[4];

    public TranslucentResizeRelativeLayout(Context context) {
        super(context);
    }

    public TranslucentResizeRelativeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public TranslucentResizeRelativeLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public TranslucentResizeRelativeLayout(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);
    }

    @Override
    public final WindowInsets onApplyWindowInsets(WindowInsets insets) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT_WATCH) {
            mInsets[0] = insets.getSystemWindowInsetLeft();
            mInsets[1] = insets.getSystemWindowInsetTop();
            mInsets[2] = insets.getSystemWindowInsetRight();
            return super.onApplyWindowInsets(insets.replaceSystemWindowInsets(0, 0, 0, insets.getSystemWindowInsetBottom()));
        } else {
            return insets;
        }
    }
}
```

布局
```xml
<com.your.package.TranslucentResizeRelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true">

</com.your.package.TranslucentResizeRelativeLayout>
```