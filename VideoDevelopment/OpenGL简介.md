### OpenGL简介

OpenGL(Open Graphics Library开发图形接口)是一个跨平台的图形API，用于指定3D图形处理硬件中的标准软件接口。    

OpenGl的前身是SGI公司为其图形工作站开发的IRIS GL,后来因为IRIS GL的移植性不好，所以在其基础上，开发出了OpenGl。OpenGl一般用于在图形工作站，PC端使用，由于性能各方面原因，在移动端使用OpenGl基本带不动。为此，Khronos公司就为OpenGl提供了一个子集，OpenGl ES(OpenGl for Embedded System)。



### OpenGL ES

OpenGl ES是免费的跨平台的功能完善的2D/3D图形库接口的API,是OpenGL的一个子集。

移动端使用到的基本上都是OpenGl ES，当然Android开发下还专门为OpenGl提供了android.opengl包，并且提供了GlSurfaceView,GLU,GlUtils等工具类。



### OpenGL的作用



手机上做图像处理用很多方法，但是目前为止最高效的方法就是有效的使用图形处理单元(GPU)，图像的处理和渲染就是在将要渲染到窗口上的像素上做很多的浮点匀速，而GPU可以并行的做浮点运算，所以用GPU来分担CPU的部分，可以提高效率。































