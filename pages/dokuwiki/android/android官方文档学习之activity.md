title: android官方文档学习之activity 

#  android官方文档学习之Activity 
Activity 是应用程序的组件，它提供了一个屏幕，用户可与之互动.每个Activity 会提供一个窗口，在其中绘制它的用户界面。通常窗口会填满整个屏幕，但也有可能比屏幕小并且浮动在其他窗口之上。
应用程序通常是多个松散并相互绑定的Activity组成。一般，用户首次启动应用时，将启动一个被指定为“main”的Activity。每个Activity都可以启动另一个Activity，以执行不同的动作。每次启动一个新的Activity，以前的Activity停止，但在系统堆栈保留Activity（“回栈”）。一个新的Activity启动时，它将Activity压到栈里面并取得用户的焦点。**回堆栈遵守基本的“后进先出”的堆栈机制**，这样，当用户在前Activity，按下返回按钮，则弹出堆栈（并销毁）和恢复以前的Activity。
##  生成Activity 
要生成一个Activity，你必须生成一个Activity 子类Activity （或已有的子类）。在你的子类中，创建，停止，恢复或销毁Acitivty时，你需要实现回调方法，系统调用时，用这些方法表示在Activity生命周期的各种状态之间的转换。两个最重要的回调方法是：
**onCreate()**
必须实现这个方法。生成Activity时系统调用。在你实现中，你应该初始化您的Activity中的组件。最重要的是，这是你必须调用 setContentView() 定义Activity的用户界面的布局。
**onPause()**
当用户离开你的Activity时候（尽管它并不总是意味着被销毁Activity），系统调用此方法。通常如果你需要保持当前会话的话，你所有的改动都应在这提交（因为用户可能不返回）。
##  实现用户界面 
一个Activity的用户界面是由一组按层派生 视图类 -** View** 的视图对象组成。每个视图控制Activity窗口的特定矩形空间，以响应用户交互。
Android提供了一些现成的视图，你可以用它来​​设计和组织布局。“**Widgets**”就是视图，他提供一个可视的可交互的屏幕元素，如按钮，文本字段，复选框，或只是一个图象。“布局”是从 视图组 - **ViewGroup** 派生的，提供了对子视图特殊的布局模式，如线性布局，网格布局，或相对布局的视图组。还可以继承 视图类 - View 和 视图组 - ViewGroup（或现有的子类）来创建自己的Widgets和布局，并将其应用到您的Activity布局中。

你可以通过` setContentView() `设置您的Activity的UI布局 ，传入布局的资源ID。
##  声明Activity 
一个 <activity> 元素也可使用`  <intent-filter> ` 的元素指定的各种** intent过滤器，以说明其他应用程序组件可以如何激活它**。
```

<activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
     <intent-filter>
         <action android:name="android.intent.action.MAIN" />
         <category android:name="android.intent.category.LAUNCHER" />
     </intent-filter>
 </activity>

```
<action> 元素指定应用程序“main”的入口。`  <category> 元素指定系统的activity列表上应用启动的Activity `，（允许用户启动这个Activity）。
如果您希望您的Activity**响应**其他应用程序（和自己）的**隐式intent**，那么你必须在您的Activity中` 定义额外的intent过滤器 `。为了**响应**你需要响应的每种intent，你必须包括 ` <intent-filter> ` ，其中包括一个 ` <action> ` 元素的和一个` 可选的 <category> 元素和/或 <data> 元素 `。` 这些元素指定你的Activity可以响应的intent类型。 `
##  启动Activity 
```

 Intent intent = new Intent(this, SignInActivity.class);
 startActivity(intent);
Intent intent = new Intent(Intent.ACTION_SEND);
 intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
 startActivity(intent);

```
` EXTRA_EMAIL ` 为发送电子邮件的邮件地址字符串数组，其充当intent的额外数据。当一个电子邮件应用程序响应此intent，它读取额外提供的字符串数组，并把它们放在程序的“to”地址栏。在这种情况下，应用程序启动电子邮件Activity，当用户完成后，恢复到你的Activity。
##  启动可以返回结果的 Activity 
使用Activity的 ` startActivityForResult（） `（而不是 startActivity（） 。然后在后续实现了 onActivityResult（） 回调方法的Activity中取得结果，。后续Activity完成后，它会返回一个 intent 给您 ` onActivityResult() ` 方法。
```

 private void pickContact() {
    / /根据CONTENT PROVIDER URI定义，建议一个用于选择联系人得 INTENT
       Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
 }
 
 @Override
 protected void onActivityResult(int requestCode, int resultCode, Intent data) {   
    / /如果请求正常（OK）并且请求是PICK_CONTACT_REQUEST 
     if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        / /查询 content provider 取得联系人的名字 
        Cursor cursor = getContentResolver().query(data.getData(),
        new String[] {Contacts.DISPLAY_NAME}, null, null, null);
        if (cursor.moveToFirst()) { // 游标不为空
            int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
            String name = cursor.getString(columnIndex);
            // 对选中名字的人进行处理...
        }
    }
 }

```
使用 onActivityResult（） 方法处理Activity的结果的基本逻辑。首要条件是检查请求是否成功和这个结果是响应是否已知，如果是，那么ResultCode会是 ` Activity.RESULT_OK ` ，在这种情况下，requestCode匹配 startActivityForResult（） 的第二个参数 。从那起，代码在 intent 中处理取得Activity查询数据的返回结果的（data 参数）。
##  关闭Activity 
通过调用`  finish() ` 方法关闭Activity的。
##  管理Activity生命周期 
![](/data/dokuwiki/android/pasted/20150606-183224.png)
通过实现这些方法，您可以监视Activity生命周期的三个嵌套循环：
Activity的整个存在周期发生在`  OnCreate（） 调用和 OnDestroy `（ 调用之间。Activity调用 OnCreate（） 执行“全局”状态设置（如定义布局） ，并调用 OnDestroy（ 释放所有剩余资源 。例如，如果Activity有一个线程在后台运行，从网络上下载数据，它可能会调用 OnCreate（） 创建该线程 ，然后由 OnDestroy（ 停止线程的 。

` Activity的可见周期发生在 OnStart（） 调用和 onStop（） 调用之间 `。在这段时间内，用户可以看到屏幕上的Activity，并与它进行交互。例如，一个新的Activity时启动时，onStop（）被调用，这时Activity不再可见。这两种方法之间，你可以维持运行Activity所需要的资源，提供给用户。例如，你可以在调用OnStart（） 注册 BroadcastReceiver 监测影响你用户界面的变化，当用户可以不再看到你内容时，在 onStop（） 时注销。在整个存在周期的Activity，Activity在可见和不可见的变化中，系统可能会多次调用OnStart（） 和 OnStop （） 。

Activity的` 前台周期发生在 onResume() 调用和 onPause() 调用 `之间。在这段时间内，该Activity显示在屏幕上，在所有其他Activity之前，**具有用户输入焦点**。一个Activity经常在前台后台之间转换，例如，当设备进入睡眠状态，或弹出一个对话框是 onPause()被称调用。因为这种状态转换，在这两种方法中的代码应该是相当轻量级的，以避免缓慢的转换，使用户等待。

Activity创建后，`  onPause()是在系统能kill进程之前保证能背调用到得方法——如果系统在紧急情况下必须恢复内存，之后 onStop() 和 onDestroy()可能不会被调用， `因此，你应该使用 onPause()写入些持久性数据（如存储用户正在编辑的数据）。然而，对于要保存的数据，你应该有选择性 ，因为任何阻塞方法将阻塞系统转到下一个Activity，降低了用户体验。

##  保存Activity状态 
系统为了回收资源或者设备配置变化（如设备旋转）而销毁 Activity，这时系统不能简单地恢复到之前的完整状态。相反，如果用户返回到Activity,系统必须重新创建 Activity 对象 。要保证有关Activity状态的重要信息是通过实现一个叫做 ` onSaveInstanceState（Bundle） ` 辅助回调函数来完成的,**它允许你保存Activity的状态信息**。
系统重建Activity时，系统会同时传给`  onCreate() 和 onRestoreInstanceState() Bundle ` 。使用这两种方法，你可以从 Bundle 提取到保存的状态信息来恢复Activity。如果没有恢复的状态信息， Bundle传递的是空（首次创建Activity的情况）。` 所以需要检查Bundle是否为null `
![](/data/dokuwiki/android/pasted/20150606-183243.png)
##  处理配置更改 
某些设备配置在运行时可以改变（如屏幕方向，键盘的可用性，和语言）。当这种变化发生时，Android重新运行Activity（系统调用的 onDestroy() ，然后立即调用的 onCreate（） ） 。这种行为旨在帮助您用您所提供的（如不同的屏幕方向和大小不同的布局）的替代资源进行应用程序自动重载，以适应新的配置。
处理重新启动来保存和恢复Activity状态的一个的最佳方式是使用在上一节讨论的` onSaveInstanceState（） 和onRestoreInstanceState（） （或onCreate（） ）。 `

##  Tasks and Back Stack 
###  Activity和Task的默认行为: 
  * 当 Activity A 启动 Activity B 时,Activity A 被停止,但系统仍会保存Activity A 状态（比如滚动条位置和 form 中填入的文字）如果用户在 Activity B 中按下返回键时,Activity A 恢复运行，状态也将恢复.
  * 当用户按下Home键离开Task 时，当前 Activity 停止Task 转入后台,系统会保存Task中每个Activity 的状态。如果用户以后通过选中启动该 Task 的图标来恢复Task,Task 就会回到前台，栈顶的Activity 会恢复运行.
  * 如果用户按下返回键,当前Activity 从栈中弹出,并被销毁.栈中前一个Activity恢复运行.当Activity 被销毁时，系统不会保留Activity 的状态.Activity甚至可以在不同的Task中被实例化多次.
  * 
如上所述**,把所有已经启动的Activity 相继放入同一个Task 中以及一个“后入先出”栈，Android 管理Task和Back Stack 的这种方式适用于大多数应用,**你也不用去管理你的Activity 如何与Task关联及如何弹出Back Stack .不过,有时候你或许决定要改变这种普通的运行方式.也许你想让某个Activity启动一个新的Task（而不是被放入当前Task中）,或者，你想让 activity 启动时只是调出已有的某个实例（而不是在Back Stack 顶创建一个新的实例）或者，你想在用户离开Task 时只保留根Activity，而Back Stack 中的其它 Activity 都要清空，

你能做的事情还有很多，利用 " <activity> manifest 元素的属性和传入 startActivity() 的 intent 中的标识即可。
**这里，你可以使用的 <activity> 属性主要有：**
  * taskAffinity
  * ` launchMode `
  * allowTaskReparenting
  * clearTaskOnLaunch
  * alwaysRetainTaskState
  * finishOnTaskLaunch

**可用的 intent 标识主要有：**
  * FLAG_ACTIVITY_NEW_TASK
  * FLAG_ACTIVITY_CLEAR_TOP
  * FLAG_ACTIVITY_SINGLE_TOP

###  定义启动模式 
  * 使用 manifest 文件
  * 使用 Intent 标志

在manifest文件中声明Activity 时,你可以使用 <activity> 元素的`  launchMode ` 属性来指定Activity与Task的关系.
launchMode 属性指明了Activity启动Task的方式。 **launchMode 属性可设置四种启动模式**：
###  Activity的四种启动模式 
具体参考官方文档的Tasks and Back Stack部分介绍。
配置Activity时我们可以通过` android:launchMode `属性制定Activity的加载模式。该属性支持4种模式：
  * ` standard `-默认模式。**每次激活**Activity时都会**创建新的实例**并添加到当前的Activity栈。` 不会创建新的Task `.如果Activity的一个实例已经存在于当前Task的栈顶，该系统就会使用` onNewIntent() `方法通过intent 传递给已有实例，而不是创建一个新的Activity 实例.**注意： 一个Activity 的新实例创建完毕后，用户可以按返回键返回前一个activity.但是当Activity 已有实例正在处理刚到达的intent 时，用户无法用返回键回到onNewIntent()中 intent 到来之前的Activity 状态.**` 各个实例可以属于不同的Task，一个Task 中也可以存在多个实例. `
  * ` singleTop `:如果**栈顶**正好存在该Activity实例就会**重用**它，否则就会创建新实例。` 不会创建新的Task `
  * ` singleTask `:如果当前栈中存在Activity实例不管是否在栈顶都会**重用**它，并将其**移至栈顶**。否则创建新实例。` 采用该模式的Activity在同一个Task内只能有一个实例。 `如果Activity 已经在其它Task 中存在实例，则系统会通过调用其实例的` onNewIntent() ` 方法把 intent传给已有实例.
  * ` singleInstance `:在一个**新栈**中创建该Activity实例，并**让多个应用共享该栈中的Activity实例**。一旦该Activity存活于栈中，后续对该Activity的请求都会**重用该实例**。其效果相当于` 多个应用程序共享该Activity实例，而不管是谁激活都会重用。 `采用该模式加载的Task只包含该Activity.

**使用 Intent 标识**
在启动Activity 时，你可以在传给 startActivity() 的 intent 中包含相应标识,用于修改Activity 与Task 的默认关系。这个标识可以修改的默认模式包括：
` FLAG_ACTIVITY_NEW_TASK `
在新的Task 中启动Activity.如果要启动的Activity 已经运行于某个Task 中,**则那个Task 将调入前台中**,最后保存的状态也将会恢复,Activity 将在onNewIntent()中接收到这个新 intent.与前一章节所描述述的` "singleTask"launchMode `模式相同.
` FLAG_ACTIVITY_SINGLE_TOP `
与前一章节所述的 "` singleTop `"launchMode模式相同.
` FLAG_ACTIVITY_CLEAR_TOP `
**如果要启动的Activity 已经在当前Task中运行,则不再启动一个新的实例，且所有在其上面的Activity 将被销毁，然后通过onNewIntent()传入 intent 并恢复Activity（不在栈顶）的运行.**
此种模式在launchMode中没有对应的属性值.
` FLAG_ACTIVITY_CLEAR_TOP 经常与 FLAG_ACTIVITY_NEW_TASK 结合起来一起使用.这些标识定位在其它Task中已存在的Activity,再把它放入可以响应 intent的位置上. `
注意：如果Activity 的启动模式配置为"standard"，它就会先被移除出栈，再创建一个新的实例来处理这个 intent,因为启动模式为 "standard" 时，总是会创建一个新的实例。

**对于那些你不希望用户能够返回的Activity,将<activity>元素的finishOnTaskLaunch 设置为"true" 即可.**