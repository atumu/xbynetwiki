title: jgroups组播集群通信配置 

#  JGroups组播集群通信配置 
```

<dependency>
    <groupId>org.jgroups</groupId>
    <artifactId>jgroups</artifactId>
    <version>3.6.6.Final</version>
</dependency>

```
JGroups是一个基于Java语言的提供可靠多播(组播)的开发工具包。在IP Multicast基础上提供可靠服务，也可以构建在TCP或者WAN上。主要是由Bela Ban开发，属于JBoss.org，在JBoss的网站也有一些相关文档。目前在 SourceForge上还是比较活跃，经常保持更新。JGroups 适合使用场合---服务器集群cluster、多服务器通讯、服务器replication(复制)等，分布式cache缓存。
JGroups是一个可靠的组间通讯工具，进程可以加入一个通讯组，给组内所有的成员或单独的成员发送消息，同样，也可以从组中的成员处接收消息。 系统会记录组的每一个成员，在新成员加入或是现有的成员离开或是崩溃时，会通知组内的其他成员，这样我们就不必自己去管理这些事情，要想加入一个组，并与 组内其他的成员交互，必须建立一个Channel连接到组，同一个组内的所有成员使用相同的组名称，这样才能进行通信。
GUI界面下测试jgroup是否正常 
To test whether JGroups works okay on your machine, run the following command **twice**:
java -Djava.net.preferIPv4Stack=true org.jgroups.demos.Draw

```

import java.io.BufferedReader;
import java.io.InputStreamReader;
 
import org.jgroups.JChannel;
import org.jgroups.Message;
import org.jgroups.ReceiverAdapter;
import org.jgroups.View;
 
public class SimpleChat extends ReceiverAdapter {
  JChannel channel;
 
  public void viewAccepted(View new_view) {
    System.out.println("** view: " + new_view);
  }
 
  public void receive(Message msg) {
    String line = "[" + msg.getSrc() + "]: " + msg.getObject();
    System.out.println(line);
  }
 
  /** Method called from other app, injecting channel */
  public void start(JChannel ch) throws Exception {
    channel = ch;
    channel.setReceiver(this);
    channel.connect("ChatCluster");
    eventLoop();
    channel.close();
  }
 
  private void start(String props, String name) throws Exception {
    channel = new JChannel(props);
    if (name != null) {
      channel.name(name);
    }
    channel.setReceiver(this);
    channel.connect("ChatCluster");
    eventLoop();
    channel.close();
  }
 
  private void eventLoop() {
    BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
    while (true) {
      try {
        System.out.print("> ");
        System.out.flush();
        String line = in.readLine().toLowerCase();
        if (line.startsWith("quit") || line.startsWith("exit")) {
          break;
        }
        Message msg = new Message(null, null, line);
        channel.send(msg);
      } catch (Exception e) {
      }
    }
  }
 
  public static void main(String[] args) throws Exception {
    String props = "udp.xml";
    String name = null;
 
    for (int i = 0; i < args.length; i++) {
      if (args[i].equals("-props")) {
        props = args[++i];
        continue;
      }
      if (args[i].equals("-name")) {
        name = args[++i];
        continue;
      }
      help();
      return;
    }
 
    new SimpleChat().start(props, name);
  }
 
  protected static void help() {
    System.out.println("SimpleChat [-props XML config] [-name name]");
  }
}

```
配置文件udp.xml
```

<config xmlns="urn:org:jgroups"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/JGroups-3.4.xsd">
 
    <UDP
         mcast_addr="${jgroups.udp.mcast_addr:235.6.6.6}"
         mcast_port="${jgroups.udp.mcast_port:45588}"
         tos="8"
         ucast_recv_buf_size="5M"
         ucast_send_buf_size="1M"
         mcast_recv_buf_size="5M"
         mcast_send_buf_size="1M"
         loopback="true"
         max_bundle_size="64K"
         max_bundle_timeout="30"
         ip_ttl="${jgroups.udp.ip_ttl:2}"
         enable_diagnostics="true"
         thread_naming_pattern="cl"
 
         timer_type="new"
         timer.min_threads="4"
         timer.max_threads="10"
         timer.keep_alive_time="3000"
         timer.queue_max_size="500"
 
         thread_pool.enabled="true"
         thread_pool.min_threads="2"
         thread_pool.max_threads="8"
         thread_pool.keep_alive_time="5000"
         thread_pool.queue_enabled="true"
         thread_pool.queue_max_size="10000"
         thread_pool.rejection_policy="discard"
 
         oob_thread_pool.enabled="true"
         oob_thread_pool.min_threads="1"
         oob_thread_pool.max_threads="8"
         oob_thread_pool.keep_alive_time="5000"
         oob_thread_pool.queue_enabled="false"
         oob_thread_pool.queue_max_size="100"
         oob_thread_pool.rejection_policy="Run"/>
 
    <PING timeout="2000" num_initial_members="3"/>
    <MERGE2 max_interval="30000" min_interval="10000"/>
    <FD_SOCK/>
    <FD_ALL/>
    <VERIFY_SUSPECT timeout="1500"  />
    <BARRIER />
    <pbcast.NAKACK use_mcast_xmit="true"
                   retransmit_timeout="300,600,1200"
                   discard_delivered_msgs="true"/>
 
    <pbcast.STABLE stability_delay="1000"
                   desired_avg_gossip="50000"
                   max_bytes="4M"/>
    <pbcast.GMS print_local_addr="true"
                print_physical_addrs="true"
                join_timeout="3000"
                view_bundling="true"
                max_join_attempts="3"/>
                 
    <UFC max_credits="2M" min_threshold="0.4"/>
    <MFC max_credits="2M" min_threshold="0.4"/>
    <FRAG2 frag_size="60K"  />
    <pbcast.STATE_TRANSFER />
 
</config>

```
**使用的是组播UDP，  运行java加上 -Djava.net.preferIPv4Stack=true 参数**

云服务器上，端口的权限默认没有开放，**你可以用jgroups自带测试程序看看是否能通讯，**先在一台服务器上启动McastReceiverTest
java org.jgroups.tests.McastReceiverTest -mcast_addr 235.6.6.6 -port 45588

然后去另外一个服务器上启动McastSenderTest
java org.jgroups.tests.McastSenderTest -mcast_addr 235.6.6.6 -port 45588
如果在sender端发消息在receiver端能收到，说明是能够通讯的



参考：http://www.jgroups.org/tutorial/index.html
http://www.oschina.net/question/62530_2134183
http://blog.csdn.net/liuzhenwen/article/details/5895032
http://www.jgroups.org/manual/index.html