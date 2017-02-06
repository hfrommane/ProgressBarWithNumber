## 开发
设计搞了一个带圆形进度的进度条，在GitHub上逛了一圈，发现没有，自己撸吧。

先看界面效果：

![](https://raw.githubusercontent.com/hfrommane/ProgressBarWithNumber/master/images/%E8%87%AA%E5%AE%9A%E4%B9%89%E8%BF%9B%E5%BA%A6%E6%9D%A1.gif)

主要思路是写一个继承ProgressBar的自定义View。

## 使用
在布局文件中加入：
```
<com.fun.progressbarwithnumber.HorizontalProgressBarWithNumber
        android:id="@+id/hpbwn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        fun:progress_circle_color="#ff000000"
        fun:progress_reached_bar_height="20dp"
        fun:progress_reached_color="#FFFF4081"
        fun:progress_text_color="#ffffffff"
        fun:progress_text_size="14sp"
        fun:progress_unreached_bar_height="20dp"
        fun:progress_unreached_color="#ffBCB4E8" />
```

 1. progress_reached_bar_height：当前进度的高度
 2. progress_unreached_bar_height：剩余进度的高度
 3. progress_text_size：圆圈内文字的大小

**注意：**
当前进度和剩余进度的**高度要一致**，圆圈大小和圆圈内文字的大小要**配合Java代码调整**。

如果您喜欢在Github上给个Star，如果转载请注明出处，感谢。

吼吼吼，就这样。