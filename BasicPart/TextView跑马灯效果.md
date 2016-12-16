TextView跑马灯效果
===

TextView跑马灯效果实现方式一:
---

当`TextView`内容过多时默认会采用截取的方式以`...`来截取。如何能够实现内容过多时的跑马灯效果。
 
自定义视图步骤:
----

1. 自定义一个类继承`TextView`,重写它的`isFocused()`方法
2. 在布局的文件中使用自定义的`TextView`
 
 
示例代码：
----

1. 继承TextView
    ```java
    //继承TextView并且实现抽象方法
    public class FocusedTextView extends TextView {
     
        public FocusedTextView(Context context, AttributeSet attrs, int defStyle){
            super(context, attrs, defStyle);
        }
     
        public FocusedTextView(Context context, AttributeSet attrs) {
            super(context, attrs);
        }
     
        public FocusedTextView(Context context) {
            super(context);
        }
     
        //重写isFocused方法，让其一直返回true
        public boolean isFocused() {
            return true;
        }
    }
    ``` 

2. 在清单文件中使用该类
```xml
<com.charon.test.ui.FocusedTextView   //注意这里要使用包名.类名
    android:ellipsize="marquee"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:singleLine="true"
    android:text="我是你的嘎嘎嘎、、、、、、、、哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈" />
 ```

TextView跑马灯效果实现方式二:
---

TextView实现跑马灯的效果,不用自定义View
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:ellipsize="marquee"
    android:focusable="true"
    android:focusableInTouchMode="true"
    android:marqueeRepeatLimit="marquee_forever"
    android:singleLine="true"
    android:text="测试啊啊啊啊啊啊啊啊ggggggggggggggggggggggggggggg" />
```

---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
