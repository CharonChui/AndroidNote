LayoutInflater.inflate详解
===

`LayoutInflater`概述    
---

从`XML`文件中实例化一个布局成对应的`View`类， 它从来不会直接使用， 而是使用`getLayoutInflater()`或者`getSystemService(String)`来获得一个对应当前`context`的标准`LayoutInflater`
实例。 

例如：　　　　
```java
LayoutInflater inflater = (LayoutInflater)context.getSystemService
      (Context.LAYOUT_INFLATER_SERVICE);
```               

如果要在自己的`views`中通过`LayoutInflater.Factory`来创建`LayoutInflater`你可以用`cloneInContext(Context)`来克隆一个当前存在的`ViewFactory`然后再用`setFactory(LayoutInfalter.FActory)`
来设置成自己的`FActory`.	              


起因
---

原来在项目中一直习惯用`inflater.inflate(R.layout.layout, null)`最近却发现会有一个黄色的警告。　十分好奇，所以决定找出原因。      

我们发现`inflate`方法有两个：     
- View inflate(int resource, ViewGroup root)
- View inflate(int resource , ViewGroup root, boolean attachToRoot)
第二个参数是指实例的布局所要放入的跟视图。        
一般我们在不需要将该布局放入跟视图的时候都会把第二个参数传为`null`，这样系统就不会去进行相应的绑定操作了，不然就蹦了。 我相信很多人都会这样理解，所以都很少用到
第二个方法， 其实这样是错误的。    

原因在于`android:layout_xyz`属性会在父视图中重新计算，而你用第一个方法是说需要被添加到父视图中，只不过你不知道添加到哪一个父视图中， 所以你在`xml`中定义的`LayoutParams`
就会被忽略(因为没有已知的父视图)。     

示例
---

大家肯定遇到过在`ListView`的`item`布局中设置的高度没有效果的问题。 
```java
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="100dip"
    android:gravity="center_vertical"
    android:orientation="horizontal">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="test" />
</LinearLayout>
```

```java
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
        convertView = inflate(R.layout.item_lv_test, null);
    }
    return convertView;
}
```

如果用上面的代码会发现设置`100dp`是无效的。     

而如果换成下面的代码就可以了。
```java
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
        convertView = inflate(R.layout.item_lv_test, null, false);
    }
    return convertView;
}
```
这里你该会想一想为什么很多需要显示`View`的方法中都有`ViewGroup`这个参数。      
所以有些人会说在跟布局中设置是无效的，要再嵌套一层布局。 这样是错误的，会造成布局层级增多，影响性能。 


细节
---

- `setContentView()`与`LayoutInfalter.inflate()`的区别
    `setContentView()`一旦调用，就会立即显示`UI`. 而`LayoutInfalter.inflate()`这是把布局转换成对应的`View`对象，不会立即显示，只有需要的时候再显示出来。   
	
- `View.inflate()`方法与`LayoutInflater.inflate()`的区别
    直接上源码：     
	```java
	public static View inflate(Context context, int resource, ViewGroup root) {
        LayoutInflater factory = LayoutInflater.from(context);
        return factory.inflate(resource, root);
    }
	```

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
