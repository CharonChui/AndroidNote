通过Hardware Layer提高动画性能
===

项目中越来越多的动画，越来越多的效果导致了应用性能越来越低。该如何提升。   

### 简介 

在`View`播放动画的过程中每一帧都需要被重绘。如果使用`view layers`，就不用每帧都去重绘，因为`View`渲染一旦离开屏幕缓冲区就可以被重用。

而且，`hardware layers`会在`GPU`上缓存，这样就会让一些动画过程中的操作变得更快。通过`hardware layers`可以快速的渲染一些简单的转变(位移、选中、缩放、颜色渐变)。由于很多动画都是这些动作的结合，所以`hardware layers`可以显著的提高动画性能。

在`View`当中提供了三种类型的`Layer type`:    

- LAYER_TYPE_HARDWARE
    > Indicates that the view has a hardware layer. A hardware layer is backed by a hardware specific texture (generally Frame Buffer Objects or FBO on OpenGL hardware) and causes the view to be rendered using Android's hardware rendering pipeline, but only if hardware acceleration is turned on for the view hierarchy. When hardware acceleration is turned off, hardware layers behave exactly as software layers.

    > A hardware layer is useful to apply a specific color filter and/or blending mode and/or translucency to a view and all its children.

    > A hardware layer can be used to cache a complex view tree into a texture and reduce the complexity of drawing operations. For instance, when animating a complex view tree with a translation, a hardware layer can be used to render the view tree only once.

    > A hardware layer can also be used to increase the rendering quality when rotation transformations are applied on a view. It can also be used to prevent potential clipping issues when applying 3D transforms on a view.    
    

- LAYER_TYPE_SOFTWARE
    > Indicates that the view has a software layer. A software layer is backed by a bitmap and causes the view to be rendered using Android's software rendering pipeline, even if hardware acceleration is enabled.

    > Software layers have various usages:

    > When the application is not using hardware acceleration, a software layer is useful to apply a specific color filter and/or blending mode and/or translucency to a view and all its children.

    > When the application is using hardware acceleration, a software layer is useful to render drawing primitives not supported by the hardware accelerated pipeline. It can also be used to cache a complex view tree into a texture and reduce the complexity of drawing operations. For instance, when animating a complex view tree with a translation, a software layer can be used to render the view tree only once.

    > Software layers should be avoided when the affected view tree updates often. Every update will require to re-render the software layer, which can potentially be slow (particularly when hardware acceleration is turned on since the layer will have to be uploaded into a hardware texture after every update.)

- LAYER_TYPE_NONE
    > Indicates that the view does not have a layer.
    默认值。

### 使用

首先使用的前提是在清单文件中开启了硬件加速。否则将无法使用`hardware layer`。这一点在上面的文档中也有说明。

`API`也是非常简单的，直接使用`View.setLayerType()`就好。使用时应该只是暂时的设置`Hardware Layer`，因为它们无法自动释放。     
基本的使用步骤：    
 
- 对每个想要在动画过程中进行缓存的`view`调用`View.setLayerType(View.LAYER_TYPE_HARDWARE, null)`方法。
- 执行动画。
- 在动画执行结束后调用`View.setLayerType(View.LAYER_TYPE_NONE, null)`方法来进行清除。

示例:      
```java
mView.setLayerType(View.LAYER_TYPE_HARDWARE, null);

animator.addListener(new AnimatorListenerAdapter() {  
  @Override
  public void onAnimationEnd(Animator animation) {
    mView.setLayerType(View.LAYER_TYPE_NONE, null);
  }
});

animator.start();  

```
但是如果在`4.0.x`的版本中使用上面的代码会崩溃，必须要把`setLayerType`放到`Runnable`中。如下:   
```java
mView.setLayerType(View.LAYER_TYPE_HARDWARE, null);

animator.addListener(new AnimatorListenerAdapter() {  
  @Override
  public void onAnimationEnd(Animator animation) {
    //This will work successfully
    post(new Runnable() {
        @Override
        public void run () {
            setLayerType(LAYER_TYPE_NONE, null);
        }
    }
  }
});

animator.start();  
```

如果你基于`minSdkVersion 16`以上并且使用`ViewPropertyAnimator`时，你可以使用`withLayer()`方法替代如上的操作:     

```java
mView.animate().translationX(150).withLayer().start();
```

或者在`api 14`以上时使用`ViewCompat.animate().withLayer()`
这样做，你的动画就会变得更流畅！
    	

### 注意事项  

你应该知道，事情没那么简单。     
`Hardware layers`有着惊人的提升动画性能的能力。然而，如果滥用，它的危害更大。**不要盲目的使用`layers`**

- 首先，在有些情况下，`hardware layers`除了`view`渲染外还会执行更多的工作。缓存`layer`将会需要时间，因为首选第一步就需要两个过程: 先将这些`view`渲染到`GPU`的一个`layer`中然后`GPU`再渲染该`layer`到`Window`上。如果要渲染的`View`非常简单(例如一个纯色值),那么这样在初始化的时候就会增加`Hardware Layer`不必要的开销。

- 其次，对所有的缓存来讲，都有一个缓存失效的可能性。任何时候如果在动画过程中调用`view.invalidate()`，那么`layer`就必须要重新渲染。经常的废弃`hardware layers`会比没有`layers`的情况下更糟糕，因为如同上面讲到的`hardware layers`在设置缓存时会有额外的开销。如果你需要经常的重新缓存`layer`，那就会有极大的损害。    

    这个问题也是非常容易出现的，因为动画经常有多个移动的部分。假如现在有一个三个部分移动的动画:     
    ```java
    Parent ViewGroup
    —-> Child View1 (往左移动)
    —-> Child View2 (往右移动)
    —-> Child View3 (往上移动)
    ```
    
    如果你只在父布局`ViewGroup`上设置一个`layer`，那就将经常的缓存失效，因为`ViewGroup`会随着子`View`不断地改变。然而对每个单独的子`Views`而言，他们只是在位移。这种情况下，最好是对每个子`View上`设置`Hardware Layer`（而不是在父布局上）。
    
    ***再次重申，通常是对多个子`View上`适当的设置`Hardware Layer`，这样他们就不会在动画运行时失效。***
    
    在手机开发者选项中的*显示硬件层更新（Show hardware layers updates）*功能是追踪这个问题的开发利器。当`View`渲染`Hardware Layer`的时候闪烁绿色，它应该在动画开始的时候闪烁一次（也就是`Layer`渲染初始化的时候），然而，如果你的`View`在整个动画期间都是绿色，那就是遇到失效的问题了。

- 最后，`hardware layers`使用`GPU`内存，你当然不想出现内存泄漏的问题。所以你应该在必要的时候再去使用`hardware layers`，就想播放动画时。  


这里也没有硬性规则。`Android`渲染系统是非常复杂的。就像所有性能问题一样，测试才是关键。通过使用“显示硬件层更新”开发者选项来确定`layers`是在帮你还是害你。





---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 