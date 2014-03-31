PopupWindow响应back按键并关闭
===

`PopupWindow`跟我们的`Activity`不一样，因为我们在构造时往往不是继承来的,而是`new`出来的。所以不能使用重写`onKeyDown()`之类的方法来截获键盘事件。

1. 最简单        
    在`new`的时候，使用下面的方法：
	`new PopupWindow(view, LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, true);`关键在于最后一个参数,`Focusable`属性,让它能够接受焦点。      
当然你也可以手动设置它：      
`PW.setFocusable(true);`    
`PW.setFocusableInTouchMode(true);`  //为了保险起见加上这句        
此时实际上还是不能起作用的，百度给出的结论是，必须加入下面这行作用未知的语句才能发挥作用：        
`pw.setBackgroundDrawable(new BitmapDrawable());` // 响应返回键必须的语句       
请放心，设置 `BackgroundDrawable`并不会改变你在配置文件中设置的背景颜色或图像。

2. 最通用        
    首先在布局文件`(*.xml)`中随意选取一个不影响任何操作的`View`，推荐使用最外层的`Layout`。
然后设置该`Layout`的`Focusable`和`FocusableInTouchMode`都为`true`。
接着回到代码中对该`View`重写`OnKeyListener()`事件了。捕获`KEYCODE_BACK`给对话框`dismiss()`。
给出一段示例：        
    ```java
    private PopupWindow pw;
    private View view;
    private LinearLayout layMenu;
     
    LayoutInflater inflater = (LayoutInflater) main.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    view = inflater.inflate(R.layout.popup_main_menu, null, false);
    layMenu = (LinearLayout) view.findViewById(R.id.layMenu);
    pw = new PopupWindow(view, LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT, true);
     
    layMenu.setOnKeyListener(new OnKeyListener()
    {
        public boolean onKey(View v, int keyCode, KeyEvent event)
        {
            if (event.getAction() == KeyEvent.ACTION_DOWN && keyCode == KeyEvent.KEYCODE_BACK)
                pw.dismiss();
     
            return false;
        }
    });
    ```
上面两种方法的代码可以并存，当然如果使用第一种方法的话就不需要再捕获返回键了，不过你可以尝试捕获其他你需要的按键

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 