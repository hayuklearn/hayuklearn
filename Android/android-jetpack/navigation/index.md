[TOC]

# Navigation

```
Navigation Graph

包含程序中所有的页面以及页面间的跳转关系。

NavHostFragment

可以认为它是一个特殊的 Fragment，是其他 Fragment 的容器。
Navigation Graph 中展示的页面和页面关系就是 NavHostFragment 中的内容，
可以认为 Navigation Graph 是可视化展示，但是 NavHostFragment 是对应的代码逻辑。

NavController

这是一个 Java/Kotlin 对象，用于完成在 Navigation Graph 中具体的页面切换工作。
当想切换 Fragment 时，使用 NavController 对象，告诉它你想去 Navigation Graph 中的哪个 Fragment，
NavController 会将你想去的 Fragment 展示在 NavHostFragment 中。
```

## Navigation 传参方式

- traditional bundle
- safe args plugin