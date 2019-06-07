### Context
#### 什么是Context
Context是一个抽象基类，我们通过它访问当前包的资源（getResources、getAssets）和启动其他组件（Activity、Service、Broadcast）以及得到各种服务（getSystemService），当然，通过Context能得到的不仅仅只有上述这些内容。对Context的理解可以来说：Context提供了一个应用的运行环境，在Context的大环境里，应用才可以访问资源，才能完成和其他组件、服务的交互，Context定义了一套基本的功能接口，我们可以理解为一套规范，而Activity和Service是实现这套规范的子类，这么说也许并不准确，因为这套规范实际是被ContextImpl类统一实现的，Activity和Service只是继承并有选择性地重写了某些规范的实现。

Context作为一个抽象的基类，它的实现子类有三种：**Application**、**Activity**和**Service**,那么他们的区别是什么呢？
![image](http://img.blog.csdn.net/20140322224357343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luZ3doYXRpd2FubmE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![image](http://img.blog.csdn.net/20140322224928671?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2luZ3doYXRpd2FubmE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过对比可以清晰地发现，Service和Application的类继承关系比较像，而Activity还多了一层继承ContextThemeWrapper，这是因为Activity有主题的概念，而Service是没有界面的服务，Application更是一个抽象的东西，它也是通过Activity类呈现的。

Context的真正实现都在ContextImpl中，也就是说Context的大部分方法调用都会转到ContextImpl中，而三者的创建均在ActivityThread中完成

##### 1. Activity对象中ContextImpl的创建
在ActivityThread中的performLaunchActivity方法里面

```
ContextImpl appContext = createBaseContextForActivity(r);
        Activity activity = null;
        ......
         activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback);

```
可以看出，Activity在创建的时候会new一个ContextImpl对象并在attach方法中关联它
##### 2. Application对象中ContextImpl的创建
在ActivityThread中的**handleBindApplication**方法中，此方法内部调用了**makeApplication**方法

```
java.lang.ClassLoader cl = getClassLoader();  
        ContextImpl appContext = new ContextImpl();  
        appContext.init(this, null, mActivityThread);  
        app = mActivityThread.mInstrumentation.newApplication(  
                cl, appClass, appContext);  
        appContext.setOuterContext(app);  
```
看代码发现和Activity中ContextImpl的创建是相同的。

##### 3. 其实Service对象中ContextImpl的创建和Activity、Application也是一致的。

#### Context对资源的访问
不同的Context得到的都是同一份资源，得到资源的方式为context.getResources，而真正的实现位于ContextImpl中的getResources方法，在ContextImpl中有一个成员 private Resources mResources，它就是getResources方法返回的结果，mResources的赋值代码为：

```
mResources = mResourcesManager.getTopLevelResources(mPackageInfo.getResDir(),
                    Display.DEFAULT_DISPLAY, null, compatInfo, activityToken);
```
不同的ContextImpl访问的是同一套资源，但是同一套资源未必是同一个资源，因为资源可能位于不同的目录，但它一定是我们的应用的资源,,所以，尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，因此，它们的mResources都是同一个对象。
#### getApplication和getApplicationContext的区别
getApplication返回结果为Application，且不同的Activity和Service返回的Application均为同一个全局对象，在ActivityThread内部有一个列表专门用于维护所有应用的application
```
final ArrayList<Application> mAllApplications  = new ArrayList<Application>();
```
getApplicationContext返回的也是Application对象，只不过返回类型为Context

```
@Override  
public Context getApplicationContext() {  
    return (mPackageInfo != null) ?  
            mPackageInfo.getApplication() : mMainThread.getApplication();  
}
```
上面代码中mPackageInfo是包含当前应用的包信息、比如包名、应用的安装目录等，原则上来说，作为第三方应用，包信息mPackageInfo不可能为空，在这种情况下，getApplicationContext返回的对象和getApplication是同一个。但是对于系统应用，包信息有可能为空。一个应用只存在一个Application对象，且通过getApplication和getApplicationContext得到的是同一个对象.







