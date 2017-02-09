title: android官方文档学习之输入控件 

#  android官方文档学习之输入控件 
部分输入控件列表：
![](/data/dokuwiki/android/pasted/20150609-075047.png)
##  Buttons 
```

需要有文字的按钮，使用<Button>类：
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_text"
    ... />
需要有图标的按钮，使用<ImageButton>类：
<ImageButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@drawable/button_icon"
    ... />
需要有文字和图标的按钮，使用具有android:drawableLeft属性的<Button>类：
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_text"
    android:drawableLeft="@drawable/button_icon"
    ... />

```
###  按钮的样式 
**无边框按钮** - Borderless button
一种有用的设计是无边框按钮。无边框按钮与基本按钮相似，但是无边框按钮没有无边框或背景，但在不同状态如点击时，会改变外观。
要创建一个无边框“按钮，为按钮应用<borderlessButtonStyle>样式。例如：
```

<Button
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage"
    style="?android:attr/borderlessButtonStyle" />

```
**定制背景**
在**res/drawable/** directory下创建一个新的XML文件（命名类似于button_custom.xml）。使用下面的XML：
```

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/button_pressed"
          android:state_pressed="true" />
    <item android:drawable="@drawable/button_focused"
          android:state_focused="true" />
    <item android:drawable="@drawable/button_default" />
</selector>

```
```

<Button
    android:id="@+id/button_send"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button_send"
    android:onClick="sendMessage"
    android:background="@drawable/button_custom"  />

```
##  文本框 
文本框允许用户在应用程序中输入文本。它们可以是单行的，也可以是多行的。点击文本框后显示光标，并自动显示键盘。除了输入，文本框还包含其它操作，比如文本选择（剪切，复制，粘贴）以及数据的自动查找功能。你可以使用EditText对象在布局中添加一个文本字段， android里的写法通常是在XML布局文件中添加<EditText>元素
文本字段可以有**不同的输入类型，如数字，日期，密码，或电子邮件地址**。类型确定文本框内允许输入什么样的字符，可能会提示虚拟键盘调整其布局来显示最常用的字符。
你可以在EditText对象使用Android:inputType属性指定输入类型的键盘，
```

<EditText
    android:id="@+id/email_address"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/email_hint"
    android:inputType="textEmailAddress" />

```
android:inputType
针对不同的情况有几种不同的输入类型。你可以找到所有的文件中列出的android：inputType属性
  * "text"普通文本键盘
  * "textEmailAddress"带有@字符的普通文本键盘
  * "textUri"带有/字符的普通文本键盘
  * "number"基本数字键盘
  * "phone"手机式键盘

android：inputType还允许您指定操作行为，如在某此键盘上是否要利用所有新词，或使用自动完成和拼写建议功能。
inputType的属性允许按位组合，让您可以一次指定一个键盘布局和一个或多个操作行为。
下面是一些定义键盘行为的普通输入值：
  * "textCapSentences"每个句子首字母大写
  * "textCapWords"每个词语均大写，适用于标题或人名
  * "textAutoCorrect"纠正常见单词的错误拼写
  * "textPassword"输入的字符转出点
  * "textMultiLine"允许用户输入包括换行符（回车）在内的长字符串

例如，你如何收集邮政地址，利用每一个字，并禁用文字的行为：
 
```

<EditText
    android:id="@+id/postal_address"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/postal_address_hint"
    android:inputType="textPostalAddress|
                 textCapWords|
                 textNoSuggestions" />

```
###  指定键盘操作 

除了改变键盘的输入类型，当用户完成输入时，android允许你指定特殊的按钮进行相应的操作，**如把回车键作为 “搜索”或 “发送”操作。**
如果你声明的Android imeOptions =“actionSend” ，键盘包括发送的行动。
您可以通过android：imeOptions属性设置指定的动作。例如，这里你可以指定发送的行为：
```

<EditText
    android:id="@+id/search"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/search_hint"
    android:inputType="text"
    android:imeOptions="actionSend" />

```
**响应按钮事件**
如果您已指定键盘采用` Android：imeOptions `属性（“actionSend”等）的操作方法，你可以使用` TextView.OnEditorActionListener `监听事件行为。TextView.OnEditorActionListener接口提供了一个回调方法` onEditorAction（） `，它通过输入的动作ID，如` IME_ACTION_SEND或IME_ACTION_SEARCH `行为调用相关的动作类型方法。
例如，你可以监听用户点击键盘上的发送按钮：
 
```

EditText editText = (EditText) findViewById(R.id.search);
editText.setOnEditorActionListener(new OnEditorActionListener() {
    @Override
    public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
        boolean handled = false;
        if (actionId == EditorInfo.IME_ACTION_SEND) {
            // Send the user message
            handled = true;
        }
        return handled;
    }
});

```
**设置文本框标签**
你可以通过设置` imeActionLabel `属性来**定制按钮的文本**。
 
```

<EditText
    android:id="@+id/launch_codes"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:hint="@string/enter_launch_codes"
    android:inputType="number"
    android:imeActionLabel="@string/launch" />

```
**添加其他键盘标志**
Android：imeOptions
#  提供自动完成建议 
` AutoCompleteTextView `控件。为了实现自动完成，你必须指定一组（@link adndroid.widget.` Adapter `）文字提供建议。有几种可用的适配器，匹配数据项，如从数据库或一个数组获取所需要匹配值。
```

<?xml version="1.0" encoding="utf-8"?>
<AutoCompleteTextView xmlns:android="http://schemas.android.com/apk/res/android" 
    android:id="@+id/autocomplete_country"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content" />

定义一个数组，里面包含数组所需的子值。例如，这里是一个国家的名字，这是定义在一个XML资源文件（res/values/strings.xml）数组：
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="countries_array">
        <item>Afghanistan</item>
        <item>Albania</item>
        <item>Algeria</item>
        <item>American Samoa</item>
        <item>Andorra</item>
        <item>Angola</item>
        <item>Anguilla</item>
        <item>Antarctica</item>
        ...
    </string-array>
</resources>

```
```

// Get a reference to the AutoCompleteTextView in the layout
AutoCompleteTextView textView = (AutoCompleteTextView) findViewById(R.id.autocomplete_country);
// Get the string array
String[] countries = getResources().getStringArray(R.array.countries_array);
// Create the adapter and set it to the AutoCompleteTextView 
ArrayAdapter<String> adapter = 
        new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, countries);
textView.setAdapter(adapter);

```
##  CheckBox 
**复选框**允许用户从列表中选择一个或多个选项。通常，你应该在垂直的列表中显示每一个选项。
##  Radio Buttons 
**单选按钮**允许用户从一系列选项中选择其中某个选项。如果认为有必要让用户看到所有并列的可选项，并且各个选项中只能有一个被选择，那么单选按钮是个好选择。如果没有必要显示所有的并列选项，那么可以用**spinner下拉列表**代替。
创建单个选项之前，需要在布局文件中创建选项按钮RadioButton。然而，因为单选按钮之间是互斥的，它们需要被放在同一个选项组` RadioGroup `里。这样系统会认为在同一个选项组里每次只能有一个选项被选择。
```

<?xml version="1.0" encoding="utf-8"?>
<RadioGroup xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">
    <RadioButton android:id="@+id/radio_pirates"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/pirates"
        android:onClick="onRadioButtonClicked"/>
    <RadioButton android:id="@+id/radio_ninjas"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/ninjas"
        android:onClick="onRadioButtonClicked"/>
</RadioGroup>

```
##  Toggle Buttons 开关按钮 
![](/data/dokuwiki/android/pasted/20150609-081446.png)![](/data/dokuwiki/android/pasted/20150609-081452.png)
你可以使用` ToggleButton `对象把一个基本的开关按钮添加到布局文件。Android 4.0（API 级别14）引进了另外一种叫做` Switch的开关 `按钮，这种按钮提供了滑动控制的效果，你可以使用Switch对象添加到布局文件来实现。
` ToggleButton和Switch控件都是CompoundButton `组合按钮的子类并且有着相同的功能，所以你可以用同样的方法来实现他们功能。
```

<ToggleButton 
    android:id="@+id/togglebutton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:textOn="Vibrate on"
    android:textOff="Vibrate off"
    android:onClick="onToggleClicked"/>

```
###  使用OnCheckedChangeListener监听器 

```

ToggleButton toggle = (ToggleButton) findViewById(R.id.togglebutton);
toggle.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        if (isChecked) {
            // The toggle is enabled
        } else {
            // The toggle is disabled
        }
    }
});

```
##  Spinners下拉列表
![](/data/dokuwiki/android/pasted/20150609-081744.png)
```

<Spinner
    android:id="@+id/planets_spinner"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content" />
      
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string-array name="planets_array">
        <item>Mercury</item>
        <item>Venus</item>
        <item>Earth</item>
        <item>Mars</item>
        <item>Jupiter</item>
        <item>Saturn</item>
        <item>Uranus</item>
        <item>Neptune</item>
    </string-array>
</resources>

```
要加入选项列表到spinner的话，你要先在你的Activity或者是Fragment的代码里指定一个Adapter。
```

Spinner spinner = (Spinner) findViewById(R.id.spinner);
// Create an ArrayAdapter using the string array and a default spinner layout
ArrayAdapter<CharSequence> adapter = ArrayAdapter.createFromResource(this,
        R.array.planets_array, android.R.layout.simple_spinner_item);
// Specify the layout to use when the list of choices appears
adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
// Apply the adapter to the spinner
spinner.setAdapter(adapter);

```
要给spinner定义选择的动作事件，继承` AdapterView.OnItemSelectedListener `接口和实现相应的onItemSelected()回调函数。例如，这有一个继承了此接口的Activity：
Spinner spinner = (Spinner) findViewById(R.id.spinner);
` spinner.setOnItemSelectedListener(this); `
##  Pickers时间日期选择器 
Android给用户提供了选择时间或日期的对话框控件。每个选择器提供了选择时间（小时，分钟，上午/下午）或日期（月，日，年）的控件。
**我们建议您使用DialogFragment承载每个时间或日期选择器。** ` DialogFragment `能够自动管理对话框的生命周期，并且允许您在不同的布局配置中显示选择器，如在手机上的基本对话框，或大屏幕上的布局嵌入的一部分。
###  继承DialogFragment，创建TimePickerDialog 
```

public static class TimePickerFragment extends DialogFragment
                            implements TimePickerDialog.OnTimeSetListener {
 
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // Use the current time as the default values for the picker
        final Calendar c = Calendar.getInstance();
        int hour = c.get(Calendar.HOUR_OF_DAY);
        int minute = c.get(Calendar.MINUTE);
 
        // Create a new instance of TimePickerDialog and return it
        return new TimePickerDialog(getActivity(), this, hour, minute,
                DateFormat.is24HourFormat(getActivity()));
    }
 
    public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
        // Do something with the time chosen by the user
    }
}

...
  public void showTimePickerDialog(View v) {
    DialogFragment newFragment = new TimePickerFragment();
    newFragment.show(getSupportFragmentManager(), "timePicker");
}

```
**继承DialogFragment，创建DatePickerDialog**:略。。