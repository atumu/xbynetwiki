title: notification显示多行文本 

#   Notification显示多行文本 
```

 Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(AppConfig.APP_DOWNLOAD_SITE + "?pkgName=" + ctx.getPackageName()));
        PendingIntent pendingIntent = PendingIntent.getActivity(ctx, 0, intent, PendingIntent.FLAG_CANCEL_CURRENT);
        NotificationManager mNotificationManager = (NotificationManager) ctx.getSystemService(Context.NOTIFICATION_SERVICE);
        NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(ctx)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle(ctx.getString(R.string.update_notification_title))
                .setContentText(ctx.getString(R.string.update_notification_content))
                .setContentIntent(pendingIntent);
	<!--关键代码开始 -->
        Notification notification= mBuilder
                    .setStyle(new NotificationCompat.BigTextStyle()
                            .bigText(contentStr))
                            .build();
		<!--关键代码结束 -->
        //点击之后移除通知
        notification.flags |= Notification.FLAG_AUTO_CANCEL;
        notification.defaults |= Notification.DEFAULT_SOUND;
        notification.defaults |= Notification.DEFAULT_LIGHTS;
        notification.defaults |= Notification.DEFAULT_VIBRATE;
        mNotificationManager.notify(0x121, notification);

```
##  Notification Type: 
` 如需要兼容4.1以下请使用NotificationCompact.Builder `
` 注意：如果开启了360状态栏显示，则不会起作用，所以需要提前关闭360， `否则会让你很无脑地苦闷。建议使用模拟器测试，而先不要在真机测试，以免出现类似问题。
Basic Notification – Shows simple and short notification with icon.
Big Picture Notification – Shows visual content such as bitmap.
Big Text Notification – Shows multiline Textview object.
Inbox Style Notification – Shows any kind of list, e.g messages, headline etc.
` 再次提示，如需要兼容4.1以下请使用NotificationCompact.Builder替代下述代码 `
` Notification.BigPictureStyle, Notification.BigTextStyle, and Notification.InboxStyle. `
```

 public void sendBasicNotification(View view) {
  Notification notification = new Notification.Builder(this)
    .setContentTitle("Basic Notification")
    .setContentText("Basic Notification, used earlier")
    .setSmallIcon(R.drawable.ic_launcher_share).build();
  notification.flags |= Notification.FLAG_AUTO_CANCEL;
  NotificationManager notificationManager = getNotificationManager();
  notificationManager.notify(0, notification);
 }

 public void sendBigTextStyleNotification(View view) {
  String msgText = "Jeally Bean Notification example!! "
    + "where you will see three different kind of notification. "
    + "you can even put the very long string here.";

  NotificationManager notificationManager = getNotificationManager();
  PendingIntent pi = getPendingIntent();
  Builder builder = new Notification.Builder(this);
  builder.setContentTitle("Big text Notofication")
    .setContentText("Big text Notification")
    .setSmallIcon(R.drawable.ic_launcher)
    .setAutoCancel(true);
    .setPriority(Notification.PRIORITY_HIGH)
    .addAction(R.drawable.ic_launcher_web, "show activity", pi);
  Notification notification = new Notification.BigTextStyle(builder)
    .bigText(msgText).build();
 
  notificationManager.notify(0, notification);
 }

 public void sendBigPictureStyleNotification(View view) {
  PendingIntent pi = getPendingIntent();
  Builder builder = new Notification.Builder(this);
  builder.setContentTitle("BP notification")
    // Notification title
    .setContentText("BigPicutre notification")
    // you can put subject line.
    .setSmallIcon(R.drawable.ic_launcher)
    // Set your notification icon here.
    .addAction(R.drawable.ic_launcher_web, "show activity", pi)
    .addAction(
      R.drawable.ic_launcher_share,
      "Share",
      PendingIntent.getActivity(getApplicationContext(), 0,
        getIntent(), 0, null));

  // Now create the Big picture notification.
  Notification notification = new Notification.BigPictureStyle(builder)
    .bigPicture(
      BitmapFactory.decodeResource(getResources(),
        R.drawable.big_picture)).build();
  // Put the auto cancel notification flag
  notification.flags |= Notification.FLAG_AUTO_CANCEL;
  NotificationManager notificationManager = getNotificationManager();
  notificationManager.notify(0, notification);
 }

 public void sendInboxStyleNotification(View view) {
  PendingIntent pi = getPendingIntent();
  Builder builder = new Notification.Builder(this)
    .setContentTitle("IS Notification")
    .setContentText("Inbox Style notification!!")
    .setSmallIcon(R.drawable.ic_launcher)
    .addAction(R.drawable.ic_launcher_web, "show activity", pi);

  Notification notification = new Notification.InboxStyle(builder)
    .addLine("First message").addLine("Second message")
    .addLine("Thrid message").addLine("Fourth Message")
    .setSummaryText("+2 more").build();
  // Put the auto cancel notification flag
  notification.flags |= Notification.FLAG_AUTO_CANCEL;
  NotificationManager notificationManager = getNotificationManager();
  notificationManager.notify(0, notification);
 }

 public PendingIntent getPendingIntent() {
  return PendingIntent.getActivity(this, 0, new Intent(this,
    HandleNotificationActivity.class), 0);
 }

 public NotificationManager getNotificationManager() {
  return (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
 }
}

```

参考：http://www.javacodegeeks.com/2012/08/android-jelly-bean-notification-tutorial.html
官方文档：http://developer.android.com/guide/topics/ui/notifiers/notifications.html#ApplyStyle