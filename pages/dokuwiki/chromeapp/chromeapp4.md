title: chromeapp4 

#  Chrome扩展与应用开发之Chrome应用基础 
Chrome将其平台上的程序分为扩展与应用，并且使用了同样的文件结构
总的来说，扩展与浏览器结合得更紧密些，更加强调扩展浏览器功能。而应用无法像扩展一样轻易获取用户在浏览器中浏览的内容并进行更改，实际上应用有更加严格的权限限制。所以应用更强调是一个独立的与Chrome浏览器关联不大的程序，此时你可以把Chrome看成是一个开发环境，而不是一个浏览器。
除此之外，Chrome应用还分为**Hosted App（托管应用）和Packaged App（打包应用）**，这两者也是有明显区别的。相对而言，Hosted App更像是一个高级的书签，这种应用只提供一个图标和Manifest文件，在Manifest中声明了此应用的启动页面URL，以及包含的其他页面URL和这些页面请求的高级权限。比如下面的例子创建了一个启动页面为http://mail.google.com/mail/，包含mail.google.com/mail/和www.google.com/mail/且请求unlimitedStorage和notifications权限的应用。
```

{
    "name": "Google Mail",
    "description": "Read your gmail",
    "version": "1",
    "app": {
        "urls": [
            "*://mail.google.com/mail/",
            "*://www.google.com/mail/"
        ],
        "launch": {
            "web_url": "http://mail.google.com/mail/"
        }
    },
    "icons": {
        "128": "icon_128.png"
    },
    "permissions": [
        "unlimitedStorage",
        "notifications"
    ]
}

```
Packaged App，顾名思义，就是将所有文件打包在一起的应用，这类应用通常可以在离线时使用，因为它运行所需的全部文件都在本地。


----
