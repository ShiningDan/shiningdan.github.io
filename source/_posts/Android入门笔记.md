---
title: Android入门笔记
date: 2018-04-03 15:32:37
categories: coding
tags:
  - Android
---

本文是学习 Android 的笔记

<!--more-->

## Android 简介

四大组件：Android四大基本组件分别是Activity，Service服务,Content Provider内容提供者，BroadcastReceiver广播接收器。

四大组价的使用都需要在 Manifest 中进行注册。

## Andriod 项目

Application Name：应用发布以后，在手机上的名称

### Andriod 项目目录

* drawable：存放不同分辨率的图片
* layout：存放布局文件
* values：存放字符串strings.xml、主题，颜色、样式styles.xml等资源文件
* menu：存放菜单布局文件
* manifest：清单文件，配置与应用相关的重要信息，如包名，权限，程序组件等

## Android 控件

### 控件通用属性介绍

#### id 

Android中的组件需要用一个int类型的值来表示，这个值也就是组件标签中的id属性值。

id属性只能接受资源类型的值，也就是必须以`@`开头的值，例如，`@id/ab`、`@+id/xyz`等。

如果在`@`后面使用 `+`，表示当修改完某个布局文件并保存后，系统会自动在`R.java`文件中生成相应的int类型变量。变量名就是` /` 后面的值，例如，`@+id/xyz` 会在 `R.java` 文件中生成 `int xyz = value`，其中 `value` 是一个十六进制的数。如果xyz在R.java中已经存在同名的变量，就不再生成新的变量，而该组件会使用这个已存在的变量的值。

#### layout_width

* wrap_content：包裹实际文本内容
* match_parent：当前控件宽度铺满父类容器，2.3 API 之后添加的属性值
* fill_parent：当前控件铺满父类容器，2.3 API 之前添加的属性值

### TextView

显示文本框控件，用来显示文字

#### TextView 属性

* id：控件的id
* width：空间的宽度，使用 dp 作为绝对宽度的单位
* height：空间的高度
* text：文本内容
* textSize：本文大小，单位是 sp，sp 保证文字的大小随着系统文字大小的变化而变化
* textColor：文本颜色
* background：控件背景

### EditText

输入文本框，可以进行文本编辑

#### EditText 属性

* id：控件的id
* width：空间的宽度
* height：空间的高度
* text：文本内容
* textSize：本文大小
* textColor：文本颜色
* background：控件背景

特殊属性

* hint：输入提示文本
* inputType：输入文本类型

### ImageView

* src：ImageView 的内容图片（`android:src=@drawable/ic_launcher`）
* background：ImageView 的背景图片（`android:background=@drawable/ic_launcher`）

安卓系统识别用户手机分辨率的大小，然后自动选择合适分辨率的图片

### Button

Button 有 text 属性，用来表示按钮的文本，但是 ImageButton 没有 text 属性

### ImageButton

ImageButton 相比于 Button 控价多了一个 `android:src` 来设置按钮的背景图片

#### 监听点击事件

除了 Button 以外，很多类都可以监听点击事件

实现监听事件的方法有：

**匿名内部类的实现**

```
button.setOnClickListener(new OnClickListener () {
    @Override
    public void onClick(View v) {
        System.out.println("我的 Button 被点击了");
    }
})
```

**独立类的实现**

```
btn2.setOnclickListener(new Button2OnClickListener());

class Button2OnClickListener implements View.OnClickListener {

    @Override
    public void onClick(View v) {
        System.out.println("Button2 被点击了");
    }
}
```

**通过接口的方式来实现**

略

### AutoCompleteTextView

动态匹配输入的内容，如搜索框

#### 属性

* completionThreshold：设置输入多少字符时自动匹配

### MultiAutoCompleteTextView

可以支持选择多个值，分别用分隔符隔开，并且在每个值选中的时候再次输入值时会去自动匹配。可用在发短信，发邮件的时候选择联系人的这种类型。

#### 属性

* completionThreshold：设置输入多少字符时自动匹配
* mtxt:setTokenizer(new MultiAutoCompleteTextView.CommaTokenizer())：设置分隔符

### ToggleButton

多状态按钮有两个状态，一个是选中状态，一个是未选中状态，并且可以为不同的状态显示不同的文本内容

#### 属性

* checked：是否被选中，如果被选中为 true，未被选中为 false
* textOff：未选中的文本
* textOn：选中的文本

### checkBox

复选框，有两种状态，选中状态（true）和未选中状态（false）

#### 属性

* checked：是否被选中
* text：复选框的文字

### RadioGroup 和 RadioButton

单选框。单独的 RadioButton 是按下去就不可以取消的，所以不建议单独使用 RadioButton

#### 属性

* orientation：设置当前的 RadioButton 是竖直排布还是水平排布

## Android 布局

一般 linearLayout 和 RelativeLayout 搞定所有的布局

布局里面还可以套用布局

### 线性布局（LinearLayout）

子空间以横向或者纵向排布

#### 属性

* orientation：排布方式（vertical，纵向；horizontal，横向）
* gravity：子类的位置（center | center_vertival，垂直居中 | center_horizontal，水平居中 | right | left | bottom）。在这里的 gravity 属性值可以多个连用，如：`right|bottom`

#### LinearLayout 布局中控件经常用到的属性

* layout_gravity：指的本控件在当前父容器中的 xy 位置，如 `bottom`
* layout_weight：指的本控件占当前父容器的比例，值是 Int 或者 Float

### 相对布局（RelativeLayout）

相对布局的子控件将以控件之间的相对位置或者子控件相对父控件容器的位置的方式排列

相对布局在拖拽控件的时候，可以布局在父控件容器的任意位置

#### RelativeLayout 布局中子控件常用到的属性

* alignParentLeft：子控件相对当前父控件容器的左边（true | false）
* alignParentTop：子控件相对当前父控件容器的上面（true | false）
* marginLeft：子控件相对父控件容器左边的距离
* marginTop：子控件相对父控件容器上面的距离
* centerInParent：子控件相对父类容器既水平又垂直居中（true | false）
* centerHorizontal：水平居中（true | false）
* centerVertical：垂直居中（true | false）

**除此以外，还可以通过属性设置来设置子类控件相对子类控件的位置**

* layout_below="@+id/button1"：该控件位于给定 id 控件的底部
* layout_toRightOf：该控件位于给定 id 控件的右边
* layout_above：该控件位于给定 id 控件的上部
* layout_toLeftOf
* layout_alignBaseline：该控件位与给定 id 控件的内容在同一条线上
* layout_alighBottom：该控件位与给定 id 控件的底部边缘对齐
* layout_alighLeft：该控件位与给定 id 控件的左边缘对齐
* layout_alighRight：该控件位与给定 id 控件的右边缘对齐
* layout_alighTop：该控件位与给定 id 控件的顶部边缘对齐

### FragmentTableHost

FragmentTableHost 是一种控件，但是可以组成我们非常常用的布局，类似于 IM 应用的下边栏等，可以了解一下。

### FrameLayout 帧布局

在这个布局中，所有的子控件都不能被放到指定的位置上，它们统统放在这块区域的左上角，并且后面的子控件直接放在前面的子控件之上，将前面的子控件全部或部分遮挡

### AbsoluteLayout 绝对布局

又叫做坐标布局，可以对子控件指定绝对位置（xy），用的比较少

由于手机尺寸差别比较大，绝对布局在屏幕适配上有缺陷

### TableLayout 表格布局

模型以行列的形式管理控件，每一行为一个 TableRow 的对象，当然也可以为其他的控件对象



### Activity 和布局

Activity 是用户可以执行的单一任务。Activity 负责创建新的窗口供应用绘制和从系统中接收事件。Activity 是用 Java 编写的，扩展自 Activity 类。

### 布局 XML

#### 视图类型：UI 组件

视图分为两大类别。第一种类型是 UI 组件，通常具有互动性。

```
类名称         	说明
TextView	在屏幕上创建文本；通常是非互动性文本。
EditText	在屏幕上可以输入文本
ImageView	在屏幕上创建图片
Button	    在屏幕上创建按钮
Chromometer	在屏幕上创建简单的计时器
```

[android.widget](http://developer.android.youdaxue.com/reference/android/widget/package-summary.html) 程序包包含了你可以使用的大部分 UI 视图类的列表。

#### 视图类型：容器视图

第二种视图叫做“布局”或“容器”视图。它们扩展自 ViewGroup 类。它们主要负责包含一组视图并判断放在屏幕的何处。

```
类名称	             说明
LinearLayout	在一行或一列里显示视图。
RelativeLayout	相对某个视图放置其他视图。
FrameLayout	ViewGroup 包含一个子视图。
ScrollView	一种 FrameLayout，旨在让用户能够在视图中滚动查看内容。
ConstraintLayout	这是更新的 viewgroup；可以灵活地放置视图。在这节课的稍后阶段，我们将学习 ConstraintLayout。
```

### XML 属性

#### Text

`android:text="Your name appears here"`

使 TextView 显示单词

#### 宽度和高度

```
android:layout_width="200dp"
android:layout_height="300dp"
```

为了使布局能够自适应，并针对不同的设备进行调整，两个常见的值根本不是数字

```
android:layout_width="wrap_content"
android:layout_height="match_parent"
```

### Padding 和 Margin

#### XML 布局与 Java Activity 有何关系

创建 XML 布局后，你需要将其与你的 Activity 相关联。你可以在 Activity 的 `onCreate` 方法中使用 `setContentView` 方法进行关联。

```
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       // other code to setup the activity
    }
    // other code
}
```


#### R 类

当你的应用被编译时，系统会生成 R 类。它会创建常量，使你能够动态地确定 `res` 文件夹的各种内容，包括布局。

其中包含您的 `res/` 目录中所有资源的资源 ID。

访问资源的方法有两种：

在代码中：使用来自 R 类的某个子类的静态整型数，例如：
`R.string.hello`
`string` 是资源类型，`hello` 是资源名称。当您提供此格式的资源 ID 时，有许多 Android API 可以访问您的资源。

在 XML 中：使用同样与您 R 类中定义的资源 ID 对应的特殊 XML 语法，例如：
`@string/hello`
`string` 是资源类型，`hello` 是资源名称。

#### setContentView

那么 `setContentView` 方法是干什么的？它会扩展布局。本质上是 Android 会读取你的 XML 文件并为你的布局文件中的每个标记生成 Java 对象。然后，你可以在 Java 代码中通过对 Java 对象调用方法修改这些对象。

### 视觉布局编辑器

可以支持使用拖拽的方式来实现对布局的修改。

## 资源

res 目录是放置图片、字符串和布局等的地方。

### 一些常见资源类型

```
名称	          存储的内容
值	     包含简单值（例如字符串或整数）的 XML 文件
drawable	一些外观文件，包括位图文件类型和形状。
布局	          应用的 XML 布局
```

### 其他资源类型

```
animator	属性动画的 XML 文件
anim	补间动画的 XML 文件
color	定义状态列表颜色的 XML 文件
mipmap	启动器图标的 Drawable 文件
menu	定义应用菜单的 XML 文件
raw	保存为原始格式的任意文件的资源文件。例如，你可以将音频文件放在此处。
xml	任意 XML；如果你具有 XML 配置文件，那么建议放在这里
```

### 在 XML 和 Java 中使用资源

你已经实际地看到如何使用资源。例如，你已经在 MainActivity 中看到资源的使用过程。`setContentView(R.layout.activity_main)` 是指引用资源文件 (`activity_main.xml`) 并用作 `MainActivity` 的布局。

#### 使用 strings.xml

在 Java 中，你可以通过调用 `getString` 方法获取保存在 `res -> values -> strings.xml` 中的 String。


## 日志的使用

* ERROR：记录明显的的错误
* WARN：记录不会使得应用崩溃，但是需要注意的事情（比如媒体应用的磁盘空间不足）
* INFO：记录纯信息（比如连接到了互联网）
* DEBUG：不会在正式版出现的消息，通常用来输出通过一个函数构成的 URL 或者来自 WEB 服务的响应
* VERBOSE：不会在正式版出现的消息，用于提供非常精细的信息的时候使用，大多数时候不会使用
* WTF：Android 特有的日志等级（What a Terrible Failure），比 ERROR 的等级还要高，用于报告永远不可能发生的错误。在 Debug 模式下，WTF 可能会强制设备停止并且输出调试报告，最好不要使用

## Activity 声明周期

![](http://hi.csdn.net/attachment/201109/1/0_1314838777He6C.gif)

在 [Activity 生命周期详细解释](https://blog.csdn.net/qq_34927117/article/details/52404115) 这篇文章中可以看到 Activity 不同生命周期对应的回调函数的介绍

## 页面之间的跳转与传值

Activity 的启动分为显示启动和隐式启动。

### 显示启动

显示启动就是通过 Intent 来指定被启动的 Activity，提供要启动的 activity 的 class，这样就可以启动同一个应用的 activity。

创建 Activity 的时候，要通过 new Activity 来创建，这样创建的 activity 才能被自动添加到 AndroidManifest.xml 中

### 隐式启动

隐式启动时通过 intent-filter 进行匹配，主要应用于存在有相互调起的逻辑的应用之间，比如启动浏览器，或者启动电话、短信界面等。

首先在应用A的某个类中：

```
Intent intent = new Intent();
intent.setAction("com.android.activity");//"com.android.activity"为自定义action
//Intent intent = new Intent("com.android.activity");或者使用这种写法
intent.putExtra("KEY","VALUE");//传递数据
startActivity(intent);
```

在应用B中接收A发来的action：

```
<activity android:name=".DemoActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"
        <action android:name="com.android.activity" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

### 传值

在传值的 Activity 可以使用 `intent.putExtra()` 来输入要传入的值，在接收方使用 `getIntent().getExtra()` 来获得值。

使用 intent 也可以传一个 Bundle 类型的对象。Bundle 就是把基础数据类型打包，然后一起发给接收方。

```
Bundle bundle = new Bundle();
bundle.putInt(1);
bundle.putString("123");
intent.putExtra(bundle);
```

## 资源

资源都在 res 文件夹下

* drawable：主要是指内存中的一张图片，因为是用 xml 来描述一张图片
* layout：布局
* mipmap：主要放置应用启动图标，因为对应不同分辨率的设备需要不同大小的图标，其他的图片还是推荐放在 drawable 中
* color：放置全局的颜色配置
* dimens：全局尺寸配置，比如通用文字大小等
* strings：全局字符串，一般在做国际化的时候，主要对 string.xml 文件中的字符串进行翻译
* styles：全局的样式

所有配置的资源，在编译的时候，都会被记录到 R.class 中，之后去引用文件什么的，直接通过 R 文件去引用。

### drawable

在 drawable 中使用 XML 来描述，比如说，我们需要一个圆角的按钮，则我们可以在 drawable 中定义一个圆角的按钮，然后在 `<Button>` 标签中通过 `background` 来进行引用

```
<shape>
    <solid color="@color/red" />
    <corners radius="3dp" />
</shape>
```

### 自定义资源文件夹

除了以上的文件夹以外，我们还可以自定义资源文件件，如 assets，row 等。两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制。

* res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即
* res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹
* assets 下面的文件需要通过流来进行读取

## 线程

handler 主要用于子线程和 UI 线程之间的通信

一个安卓进程中有一个主线程，也叫 UI 线程，只有 UI 线程才能操作 View。如果在子线程中操作了 View，则会直接崩溃。UI 线程不能做耗时操作，超过 10s 也会崩溃。所以耗时的操作必须 new 一个子线程


Handler 的用法，就是在一个新的线程中，把需要传输的数据通过 Message 对象进行包裹，然后调用 handler.sendMessage 来进行数据的传递。然后在 Handler.handleMessage 中进行数据的处理。

其中，handler 可能处理多种数据类型，所以在构造 Message 对象的时候，还需要给 message.what 进行赋值，表示这个消息的类型。

### AsyncTask

AsyncTask 是对 Thread 和 Handler 的一个封装，这样在开发的时候，就不用每一次都 new 一个 Thread 来进行后台操作，而是直接使用 AsyncTask。

## 网络

访问网络，需要在 AndroidManifest.xml 中加一个权限 

```
<uses-permission android:name="android.permission.INTERNET" />
```

## 广播

广播主要用在应用内不同组件之间的通信，和不同应用之间的通信。

广播的生命周期比较简单，也不能做耗时的操作，一般是超过 5 秒，广播就会被自动销毁 

### 广播的注册方式

#### 注册在 Manifest.xml 中

注册在 Manifest.xml 中的广播，在应用安装的时候，配置会被解析，然后注册在全局系统中。

常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行。

```
<!--广播注册-->  
<receiver android:name=".SmsBroadCastReceiver">  
    <intent-filter android:priority="20">  
        <action android:name="android.provider.Telephony.SMS_RECEIVED"/>  
    </intent-filter>  
</receiver>  
```

通过 action 来接受广播信息

#### 在代码中进行注册

在代码中进行注册的广播，其生命周期就是局部 Activity 的生命周期。广播通常在 `onCreate` 中进行注册，然后在 `onDestroy` 的时候进行反注册，否则可能会出现内存泄露的情况。

```
...

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_linear);

    myBroadcastReceiver = new MyBroadcastReceiver();
    intentFilter = new IntentFilter();
    intentFilter.addAction("android.provider.Telephony.SMS_RECEIVED");
    registerReceiver(myBroadcastReceiver, intentFilter);
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (myBroadcastReceiver != null) {
        unregisterReceiver(myBroadcastReceiver);
    }
}

...
public class MyBroadCastReceiver extends BroadcastReceiver   
{  
   @Override  
   public void onReceive(Context context, Intent intent)   
   {   
       //在这里可以写相应的逻辑来实现一些功能
       //可以从Intent中获取数据、还可以调用BroadcastReceiver的getResultData()获取数据
   }   
}
```

### 广播的第三方库

现在项目中手写广播代码的机会比较少，多数还是使用第三方库：

* eventbus
* rxbus

### 广播类型

1. 普通广播：Intent 发出来的广播
2. 系统广播
3. 有序广播：有序广播的接受者有一个优先级，接受者按照优先级接收到广播信息
4. 粘性广播：不推荐使用
5. APP 内广播：应用内接收到的广播，安全性更高，更加高效

## 网络请求

在 Android 中做网络请求通常使用第三发的库，如 okhttp/retrofit2，

## 数据持久化

主要有两种方式，第一个是 sharedPreference，类似于一个 XML 文件。第二个是 SQLite。

### sharedPreference

```
// 获得 sharedPreference，第二个参数是数据文件的权限，如 private，全局可读等
SharedPreferences sp = getSharedPreferences("zyuhcen", MODE_PRIVATE);
SharedPreferences.Editor editor = sp.edit();
editor.putString("first", "hello");
editor.putInt("second", 123);
editor.putBoolean("third", true);
editor.apply();
```

### SQLite

使用 SQLite，需要继承 SQLiteOpenHelper 这个类来进行实现

在 `onCreate` 函数中创建数据表，数据库在创建完成后就再也不会调用该函数

`onUpgrade` 函数是数据库版本升级的时候调用。

```
public class MySQLiteHelper extends SQLiteOpenHelper{

    public MySQLiteHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version) {
        super(context, name, factory, version);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("create table user(id integer primary key autoincrement,"
            + "name text,"
            + "age integer"
            + ")");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}

...
MySQLiteHelper mySQLiteHelper = new MySQLiteHelper(this, "mydb", null, 1);
// 调用 getReadableDatabase 或者 getWritableDatabase 的时候才会调用 onCreate 方法
SQLiteDatabase db = mySQLiteHelper.getReadableDatabase();
// 最后通过 db 对象进行 CRUD
...
```

不过现在很少直接用 SQL 语句操作 SQLite，而是用 ORM 的第三方库


## Service

直接使用频率比较少

service 没有用户界面，可以长时间在后台运行

service 既不是一个进程，也不是一个线程，只是一个组件

service 默认也是运行在UI线程中（四大组件默认都在UI线程中），所以 service 中要做耗时操作也要开启子线程来做

service 优先级相对于 activity 等来说高一些，所以当系统内容紧张时 service 被回收的概率比较低，这是 service 能够长期在后台运行的基础。 当然内存非常紧张时 service 实例还是有可能被回收的，我们的整个进程也可能会被系统回收，这时我们要通过一些手段在 service 或 进程被回收后重启进程和 service 来保证 service 的长期运行。 （可查阅android进程保活的方法）
 
现在 android 消息推送都是基于 TCP 长连接来做的，客户端会启动一个 push service 一直在后台运行。这样就保证了我们的手机永远在线，后台发送消息时就能随时通过 socket 写过来。 push service 通常会放到一个单独的进程中执行。

### 

继承 Service 类可以创建一个 Service，然后实现 Service 类中的主要声明周期函数。

```
public class MyService extends Service {
    public MyService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d("zyuchen", "service start command");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onCreate() {
        Log.d("zyuchen", "create service");
        super.onCreate();
    }

    @Override
    public void onDestroy() {
        Log.d("zyuchen", "destroy service");
        super.onDestroy();
    }
}

...

btn1.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Intent intent = new Intent(getApplicationContext(), MyService.class);
        startService(intent);
    }
});

...

```

其中，`onCreate` 和 `onDestroy` 只会在 Serivce 创建和删除的时候调用，`onStartCommand` 可以被多次触发。

主线程或其他线程，通过 `onBind` 方法和 Service 进行交互。


## ContentProvider

使用频率比较少

## 代码混淆

代码混淆：apk 可以通过反编译获得源码。混淆可以把类名，路径，变量都变成可读性差的变量

在 Android 中，可以使用 SDK 下面的 /tools/proguard 来实现代码混淆

在 build.gradle 中，将 buildType/release/minifyEnabled 设置为 true，则在构建的时候 release 版本的时候会对代码进行混淆。

如果想要反编译，可以使用 apktool 这个工具

## 签名

只有签名的 APK 才能安装与发布

可以使用 keytool 工具来进行签名，也可以使用 Android Studio 中 build/generate signed apk 来进行签名

签名时生成的签名文件就相当于一个版权的概念。

看一个安装包有没有进行签名，可以使用 apktool 工具进行反编译，然后签名文件所在的目录为：original/META_INF 下面。

## 大象 Demo 开发记录

### 组件样式修改

默认的输入框，在被选中的时候，会显示红色的下划线。所以我们需要对组件样式进行修改。在 `res/values/styles.xml` 中，添加我们的 style

```
<style name="EditTextTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorControlNormal">@color/colorPrimary</item>
    <item name="colorControlActivated">@color/colorPrimary</item>
    <item name="colorControlHighlight">@color/colorPrimary</item>
</style>
```

然后在输入框中配置我们的 style：

```
<EditText
        android:id="@+id/password"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:inputType="textPassword"
        android:textSize="@dimen/fontSizeMiddle"
        android:theme="@style/EditTextTheme"
        android:hint="密码" />
```

### 主页面的实现

主页面的实现，使用的是 ViewPager + Fragment。

使用 ViewPager + Fragment 的方式综合了使用ViewPager和使用Fragment的优势，即：既能使用Fragment管理Tab对应页面的布局及业务逻辑的实现，使得Activity与Tab对应的页面分离，又能使用ViewPager实现左右滑动切换页面的效果。这种方式需要为ViewPager设置FragmentPagerAdapter适配器



## 参考

* [Android 简介 | Developer.Android.com](https://developer.android.com/guide/index.html)
* [三种实现Android主界面Tab的方式](https://www.cnblogs.com/caobotao/p/5103673.html)
* [Android必学之数据适配器BaseAdapter](http://www.cnblogs.com/caobotao/p/5061627.html)
