title: linux_postfix搭建邮箱服务器 

#   linux使用postfix搭建邮箱服务器 
首先说明基本的背景知识。一个邮件服务器通常包括如下两个基本组件：
Mail Transfer Agent (MTA)，用于向收件人的目标 agent 发送邮件和接收来自其他 agent 的邮件。我们使用 Postfix 作为 MTA，它比 sendmail 更安全高效，且在 Ubuntu 平台上官方源提供更新。
Mail Delivery Agent (MDA)，用于用户到服务器上访问自己的邮件。我们使用 Dovecot 作为 MDA，它在 Ubuntu 平台上也是官方源提供更新。
##  Install 

''# apt-get install postfix  
# apt-get install dovecot-common  
# apt-get install dovecot-imapd dovecot-pop3d''
##  Postfix 基本设置 

编辑 /etc/postfix/main.cf 文件，做如下更改：
1. 为支持 TLS 安全连接，确保证书可用（通常默认安装已生成相应文件）
''smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem  
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key  
smtpd_use_tls=yes ''
使用安全连接可保证通过客户端发送邮件时不被截获和窃取。
2. 保证邮件服务器的域名存在于下述列表中
` mydestination = xby1993.net, lab, localhost.localdomain, localhost  ` 
这样收件人为该域名的邮件才会被服务器留存而不是转给其他 MTA。
3. 侦听所有网口
` inet_interfaces = all  `
4. 使用 Maildir 格式存放数据
` home_mailbox = Maildir/ `  
这种格式的好处是邮件分开存放，MDT 访问时不必加锁。而且有些 MDT 仅支持该格式。
5.  配置邮箱和信件大小限制
''mailbox_size_limit = 2000000000  
message_size_limit = 20000000 ''
这里设置邮箱大小为 2 GB，邮件大小为 20 MB。
最后，执行如下命令使上述配置生效：
` sudo service postfix reload  `

##  验证 Postfix 和添加账户 
` telnet localhost smtp  `
可看到如下输出
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 lab ESMTP Postfix (Ubuntu)
进而输入邮件内容，可以发送给任意已有 linux 用户：
mail from: root@[YourDomain]  
rcpt to: [UserName]@[YourDomain]  
data  
Subject: Hello  
Hi, how are you?  
Regards,  
Admin.  
.  
quit  
[optional]
此时在该用户的 home 目录下，应当可以看到 Maildir 目录。进入 Maildir/new 下可以看到刚才的邮件（文本文件），用任意文本编辑器即可查看其内容。如需其他邮件账户，只需正常添加 linux 用户即可。通常，我们可以把这些专用于邮件的用户的 home 目录集中到一起，命令行如下：
useradd -m -d /home/mail-users/[UserName] -g mail-users [UserName]  
这里将邮件账户的 home 目录都放置在了 /home/mail-users 下。

##  dovecot 基本设置 
编辑 /etc/dovecot/dovecot.conf 文件，做如下更改：

1. 使用 maildir 格式（与 postfix 格式对应）
` mail_location = maildir:~/Maildir  ` 
2. 侦听所有默认端口
` listen = *  `
3. 设置安全的远程访问
为了使用户可以远程访问，必须开启基于用户名 / 密码的验证：
` disable_plaintext_auth = no `
但与此同时，由于用户名 / 密码都是明文，我们应该要求建立安全连接以防止信息泄露
''ssl = required  
ssl_cert_file = /etc/ssl/certs/dovecot.pem  
ssl_key_file = /etc/ssl/private/dovecot.pem ''
先前 Postfix 使用的是 TLS，这里的 SSL 与之类似。确保上述文件存在且可用，通常默认文件已经生成好。
最后，别忘使更改的配置生效：
` sudo service dovecot reload `
##  增加 SMTP 验证（重要） 

增加此项验证，可以防止恶意或垃圾邮件通过你的 MTA 进行传递。我们使用的验证机制是 SASL，采用和接收邮件账户相同的用户名 / 密码，匿名用户将被拒绝。

同样需要更改文件 /etc/postfix/main.cf，或者直接使用如下命令（两者等效）：
''sudo postconf -e 'smtpd_sasl_auth_enable = yes'
sudo postconf -e 'smtpd_sasl_type = dovecot'
sudo postconf -e 'smtpd_sasl_path = private/auth'
sudo postconf -e 'smtpd_sasl_security_options = noanonymous'
sudo postconf -e 'broken_sasl_auth_clients = yes'
sudo postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'''
相应地，dovecot 需要开放验证接口，编辑文件 /etc/dovecot/dovecot.conf，在 auth default 节中添加如下行，形如：
''auth default {  
  ......  
  socket listen {  
    client {  
      path = /var/spool/postfix/private/auth  
      mode = 0660  
      user = postfix  
      group = postfix  
    }  
  }  
  ......  
}''
**(参考文章里提到为了保证用户名 / 密码安全，建议发送邮件服务器也强制使用安全连接sudo postconf -e 'smtpd_tls_auth_only = yes'  。` 但是经过本人测试，这样会导致收不到outlook邮件。所以我们实际过程中，不应该添加这项。 `）**
最后别忘使配置生效：
'' sudo service postfix reload  
sudo service dovecot reload '' 
##  已完成？还没有，问题还有 
##  postfix搭建好后，客户端无法接发邮件解决 
1、确认安装 dovecot 
2、以下需要包含发件者的 IP, 允许所有 IP 可以用 0.0.0.0/24
mynetworks
更改/etc/postfix/main.cf：
` mynetworks=0.0.0.0/24 `
3、配置域名MX记录：
参考设置：![](/data/dokuwiki/linux/pasted/20150601-204906.png)

但是我只是将` @ mx 我的ip `也成功了。
4、采用**ThundBird**客户端登录，outlook登陆测试失败。后续会介绍outlook设置方法
##  配置文件示lie 
**/etc/postfix/main.cf:**
```

# See /usr/share/postfix/main.cf.dist for a commented, more complete version


# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname
myorigin = xby1993.net
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

myhostname = xby1993.net
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = ebs-28910, localhost.localdomain, , localhost, xby1993.net
relayhost = 
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 0.0.0.0/24
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

home_mailbox = Maildir/
smtpd_sasl_auth_enable = yes
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
#为了防止outlook邮件接收问题。建议注释下面那句
#smtpd_tls_auth_only = yes

```

**/etc/dovecot/dovecot.conf**
```

## Dovecot configuration file

# If you're in a hurry, see http://wiki2.dovecot.org/QuickConfiguration

# "doveconf -n" command gives a clean output of the changed settings. Use it
# instead of copy&pasting files when posting to the Dovecot mailing list.

# '#' character and everything after it is treated as comments. Extra spaces
# and tabs are ignored. If you want to use either of these explicitly, put the
# value inside quotes, eg.: key = "# char and trailing whitespace  "

# Default values are shown for each setting, it's not required to uncomment
# those. These are exceptions to this though: No sections (e.g. namespace {})
# or plugin settings are added by default, they're listed only as examples.
# Paths are also just examples with the real defaults being based on configure
# options. The paths listed here are for configure --prefix=/usr
# --sysconfdir=/etc --localstatedir=/var

# Enable installed protocols
!include_try /usr/share/dovecot/protocols.d/*.protocol

# A comma separated list of IPs or hosts where to listen in for connections. 
# "*" listens in all IPv4 interfaces, "::" listens in all IPv6 interfaces.
# If you want to specify non-default ports or anything more complex,
# edit conf.d/master.conf.
#listen = *, ::

# Base directory where to store runtime data.
#base_dir = /var/run/dovecot/

# Name of this instance. Used to prefix all Dovecot processes in ps output.
#instance_name = dovecot

# Greeting message for clients.
#login_greeting = Dovecot ready.

# Space separated list of trusted network ranges. Connections from these
# IPs are allowed to override their IP addresses and ports (for logging and
# for authentication checks). disable_plaintext_auth is also ignored for
# these networks. Typically you'd specify your IMAP proxy servers here.
#login_trusted_networks =

# Sepace separated list of login access check sockets (e.g. tcpwrap)
#login_access_sockets = 

# Show more verbose process titles (in ps). Currently shows user name and
# IP address. Useful for seeing who are actually using the IMAP processes
# (eg. shared mailboxes or if same uid is used for multiple accounts).
#verbose_proctitle = no

# Should all processes be killed when Dovecot master process shuts down.
# Setting this to "no" means that Dovecot can be upgraded without
# forcing existing client connections to close (although that could also be
# a problem if the upgrade is e.g. because of a security fix).
#shutdown_clients = yes

# If non-zero, run mail commands via this many connections to doveadm server,
# instead of running them directly in the same process.
#doveadm_worker_count = 0
# UNIX socket or host:port used for connecting to doveadm server
#doveadm_socket_path = doveadm-server

# Space separated list of environment variables that are preserved on Dovecot
# startup and passed down to all of its child processes. You can also give
# key=value pairs to always set specific settings.
#import_environment = TZ

##
## Dictionary server settings
##

# Dictionary can be used to store key=value lists. This is used by several
# plugins. The dictionary can be accessed either directly or though a
# dictionary server. The following dict block maps dictionary names to URIs
# when the server is used. These can then be referenced using URIs in format
# "proxy::<name>".

dict {
  #quota = mysql:/etc/dovecot/dovecot-dict-sql.conf.ext
  #expire = sqlite:/etc/dovecot/dovecot-dict-sql.conf.ext
}

# Most of the actual configuration gets included below. The filenames are
# first sorted by their ASCII value and parsed in that order. The 00-prefixes
# in filenames are intended to make it easier to understand the ordering.
!include conf.d/*.conf

# A config file can also tried to be included without giving an error if
# it's not found:
!include_try local.conf

mail_location = maildir:~/Maildir  
listen = *  
disable_plaintext_auth = no  
ssl = required  
ssl_cert_file = /etc/ssl/certs/dovecot.pem  
ssl_key_file = /etc/ssl/private/dovecot.pem  

auth default {  

  socket listen {  
    client {  
      path = /var/spool/postfix/private/auth  
      mode = 0660  
      user = postfix  
      group = postfix  
    }  
  }  

} 

```
##  outlook客户端无法登陆解决配置 
![](/data/dokuwiki/linux/pasted/20150601-211053.png)![](/data/dokuwiki/linux/pasted/20150601-211145.png)![](/data/dokuwiki/linux/pasted/20150601-211225.png)
##  手机客户端无法登陆解决配置 
与outlook配置参数差不多。` 只是用户名的选项不能填用户名@xby1993.net而是应该只填写用户名即可 `。
参考:
http://blog.csdn.net/basicthinker/article/details/6167606
http://www.cnblogs.com/dudu/archive/2012/12/12/linux-postfix-mailserver.html
http://www.oschina.net/question/67067_112155?sort=time