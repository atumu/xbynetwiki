title: android官方文档学习之dialog 

#  android官方文档学习之Dialog 
Dialog类是创建对话框的基类。但是，您通常不应该直接实例化一个对话框。相反，你应该使用下面的子类中的一个：
` AlertDialog `（弹出对话框）:对话框可以显示一个标题和最多三个按钮，还包括可选的项目列表以及自定义布局。
` DatePickerDialog `（日期选择对话框）及` TimePickerDialog `（时间选择对话框）:允许用户选择一个日期或者时间的对话框。
这些类定义对话框的风格和结构，但你要使用` DialogFragment作为对话框的容器。 `**DialogFragment类提供了创建对话框并管理其外观需要的所有控件，而不用再调用Dialog的方法。**
**使用DialogFragment管理对话框可以确保它正确处理生命周期事件，比如用户按下返回按钮或旋转屏幕等**。DialogFragment类还允许对话框的用户界面就像传统片段那样，作为一个更大的用户界面内的可嵌入组件重新使用。
##  创建对话片段 
例如，下方提供了一个在` DialogFragment `中管理的基本AlertDialog：
```

public class FireMissilesDialogFragment extends DialogFragment {
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // Use the Builder class for convenient dialog construction
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder.setMessage(R.string.dialog_fire_missiles)
               .setPositiveButton(R.string.fire, new DialogInterface.OnClickListener() {
                   public void onClick(DialogInterface dialog, int id) {
                       // FIRE ZE MISSILES!
                   }
               })
               .setNegativeButton(R.string.cancel, new DialogInterface.OnClickListener() {
                   public void onClick(DialogInterface dialog, int id) {
                       // User cancelled the dialog
                   }
               });
        // Create the AlertDialog object and return it
        return builder.create();
    }
}

```
现在，当你创建这个类以及show（）该对象的实例时，出现的对话框如图1。
##  添加按钮 

基于对话复杂程度，可以实现DialogFragment中包括所有基本片段生命周期方法在内的其他回调方法。
可以添加下列三种不同的操作按钮：
肯定
使用此按钮来接受并继续进行操作（ "OK"操作）。
否定
使用此按钮来取消操作。
中立
当用户不希望继续操作时使用此按钮，但并不一定取消操作。例如操作可能显示“稍后提醒”。
##  添加列表 
AlertDialog API有三种类型可用列表：
  * 传统单选项列表
  * 永久单选项列表（单选按钮）
  * 永久多选项列表（复选框）
![](/data/dokuwiki/android/pasted/20150608-145546.png)
要创建图3所示单选项列表，请使用` setItems() ` 方法：
 
```

@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
    builder.setTitle(R.string.pick_color)
           .setItems(R.array.colors_array, new DialogInterface.OnClickListener() {
               public void onClick(DialogInterface dialog, int which) {
               // The 'which' argument contains the index position
               // of the selected item
           }
    });
    return builder.create();
}

```
**因为列表在对话框内容区域显示，对话框不能同时显示消息和列表**，你应该为setTitle()对话框设置一个标题。要指定列表项目，可以调用` setItems() `来传递数组。或者可以使用` setAdapter() `指定列表。这样可以使用 ` ListAdapter `返回动态数据列表。
` 如果用 ListAdapter返回列表，则要使用Loader以便内容加载异步 `。
##  添加持久多选或单选列表 
![](/data/dokuwiki/android/pasted/20150608-145603.png)
要添加多项选择（复选框）或者单项选择（单选按钮）列表，分别使用` setMultiChoiceItems()或者setSingleChoiceItems() `方式。
```

@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    mSelectedItems = new ArrayList();  // Where we track the selected items
    AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
    // Set the dialog title
    builder.setTitle(R.string.pick_toppings)
    // Specify the list array, the items to be selected by default (null for none),
    // and the listener through which to receive callbacks when items are selected
           .setMultiChoiceItems(R.array.toppings, null,
                      new DialogInterface.OnMultiChoiceClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int which,
                       boolean isChecked) {
                   if (isChecked) {
                       // If the user checked the item, add it to the selected items
                       mSelectedItems.add(which);
                   } else if (mSelectedItems.contains(which)) {
                       // Else, if the item is already in the array, remove it 
                       mSelectedItems.remove(Integer.valueOf(which));
                   }
               }
           })
    // Set the action buttons
           .setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int id) {
                   // User clicked OK, so save the mSelectedItems results somewhere
                   // or return them to the component that opened the dialog
                   ...
               }
           })
           .setNegativeButton(R.string.cancel, new DialogInterface.OnClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int id) {
                   ...
               }
           });
 
    return builder.create();
}

```
##  创建自定义布局 
![](/data/dokuwiki/android/pasted/20150608-145627.png)
如果想要对话框自定义布局，可以创建一个布局并调用 ` AlertDialog.Builder对象的setView() `添加到 AlertDialog。
默认情况下自定义布局填充对话框窗口，但是仍可以使用AlertDialog.Builder方法来添加按钮和标题。
例如下面是图5对话框的布局文件： res/layout/dialog_signin.xml
```

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">
    <ImageView
        android:src="@drawable/header_logo"
        android:layout_width="match_parent"
        android:layout_height="64dp"
        android:scaleType="center"
        android:background="#FFFFBB33"
        android:contentDescription="@string/app_name" />
    <EditText
        android:id="@+id/username"
        android:inputType="textEmailAddress"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginLeft="4dp"
        android:layout_marginRight="4dp"
        android:layout_marginBottom="4dp"
        android:hint="@string/username" />
    <EditText
        android:id="@+id/password"
        android:inputType="textPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:layout_marginLeft="4dp"
        android:layout_marginRight="4dp"
        android:layout_marginBottom="16dp"
        android:fontFamily="sans-serif"
        android:hint="@string/password"/>
</LinearLayout>

```
```

@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {
    AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
    // Get the layout inflater
    LayoutInflater inflater = getActivity().getLayoutInflater();
 
    // Inflate and set the layout for the dialog
    // Pass null as the parent view because its going in the dialog layout
    builder.setView(inflater.inflate(R.layout.dialog_signin, null))
    // Add action buttons
           .setPositiveButton(R.string.signin, new DialogInterface.OnClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int id) {
                   // sign in the user ...
               }
           })
           .setNegativeButton(R.string.cancel, new DialogInterface.OnClickListener() {
               public void onClick(DialogInterface dialog, int id) {
                   LoginDialogFragment.this.getDialog().cancel();
               }
           });      
    return builder.create();
}

```
<note>` 提示：如果想要自定义对话框，就用活动来作为对话框 `，而不是使用对话框API。简单创建一个活动并在<activity>清单元素设置主题为` Theme.Holo.Dialog `。
` <activity android:theme="@android:style/Theme.Holo.Dialog" > `</note>
##  Passing Events Back to the Dialog's Host 
当用户触摸对话框操作按钮之一或者选择列表中的项目时，DialogFragment可能会执行必要操作，但是往往会向活动或片段传递事件。要做到这一点，就需要定义每种类型的单击事件的方法接口。然后在要接收操作事件的主机组件实现该接口。
` 即定义回调接口 `
例如DialogFragment定义了提供返回主机活动的事件的界面：
```

public class NoticeDialogFragment extends DialogFragment {
 
    /* The activity that creates an instance of this dialog fragment must
     * implement this interface in order to receive event callbacks.
     * Each method passes the DialogFragment in case the host needs to query it. */
    public interface NoticeDialogListener {
        public void onDialogPositiveClick(DialogFragment dialog);
        public void onDialogNegativeClick(DialogFragment dialog);
    }
 
    // Use this instance of the interface to deliver action events
    NoticeDialogListener mListener;
     @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        // Build the dialog and set up the button click handlers
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder.setMessage(R.string.dialog_fire_missiles)
               .setPositiveButton(R.string.fire, new DialogInterface.OnClickListener() {
                   public void onClick(DialogInterface dialog, int id) {
                       // Send the positive button event back to the host activity
                       mListener.onDialogPositiveClick(NoticeDialogFragment.this);
                   }
               })
               .setNegativeButton(R.string.cancel, new DialogInterface.OnClickListener() {
                   public void onClick(DialogInterface dialog, int id) {
                       // Send the negative button event back to the host activity
                       mListener.onDialogNegativeClick(NoticeDialogFragment.this);
                   }
               });
        return builder.create();
    }

    // Override the Fragment.onAttach() method to instantiate the NoticeDialogListener
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        // Verify that the host activity implements the callback interface
        try {
            // Instantiate the NoticeDialogListener so we can send events to the host
            mListener = (NoticeDialogListener) activity;
        } catch (ClassCastException e) {
            // The activity doesn't implement the interface, throw exception
            throw new ClassCastException(activity.toString()
                    + " must implement NoticeDialogListener");
        }
    }
    ...
}

```
```

public class MainActivity extends FragmentActivity
                          implements NoticeDialogFragment.NoticeDialogListener{
    ...
 
    public void showNoticeDialog() {
        // Create an instance of the dialog fragment and show it
        DialogFragment dialog = new NoticeDialogFragment();
        dialog.show(getSupportFragmentManager(), "NoticeDialogFragment");
    }
 
    // The dialog fragment receives a reference to this Activity through the
    // Fragment.onAttach() callback, which it uses to call the following methods
    // defined by the NoticeDialogFragment.NoticeDialogListener interface
    @Override
    public void onDialogPositiveClick(DialogFragment dialog) {
        // User touched the dialog's positive button
        ...
    }
 
    @Override
    public void onDialogNegativeClick(DialogFragment dialog) {
        // User touched the dialog's negative button
        ...
    }
}

```
因为主机活动实现了由 ` onAttach() `执行的回调方式NoticeDialogListener，**对话框片段可以使用接口回调方法来传递事件**。
##  显示对话框 
```

public void confirmFireMissiles() {
    DialogFragment newFragment = new FireMissilesDialogFragment();
    newFragment.show(getSupportFragmentManager(), "missiles");
}

```
##  关闭对话框 
当用户触摸AlertDialog.Builder创建的任何操作按钮时，系统会关闭对话框。
当用户触摸对话框（除单选按钮和复选框之外）列表中的项目时，系统也会关闭对话框。否则可以在DialogFragment调用` dismiss() `来关闭对话框。
如果对话框消失时需要执行某些操作，就可以在DialogFragment实现` onDismiss() `方法。
可以取消对话框。如果用户按下返回按钮，触摸对话框区域外的屏幕就会进行这一操作。如果显性调用对话框的` cancel( ) `（比如相应对话框的“取消”按钮）。