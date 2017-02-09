title: 基于xmpp_openfire_smack的im开发 


#  android基于XMPP Openfire smack的IM开发一 
参考：http://blog.csdn.net/shimiso/article/details/8816558
Java领域的即时通信的解决方案可以考虑openfire+spark+smack。当然也有其他的选择。
Openfire是基于Jabber协议(XMPP)实现的即时通信服务器端版本。
spark官方提供的即时通信客户端。Spark支持聊天，语音，视频，会议，文件收发，截屏，连接msn等功能。
Smack是即时通信客户端编程库
##  1.什么是XMPP 
XMPP(Extensible Messaging and Presence Protocol)，简单的来讲，它就是一个发送接收处理消息的协议，但是这个协议发送的消息，既不是二进制的东东也不是字符串，而是XML。正是因为使用了XML作为消息传递的中介，Extensible 才谈的上。XMPP的前身是Jabber，一个开源形式组织产生的网络即时通信协议。XMPP目前被IETF国际标准组织完成了标准化工作。　　
##  2.IM 

Instant Messenger，及时通信软件，就是大家使用的QQ、MSN Messenger和Gtalk等等。其中Gtalk 就是基于XMPP 协议的一个实现，其他的则不是。当前IM 几乎作为每个上网者必然使用的工具，在国外的大型企业中有一些企业级的IM应用，但是其商业价值还没完全发挥出来。设想既然XMPP 协议是一个公开的协议，那么每个企业都可以利用它来开发适合本身企业工作，提高自身生产效率的IM；甚至，你还可以在网络游戏中集成这种通信软件，不但让你可以边游戏边聊天，也可以开发出适合游戏本身的IM 应用，比如说一些游戏关键场景提醒功能，团队语音交流等等都可以基于IM来实现。
##  3.Spark,smack和Openfire 

开源界总是有许多有趣的东东，这三个合起来就是一个完整的XMPP IM 实现。包括服务器端——Openfire，客户端——Spark，XMPP 传输协议的实现——Smack（记住，XMPP是一个协议，协议是需要实现的，Smack起到的就是这样的一个作用）。三者都是基于Java 语言的实现。
Spark 提供了客户端一个基本的实现，并提出了一个很好的插件架构，这对于开发者来说不能不说是一个福音。我强烈建议基于插件方式来实现你新增加的功能，而不是去改它的源代码，这样有利于你项目架构，把原始项目的影响降到最低。
Openfire 是基于XMPP 协议的IM 的服务器端的一个实现，虽然当两个用户连接后，可以通过点对点的方式来发送消息，但是用户还是需要连接到服务器来获取一些连接信息和通信信息的，所以服务器端是必须要实现的。Openfire 也提供了一些基本功能，但真的很基本的！庆幸的是，它也提供插件的扩展，像Spark 一样，同样强烈建议使用插件扩展的方式来增加新的功能，而不是修改人家的源代码。
Smack 是一个XMPP 协议的Java 实现，提供一套可扩展的API，不过有些时候，你还是不得不使用自己定制发送的XML 文件内容的方式来实现自己的功能
下图展示了三者之间的关系：
![](/data/dokuwiki/android/pasted/20150520-061242.png)
Spark 提供了客户端一个基本的实现，并提出了一个很好的插件架构，这对于开发者来说不能不说是一个福音。我强烈建议基于插件方式来实现你新增加的功能，而不是去改它的源代码，这样有利于你项目架构，把原始项目的影响降到最低。
Openfire 是基于XMPP 协议的IM 的服务器端的一个实现，虽然当两个用户连接后，可以通过点对点的方式来发送消息，但是用户还是需要连接到服务器来获取一些连接信息和通信信息的，所以服务器端是必须要实现的。Openfire 也提供了一些基本功能，但真的很基本的！庆幸的是，它也提供插件的扩展，像Spark 一样。
Smack 是一个XMPP 协议的Java 实现，提供一套可扩展的API，不过有些时候，你还是不得不使用自己定制发送的XML 文件内容的方式来实现自己的功能。
##  4.安装Openfire 
##  5.客户端配置和调试 

这里我没用使用官方的spark客户端，而是用了潘迪安和如意通。
![](/data/dokuwiki/android/pasted/20150520-061357.png)
![](/data/dokuwiki/android/pasted/20150520-061437.png)
从上面的聊天记录我们发现所有的用户id全称都是：用户名@域名/资源名，这个就是我们在XMPP协议中通常说说的JID，即jabber id，它是一个xmpp协议帐号系统的通称，后面我们在使用smack编程库调试接口时会经常用到这个参数。

##  openfire-spark 查找联系人报错：无法连接到搜索服务 
参考：http://blog.csdn.net/sundenskyqq/article/details/8721938
原因：openfire服务器没有安装search插件。
解决：进入openfire后台管理器-->插件-->有效的插件-->安装search服务器，完成以后要刷新缓存：openfire后台管理器-->服务器-->缓存摘要-->刷新Roster缓存（或者刷新全部缓存）。但是如果openfire采用了集群，那么此处的缓存将不存在，需要刷新集群中的缓存。


#  smack类库介绍和使用 
参考：http://blog.csdn.net/shimiso/article/details/8816540
关于Smack编程库，前面我们提到，它是面向Java端的api，利用它我们可以向openfire服务器注册用户，发送消息，并且可以通过监听器获得此用户的应答消息，以及构建聊天室，分组，个人通讯录等等。
##  （1）登录操作 
```

PPConnection.DEBUG_ENABLED = true;  
AccountManager accountManager;  
final ConnectionConfiguration connectionConfig = new ConnectionConfiguration(  
        "192.168.1.78", Integer.parseInt("5222"), "csdn.shimiso.com");  
  
// 允许自动连接  
connectionConfig.setReconnectionAllowed(true);  
connectionConfig.setSendPresence(true);  
  
Connection connection = new XMPPConnection(connectionConfig);  
try {  
    connection.connect();// 开启连接  
    accountManager = connection.getAccountManager();// 获取账户管理类  
} catch (XMPPException e) {  
    throw new IllegalStateException(e);  
}  
  
// 登录  
connection.login("admin", "admin","SmackTest");  
System.out.println(connection.getUser());   
connection.getChatManager().createChat("shimiso@csdn.shimiso.com",null).sendMessage("Hello word!");  

```
在login中一共有三个参数，登录名，密码，资源名，可能有人不明白资源名到底是什么意思，其实就是客户端的来源，客户端的名称，如果不写它默认就叫smack，如果你用相同的账户不同的资源名和同一个人发三条消息，那将会弹出三个窗口，而不是一个窗口。
同时smack还为我们提供了非常好的调试工具Smack Debug，利用该工具我们可以准确的捕获详细的往返报文信息。

##  下面我们继续写个聊天的例子： 
```

PPConnection.DEBUG_ENABLED = true;  
AccountManager accountManager;  
final ConnectionConfiguration connectionConfig = new ConnectionConfiguration(  
        "192.168.1.78", Integer.parseInt("5222"), "csdn.shimiso.com");  
  
// 允许自动连接  
connectionConfig.setReconnectionAllowed(true);  
connectionConfig.setSendPresence(true);  
  
Connection connection = new XMPPConnection(connectionConfig);  
try {  
    connection.connect();// 开启连接  
    accountManager = connection.getAccountManager();// 获取账户管理类  
} catch (XMPPException e) {  
    throw new IllegalStateException(e);  
}  
  
// 登录  
connection.login("admin", "admin","SmackTest3");    
ChatManager chatmanager = connection.getChatManager();  
Chat newChat = chatmanager.createChat("shimiso@csdn.shimiso.com", new MessageListener() {  
    public void processMessage(Chat chat, Message message) {  
        if (message.getBody() != null) {  
            System.out.println("Received from 【"  
                    + message.getFrom() + "】 message: "  
                    + message.getBody());  
        }  
  
    }  
});  
Scanner input = new Scanner(System.in);  
while (true) {  
    String message = input.nextLine();   
    newChat.sendMessage(message);  

```
这里我们用Scanner来捕捉用户在控制台的键盘操作，将信息发出，同时创建了一个MessageListener监听，在其中强制实现processMessage方法即可捕获发回的信息，在初次使用上还是较为容易上手的，我们只要细心查看API即可逐步深入下去。
##  除了聊天以外我们经常还能想到就是广播 
需要给所有在线的用户发送一个通知，或者给所有在线和离线的用户全发送，我们先演示如何给在线用户发送一个广播：
```

PPConnection.DEBUG_ENABLED = false;  
AccountManager accountManager;  
final ConnectionConfiguration connectionConfig = new ConnectionConfiguration(  
        "192.168.1.78", Integer.parseInt("5222"), "csdn.shimiso.com");  
  
// 允许自动连接  
connectionConfig.setReconnectionAllowed(true);  
connectionConfig.setSendPresence(true);  
  
Connection connection = new XMPPConnection(connectionConfig);  
try {  
    connection.connect();// 开启连接  
    accountManager = connection.getAccountManager();// 获取账户管理类  
} catch (XMPPException e) {  
    throw new IllegalStateException(e);  
}  
connection.login("admin", "admin","SmackTest3");   
Message newmsg = new Message();   
newmsg.setTo("shimiso@csdn.shimiso.com");  
newmsg.setSubject("重要通知");  
newmsg.setBody("今天下午2点60分有会！");  
newmsg.setType(Message.Type.headline);// normal支持离线   
connection.sendPacket(newmsg);  
connection.disconnect();  

```
将参数设置为Message.Type.normal即可支持离线广播，openfire系统会自动判断该用户是否在线，如果在线就直接发送出去，如果不在线则将信息存入ofoffline表，现在我将shimiso用户退出登录，再给它发消息，我们可以进入openfire库的ofoffline表中，非常清楚看到里面躺着一条离线消息记录是发给shimiso这个用户的
##  那么我们如何让shimiso这个用户一登陆就取到离线消息呢？ 
```

PPConnection.DEBUG_ENABLED = false;  
AccountManager accountManager;  
final ConnectionConfiguration connectionConfig = new ConnectionConfiguration(  
        "192.168.1.78", Integer.parseInt("5222"), "csdn.shimiso.com");  
  
// 允许自动连接  
connectionConfig.setReconnectionAllowed(true);  
connectionConfig.setSendPresence(false);//不要告诉服务器自己的状态  
Connection connection = new XMPPConnection(connectionConfig);  
try {  
    connection.connect();// 开启连接  
    accountManager = connection.getAccountManager();// 获取账户管理类  
} catch (XMPPException e) {  
    throw new IllegalStateException(e);  
}   
connection.login("shimiso", "123","SmackTest");   
OfflineMessageManager offlineManager = new OfflineMessageManager(  
        connection);  
try {  
    Iterator<org.jivesoftware.smack.packet.Message> it = offlineManager  
            .getMessages();  
  
    System.out.println(offlineManager.supportsFlexibleRetrieval());  
    System.out.println("离线消息数量: " + offlineManager.getMessageCount());  
  
    Map<String, ArrayList<Message>> offlineMsgs = new HashMap<String, ArrayList<Message>>();  
  
    while (it.hasNext()) {  
        org.jivesoftware.smack.packet.Message message = it.next();  
        System.out  
                .println("收到离线消息, Received from 【" + message.getFrom()  
                        + "】 message: " + message.getBody());  
        String fromUser = message.getFrom().split("/")[0];  
  
        if (offlineMsgs.containsKey(fromUser)) {  
            offlineMsgs.get(fromUser).add(message);  
        } else {  
            ArrayList<Message> temp = new ArrayList<Message>();  
            temp.add(message);  
            offlineMsgs.put(fromUser, temp);  
        }  
    }  
  
    // 在这里进行处理离线消息集合......  
    Set<String> keys = offlineMsgs.keySet();  
    Iterator<String> offIt = keys.iterator();  
    while (offIt.hasNext()) {  
        String key = offIt.next();  
        ArrayList<Message> ms = offlineMsgs.get(key);  
  
        for (int i = 0; i < ms.size(); i++) {  
            System.out.println("-->" + ms.get(i));  
        }  
    }  
  
    offlineManager.deleteMessages();  
} catch (Exception e) {  
    e.printStackTrace();  
}  
offlineManager.deleteMessages();//删除所有离线消息  
Presence presence = new Presence(Presence.Type.available);  
            nnection.sendPacket(presence);//上线了  
            nnection.disconnect();//关闭连接  

```
` 这里我们需要特别当心的是先不要告诉openfire服务器你上线了，否则永远也拿不到离线消息，用下面英文大概意思就是在你上线之前去获取离线消息，这么设计是很有道理的。 `
The OfflineMessageManager helps manage offline messages even before the user has sent an available presence. When a user asks for his offline messages before sending an available presence then the server will not send a flood with all the offline messages when the user becomes online. The server will not send a flood with all the offline messages to the session that made the offline messages request or to any other session used by the user that becomes online.
拿到离线消息处理完毕之后删除离线消息offlineManager.deleteMessages() 接着通知服务器上线了。
##  下面我们来看看如何来发送文件 
```

PPConnection.DEBUG_ENABLED = false;  
AccountManager accountManager;  
final ConnectionConfiguration connectionConfig = new ConnectionConfiguration(  
        "192.168.1.78", Integer.parseInt("5222"), "csdn.shimiso.com");  
  
// 允许自动连接  
connectionConfig.setReconnectionAllowed(true);  
connectionConfig.setSendPresence(true);  
  
Connection connection = new XMPPConnection(connectionConfig);  
try {  
    connection.connect();// 开启连接  
    accountManager = connection.getAccountManager();// 获取账户管理类  
} catch (XMPPException e) {  
    throw new IllegalStateException(e);  
}  
  connection.login("admin", "admin","Rooyee");   
  Presence pre = connection.getRoster().getPresence("shimiso@csdn.shimiso.com");  
    System.out.println(pre);  
    if (pre.getType() != Presence.Type.unavailable) {  
        // 创建文件传输管理器  
        FileTransferManager manager = new FileTransferManager(connection);  
        // 创建输出的文件传输  
        OutgoingFileTransfer transfer = manager  
                .createOutgoingFileTransfer(pre.getFrom());  
        // 发送文件  
        transfer.sendFile(new File("E:\\Chrysanthemum.jpg"), "图片");  
        while (!transfer.isDone()) {  
            if (transfer.getStatus() == FileTransfer.Status.in_progress) {  
                // 可以调用transfer.getProgress();获得传输的进度　  
                System.out.println(transfer.getStatus());  
                System.out.println(transfer.getProgress());  
                System.out.println(transfer.isDone());  
            }  
        }  
    }  

```
在这里我们需要特别注意的是，跨资源是无法发送文件的，看connection.login("admin", "admin","Rooyee");这个代码就明白了，必须是“域名和资源名”完全相同的两个用户才可以互发文件，否则永远都没反应，如果不清楚自己所用的客户端的资源名，可以借助前面提到的SmackDebug工具查看往返信息完整报文，在to和from中一定可以看到。
##  如果我们自己要写文件接收例子的话 
，参考代码如下：
```

FileTransferManager transfer = new FileTransferManager(connection);  
transfer.addFileTransferListener(new RecFileTransferListener());  
public class RecFileTransferListener implements FileTransferListener {  
  
    public String getFileType(String fileFullName) {  
        if (fileFullName.contains(".")) {  
            return "." + fileFullName.split("//.")[1];  
        } else {  
            return fileFullName;  
        }  
  
    }  
  
    @Override  
    public void fileTransferRequest(FileTransferRequest request) {  
        System.out.println("接收文件开始.....");  
        final IncomingFileTransfer inTransfer = request.accept();  
        final String fileName = request.getFileName();  
        long length = request.getFileSize();  
        final String fromUser = request.getRequestor().split("/")[0];  
        System.out.println("文件大小:" + length + "  " + request.getRequestor());  
        System.out.println("" + request.getMimeType());  
        try {  
  
            JFileChooser chooser = new JFileChooser();  
            chooser.setCurrentDirectory(new File("."));  
  
            int result = chooser.showOpenDialog(null);  
  
            if (result == JFileChooser.APPROVE_OPTION) {  
                final File file = chooser.getSelectedFile();  
                System.out.println(file.getAbsolutePath());  
                new Thread() {  
                    public void run() {  
                        try {  
  
                            System.out.println("接受文件: " + fileName);  
                            inTransfer  
                                    .recieveFile(new File(file  
                                            .getAbsolutePath()  
                                            + getFileType(fileName)));  
  
                            Message message = new Message();  
                            message.setFrom(fromUser);  
                            message.setProperty("REC_SIGN", "SUCCESS");  
                            message.setBody("[" + fromUser + "]发送文件: "  
                                    + fileName + "/r/n" + "存储位置: "  
                                    + file.getAbsolutePath()  
                                    + getFileType(fileName));  
                            if (Client.isChatExist(fromUser)) {  
                                Client.getChatRoom(fromUser)  
                                        .messageReceiveHandler(message);  
                            } else {  
                                ChatFrameThread cft = new ChatFrameThread(  
                                        fromUser, message);  
                                cft.start();  
  
                            }  
                        } catch (Exception e2) {  
                            e2.printStackTrace();  
                        }  
                    }  
                }.start();  
            } else {  
  
                System.out.println("拒绝接受文件: " + fileName);  
  
                request.reject();  
                Message message = new Message();  
                message.setFrom(fromUser);  
                message.setBody("拒绝" + fromUser + "发送文件: " + fileName);  
                message.setProperty("REC_SIGN", "REJECT");  
                if (Client.isChatExist(fromUser)) {  
                    Client.getChatRoom(fromUser).messageReceiveHandler(message);  
                } else {  
                    ChatFrameThread cft = new ChatFrameThread(fromUser, message);  
                    cft.start();  
                }  
            }  
  
            /* 
             * InputStream in = inTransfer.recieveFile(); 
             *  
             * String fileName = "r"+inTransfer.getFileName(); 
             *  
             * OutputStream out = new FileOutputStream(new 
             * File("d:/receive/"+fileName)); byte[] b = new byte[512]; 
             * while(in.read(b) != -1) { out.write(b); out.flush(); } 
             *  
             * in.close(); out.close(); 
             */  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
  
        System.out.println("接收文件结束.....");  
  
    }  
  
}  

```
##  用户列表 
```

**  
 * 返回所有组信息 <RosterGroup>  
 *   
 * @return List(RosterGroup)  
 */  
public static List<RosterGroup> getGroups(Roster roster) {  
    List<RosterGroup> groupsList = new ArrayList<RosterGroup>();  
    Collection<RosterGroup> rosterGroup = roster.getGroups();  
    Iterator<RosterGroup> i = rosterGroup.iterator();  
    while (i.hasNext())  
        groupsList.add(i.next());  
    return groupsList;  
}  
  
/** 
 * 返回相应(groupName)组里的所有用户<RosterEntry> 
 *  
 * @return List(RosterEntry) 
 */  
public static List<RosterEntry> getEntriesByGroup(Roster roster,  
        String groupName) {  
    List<RosterEntry> EntriesList = new ArrayList<RosterEntry>();  
    RosterGroup rosterGroup = roster.getGroup(groupName);  
    Collection<RosterEntry> rosterEntry = rosterGroup.getEntries();  
    Iterator<RosterEntry> i = rosterEntry.iterator();  
    while (i.hasNext())  
        EntriesList.add(i.next());  
    return EntriesList;  
}  
  
/** 
 * 返回所有用户信息 <RosterEntry> 
 *  
 * @return List(RosterEntry) 
 */  
public static List<RosterEntry> getAllEntries(Roster roster) {  
    List<RosterEntry> EntriesList = new ArrayList<RosterEntry>();  
    Collection<RosterEntry> rosterEntry = roster.getEntries();  
    Iterator<RosterEntry> i = rosterEntry.iterator();  
    while (i.hasNext())  
        EntriesList.add(i.next());  
    return EntriesList;  
}  

```
##  用户头像的获取 
使用VCard，很强大，具体自己看API吧，可以看看VCard传回来XML的组成，含有很多信息的
```

**  
 * 获取用户的vcard信息  
 * @param connection  
 * @param user  
 * @return  
 * @throws XMPPException  
 */  
public static VCard getUserVCard(XMPPConnection connection, String user) throws XMPPException  
{  
    VCard vcard = new VCard();  
    vcard.load(connection, user);  
      
    return vcard;  
}  
  
/** 
 * 获取用户头像信息 
 */  
public static ImageIcon getUserImage(XMPPConnection connection, String user) {  
    ImageIcon ic = null;  
    try {  
        System.out.println("获取用户头像信息: "+user);  
        VCard vcard = new VCard();  
        vcard.load(connection, user);  
          
        if(vcard == null || vcard.getAvatar() == null)  
        {  
            return null;  
        }  
        ByteArrayInputStream bais = new ByteArrayInputStream(  
                vcard.getAvatar());  
        Image image = ImageIO.read(bais);  
  
          
        ic = new ImageIcon(image);  
        System.out.println("图片大小:"+ic.getIconHeight()+" "+ic.getIconWidth());  
      
    } catch (Exception e) {  
        e.printStackTrace();  
    }  
    return ic;  
}  

```
##  组操作和用户分组操作 
```

**  
 * 添加一个组  
 */  
public static boolean addGroup(Roster roster,String groupName)  
{  
    try {  
        roster.createGroup(groupName);  
        return true;  
    } catch (Exception e) {  
        e.printStackTrace();  
        return false;  
    }  
}  
  
/** 
 * 删除一个组 
 */  
public static boolean removeGroup(Roster roster,String groupName)  
{  
    return false;  
}  
  
/** 
 * 添加一个好友  无分组 
 */  
public static boolean addUser(Roster roster,String userName,String name)  
{  
    try {  
        roster.createEntry(userName, name, null);  
        return true;  
    } catch (Exception e) {  
        e.printStackTrace();  
        return false;  
    }  
}  
/** 
 * 添加一个好友到分组 
 * @param roster 
 * @param userName 
 * @param name 
 * @return 
 */  
public static boolean addUser(Roster roster,String userName,String name,String groupName)  
{  
    try {  
        roster.createEntry(userName, name,new String[]{ groupName});  
        return true;  
    } catch (Exception e) {  
        e.printStackTrace();  
        return false;  
    }  
}  
  
/** 
 * 删除一个好友 
 * @param roster 
 * @param userName 
 * @return 
 */  
public static boolean removeUser(Roster roster,String userName)  
{  
    try {  
          
        if(userName.contains("@"))  
        {  
            userName = userName.split("@")[0];  
        }  
        RosterEntry entry = roster.getEntry(userName);  
        System.out.println("删除好友:"+userName);  
        System.out.println("User: "+(roster.getEntry(userName) == null));  
        roster.removeEntry(entry);  
          
        return true;  
    } catch (Exception e) {  
        e.printStackTrace();  
        return false;  
    }  
      
}  

```
##  用户查询 
```

public static List<UserBean> searchUsers(XMPPConnection connection,String serverDomain,String userName) throws XMPPException  
    {  
        List<UserBean> results = new ArrayList<UserBean>();  
        System.out.println("查询开始..............."+connection.getHost()+connection.getServiceName());  
          
        UserSearchManager usm = new UserSearchManager(connection);  
          
          
        Form searchForm = usm.getSearchForm(serverDomain);  
        Form answerForm = searchForm.createAnswerForm();  
        answerForm.setAnswer("Username", true);  
        answerForm.setAnswer("search", userName);  
        ReportedData data = usm.getSearchResults(answerForm, serverDomain);  
           
         Iterator<Row> it = data.getRows();  
         Row row = null;  
         UserBean user = null;  
         while(it.hasNext())  
         {  
             user = new UserBean();  
             row = it.next();  
             user.setUserName(row.getValues("Username").next().toString());  
             user.setName(row.getValues("Name").next().toString());  
             user.setEmail(row.getValues("Email").next().toString());  
             System.out.println(row.getValues("Username").next());  
             System.out.println(row.getValues("Name").next());  
             System.out.println(row.getValues("Email").next());  
             results.add(user);  
             //若存在，则有返回,UserName一定非空，其他两个若是有设，一定非空  
         }  
           
         return results;  
    }  

```
##  修改自身状态 
包括上线，隐身，对某人隐身，对某人上线
```

	ublic static void updateStateToAvailable(XMPPConnection connection)
	{
		Presence presence = new Presence(Presence.Type.available);
        		nnection.sendPacket(presence);
     	
	
	public static void updateStateToUnAvailable(XMPPConnection connection)
	{
		Presence presence = new Presence(Presence.Type.unavailable);
        		nnection.sendPacket(presence);
    	}
	
	public static void updateStateToUnAvailableToSomeone(XMPPConnection connection,String userName)
	{
		Presence presence = new Presence(Presence.Type.unavailable);
		presence.setTo(userName);
        		nnection.sendPacket(presence);
	}
	public static void updateStateToAvailableToSomeone(XMPPConnection connection,String userName)
	{
		Presence presence = new Presence(Presence.Type.available);
		presence.setTo(userName);
        		nnection.sendPacket(presence);

	}

```
##  心情修改 
```

**  
 * 修改心情  
 * @param connection  
 * @param status  
 */  
public static void changeStateMessage(XMPPConnection connection,String status)  
{  
    Presence presence = new Presence(Presence.Type.available);  
    presence.setStatus(status);  
    connection.sendPacket(presence);  
  
}  

```
##  修改用户头像 
有点麻烦，主要是读入图片文件，编码，传输之
```

public static void changeImage(XMPPConnection connection,File f) throws XMPPException, IOException{  
      
        VCard vcard = new VCard();  
        vcard.load(connection);  
          
            byte[] bytes;  
            
                bytes = getFileBytes(f);  
                String encodedImage = StringUtils.encodeBase64(bytes);  
                vcard.setAvatar(bytes, encodedImage);  
                vcard.setEncodedImage(encodedImage);  
                vcard.setField("PHOTO", "<TYPE>image/jpg</TYPE><BINVAL>"  
                        + encodedImage + "</BINVAL>", true);  
                  
                  
                ByteArrayInputStream bais = new ByteArrayInputStream(  
                        vcard.getAvatar());  
                Image image = ImageIO.read(bais);  
                ImageIcon ic = new ImageIcon(image);  
                   
             
            
            vcard.save(connection);  
             
    }  
      
      private static byte[] getFileBytes(File file) throws IOException {  
                BufferedInputStream bis = null;  
            try {  
            bis = new BufferedInputStream(new FileInputStream(file));  
            int bytes = (int) file.length();  
            byte[] buffer = new byte[bytes];  
            int readBytes = bis.read(buffer);  
            if (readBytes != buffer.length) {  
                throw new IOException("Entire file not read");  
            }  
            return buffer;  
        } finally {  
            if (bis != null) {  
                bis.close();  
            }  
        }  
}  

```
##  用户状态的监听 
即对方改变头像，状态，心情时，更新自己用户列表，其实这里已经有smack实现的监听器
```

		nal Roster roster = Client.getRoster();
		
		roster.addRosterListener(
			new RosterListener() {

					@Override
					public void entriesAdded(Collection<String> arg0) {
						// TODO Auto-generated method stub
						System.out.println("--------EE:"+"entriesAdded");
					}

					@Override
					public void entriesDeleted(Collection<String> arg0) {
						// TODO Auto-generated method stub
						System.out.println("--------EE:"+"entriesDeleted");
					}

					@Override
					public void entriesUpdated(Collection<String> arg0) {
						// TODO Auto-generated method stub
						System.out.println("--------EE:"+"entriesUpdated");
					}

					@Override
					public void presenceChanged(Presence arg0) {
						// TODO Auto-generated method stub
						System.out.println("--------EE:"+"presenceChanged");
					}   
					
		});


```