[TOC]

# Android 事件分发

## `dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`的关系

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
  boolean consume = false;
  if (onInterceptTouchEvent(ev)) {
    consume = onTouchEvent(ev);
  } else {
    consume = child.dispatchTouchEvent(ev);
  }
  return consume;
}

// 任玉刚. Android开发艺术探索 (Kindle 位置 2721-2725). 电子工业出版社. Kindle 版本. 
```