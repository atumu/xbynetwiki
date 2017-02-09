modifyAt:2016-11-26 16:48:27
location:dokuwiki\python\python学习笔记033
createAt:2016-11-26 16:48:27
author:xby@xbynet.net
title: Python学习笔记之Flask上传文件 

#  Python学习笔记之Flask上传文件 
文件上传的基本概念实际上非常简单， 他基本是这样工作的:
一个 <form > 标签被标记有 enctype=multipart/form-data ，并且在里面包含一个 < input type=file > 标签。
**服务端应用通过请求对象上的 files 字典访问文件。**
使用文件的 save() 方法将文件永久地保存在文件系统上的某处。
让我们建立一个非常基础的小应用，这个小应用可以上传文件到一个指定的文件夹里， 然后将这个文件显示给用户。让我们看看这个应用的基础代码:
```

import os
from flask import Flask, request, redirect, url_for
from werkzeug import secure_filename

UPLOAD_FOLDER = '/path/to/the/uploads'
ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

```
首先我们导入一些东西，大多数内容都是直接而容易的。werkzeug.secure_filename() 将会在稍后进行解释。 UPLOAD_FOLDER 是我们储存上传的文件的地方，而 ALLOWED_EXTENSIONS 则是允许的文件类型的集合。然后我们手动为应用添加一个的 URL 规则。我们通常很少这样做，但是为什么这里要如此呢？原因是我们希望实际部署的服务器 (或者我们的开发服务器）来为我们提供这些文件的访问服务，所以我们只需要一个规则用来生成指向这些文件的 URL 。
下一步，就是检查文件类型是否有效、上传通过检查的文件、以及将用户重定向到已经上传好的文件 URL 处的函数了:
```

def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in ALLOWED_EXTENSIONS
@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        file = request.files['file']
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
    return '''
    <!doctype html>
    <title>Upload new File</title>
    <h1>Upload new File</h1>
    <form action="" method=post enctype=multipart/form-data>
      <p><input type=file name=file>
         <input type=submit value=Upload>
    </form>
    '''

```
**那么 secure_filename() 函数具体做了那些事呢？现在的问题是，有一个信条叫做“永远别相信你用户的输入” ，这句话对于上传文件的文件名也是同样有效的。所有提交的表单数据都可以伪造，而文件名本身也可能是危险的。在摄氏只需记住: 在将文件保存在文件系统之前，要坚持使用这个函数来确保文件名是安全的。**
关于文件名安全的更多信息
您对 secure_filename() 的具体工作和您没使用它会造成的后果感兴趣？试想一个人可以发送下列信息作为 filename 给您的应用:
```

filename = "../../../../home/username/.bashrc"

```
假定 ../ 的数量是正确的，而您会将这串字符与 UPLOAD_FOLDER 所指定的路径相连接，那么这个用户就可能有能力修改服务器文件系统上的一个文件，而他不应该拥有这种权限。这么做需要一些关于此应用情况的技术知识，但是相信我， 骇客们都有足够的耐心 :)
现在我们来研究一下这个函数的功能:
```

>>> secure_filename('../../../../home/username/.bashrc')
'home_username_.bashrc'

```
现在还有最后一件事没有完成: 提供对已上传文件的访问服务。 在 Flask 0.5 以上的版本我们可以使用一个函数来实现此功能:
```

from flask import send_from_directory
@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)

```
##  改进上传功能 
Flask 到底是如何处理上传的呢？如果服务器相对较小，那么他会先将文件储存在网页服务器的内存当中。否则就将其写入一个临时未知(如函数`  tempfile.gettempdir() ` 返回的路径)。
**但是怎么指定一个文件大小的上限，当文件大于此限制，就放弃上传呢? 默认 Flask 会很欢乐地使用无限制的空间，但是您可以通过在配置中设定 ` MAX_CONTENT_LENGTH ` 键的值来限制它:**
```

from flask import Flask, Request
app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024

```
**上面的代码将会把上传文件限制为最大 16 MB 。 如果请求传输一个更大的文件， Flask 会抛出一个 RequestEntityTooLarge 异常。**
##  上传进度条 
现在有了一些性能更好、运行更可靠的解决方案。WEB 已经有了不少变化，现在您可以使用 HTML5、Java、Silverlight 或者 Flash 来实现客户端更好的上传体验。看一看下面列出的库的连接，可以找到一些很好的样例。
  * Plupload - HTML5, Java, Flash
  * SWFUpload - Flash
  * JumpLoader - Java
更简单解决方案
因为存在一个处理上传文件的范式，这个范式在大多数应用中机会不会有太大改变， 所以 **Flask 存在一个扩展名为 Flask-Uploads ，这个扩展实现了一整套成熟的文件上传架构。**它提供了包括文件类型白名单、黑名单等多种功能。
