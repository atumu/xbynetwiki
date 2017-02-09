title: onsaveinstancestate_never_called 

##  onsaveinstancestate never called 
http://stackoverflow.com/questions/30549722/onsaveinstancestate-is-not-getting-called-after-screen-rotation
Unless your app is running on API21+ version of Android, your

public void onSaveInstanceState (Bundle outState, PersistableBundle outPersistentState);
will NOT be called as it simply does not exist on earlier versions of the platform than 21. To support pre API21 devices you must, instead of the above, override the following method:

public void onSaveInstanceState (Bundle outState);
This will work on API21+ as well, so you do not need to override both methods, of course (unless you know you need to deal with PersistableBundle the new one offers).