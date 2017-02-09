createAt:2017-01-15 14:32:00
author:xbynet
modifyAt:2017-01-15 14:32:00
location:Python核心编程3e学习笔记之因特网客户端编程
title:Python核心编程3e学习笔记之因特网客户端编程

# FTP文件传输协议
HTTP,FTP,scp,rsync等都属于文件传输协议。在底层FTP只是用TCP，而不使用UDP。另外客户端与服务端都使用两个套接字来通信:一个是控制和命令端口(21),另一个是数据端口(可能是20，也可能随机)。FTP有两种模式：主动和被动。只有在主动模式下服务器才使用20数据端口。在被动模式下该端口是随机的。
Python中的ftplib模块
```
#!/usr/bin/env python

import ftplib
import os
import socket

HOST = 'ftp.mozilla.org'
DIRN = 'pub/mozilla.org/webtools'
FILE = 'bugzilla-LATEST.tar.gz'

def main():
    try:
        f = ftplib.FTP(HOST)
    except (socket.error, socket.gaierror) as e:
        print('ERROR: cannot reach "%s"' % HOST)
        return
    print('*** Connected to host "%s"' % HOST)

    try:
        f.login() #可传用户名和密码
    except ftplib.error_perm:
        print('ERROR: cannot login anonymously')
        f.quit()
        return
    print('*** Logged in as "anonymous"')

    try:
        f.cwd(DIRN)
    except ftplib.error_perm: #没有权限
        print('ERROR: cannot CD to "%s" folder' % DIRN)
        f.quit()
        return
    print('*** Changed to "%s" folder' % DIRN)

    try:
        f.retrbinary('RETR %s' % FILE, 
            open(FILE, 'wb').write)   
    except ftplib.error_perm:
        print('ERROR: cannot read file "%s"' % FILE)
        if os.path.exists(FILE):
            os.unlink(FILE)
    else:
        print('*** Downloaded "%s" to CWD' % FILE)
    f.quit()

if __name__ == '__main__':
    main()

```

# 电子邮件
MTA:(Message transfer agent)消息传输代理。 MTS(消息传输系统)。SMTP(Simple Mail Transfer Protocol)简单邮件传输协议，它需要管理一个邮件队列。
为了发送电子邮件，邮件客户端必须要连接到一个MTA，MTA靠某种协议(如SMTP)进行通信。MTA之间通过MTS互相通信。
一些MTA实现：
开源的MTA: Sendmail、Postfix、Exim、qmail
商业的MTA： Exchange

Python和SMTP：发送电子邮件
smtplib模块 SMTP默认25端口，SMTP_SSL默认465端口

POP3和IMAP：接收电子邮件
POP3(Post Office Protocol)、IMAP(Internet Message Access Protocol)
模块poplib、imaplib

```
#!/usr/bin/env python
from smtplib import SMTP
from poplib import POP3
from time import sleep

SMTPSVR = 'smtp.python.is.cool'
POP3SVR = 'pop.python.is.cool'

who = 'wesley@python.is.cool'
body = '''\
From: %(who)s
To: %(who)s
Subject: test msg

Hello World!
''' % {'who': who}

sendSvr = SMTP(SMTPSVR)
errs = sendSvr.sendmail(who, [who], origMsg)
sendSvr.quit()
assert len(errs) == 0, errs
sleep(10)    # wait for mail to be delivered

recvSvr = POP3(POP3SVR)
recvSvr.user('wesley')
recvSvr.pass_('youllNeverGuess')
rsp, msg, siz = recvSvr.retr(recvSvr.stat()[0])
# strip headers and compare to orig msg
sep = msg.index('')
recvBody = msg[sep+1:]
assert origBody == recvBody # assert identical
```

## 实战发送电子邮件
其余相关模块：email、smtpd、base64、mhlib、mailbox、mimetypes、binascii
```
#!/usr/bin/env python
'email-examples.py - demo creation of email messages'

from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from smtplib import SMTP

# multipart alternative: text and html
def make_mpa_msg():
    email = MIMEMultipart('alternative')
    text = MIMEText('Hello World!\r\n', 'plain')
    email.attach(text)
    html = MIMEText(
        '<html><body><h4>Hello World!</h4>'
        '</body></html>', 'html')
    email.attach(html)
    return email

# multipart; images
def make_img_msg(fn):
    f = open(fn, 'r')
    data = f.read()
    f.close()
    email = MIMEImage(data, name=fn)
    email.add_header('Content-Disposition',
        'attachment; filename="%s"' % fn)
    return email

def sendMsg(fr, to, msg):
    s = SMTP('localhost')
    errs = s.sendmail(fr, to, msg)
    s.quit()

if __name__ == '__main__':
    print 'sending multipart alternative msg'
    msg = make_mpa_msg()
    msg['From'] = SENDER
    msg['To'] = ', '.join(RECIPS)
    msg['Subject'] = 'multipart alternative test'
    sendMsg(SENDER, RECIPS, msg.as_string())

    print 'sending image msg'
    msg = make_img_msg(SOME_IMG_FILE)
    msg['From'] = SENDER
    msg['To'] = ', '.join(RECIPS)
    msg['Subject'] = 'image file test'
    sendMsg(SENDER, RECIPS, msg.as_string())

```