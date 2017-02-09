title: flask-uploads 

#  Flask-Uploads文件上传插件 
官网：http://pythonhosted.org/Flask-Uploads/

文件上传方式：
1、直接通过Flask-WTF文件上传功能。
2、Flask-Uploads
两种方式都可以，而且还可以结合一起使用。

Flask-Uploads提出了一个uploadsets概念，每个uploadset对应一个上传目录地址，可以方便地将不同资源放到不同地址中。
sets的类型:FILES,PHOTOS, ATTACHMENTS等。下面对于不同的集，你可以配置不同，比如用PHOTOS替换FILES
UPLOADED_FILES_DEST
This indicates the directory uploaded files will be saved to.
UPLOADED_FILES_URL
If you have a server set up to serve the files in this set, this should be the URL they are publicly accessible from. Include the trailing slash.
UPLOADED_FILES_ALLOW
This lets you allow file extensions not allowed by the upload set in the code.
UPLOADED_FILES_DENY
This lets you deny file extensions allowed by the upload set in the code.

```

#flaskext.uploads.UploadSet(name='files', extensions=('txt', 'rtf', 'odf', 'ods', 'gnumeric', 'abw', 'doc', 'docx', 'xls', 'xlsx', 'jpg', 'jpe', 'jpeg', 'png', 'gif', 'svg', 'bmp', 'csv', 'ini', 'json', 'plist', 'xml', 'yaml', 'yml'), default_dest=None)
photos = UploadSet('photos', IMAGES)
@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'POST' and 'photo' in request.files:
        filename = photos.save(request.files['photo'])
        rec = Photo(filename=filename, user=g.user.id)
        rec.store()
        flash("Photo saved.")
        return redirect(url_for('show', id=rec.id))
    return render_template('upload.html')

@app.route('/photo/<id>')
def show(id):
    photo = Photo.load(id)
    if photo is None:
        abort(404)
    url = photos.url(photo.filename)
    return render_template('show.html', url=url, photo=photo)

```
应用配置与绑定：
photos = UploadSet('photos', IMAGES)
configure_uploads(app, (photos,))

你也可以动态指定目录：
media = UploadSet('media', default_dest=lambda app: app.instance_root)

上传大小限制
```

patch_request_class(app)        # 16 megabytes
patch_request_class(app, 32 * 1024 * 1024)                                # 32 megabytes

```
  
##  示例 
涉及的flask扩展
flask-uploads	flask的一个文件上传扩展, 提供了UploadSet这个概念
flask-wtf(中文)	很强大的表单的扩展
flask-bootstrap	bootstrap的flask扩展, 结合模版使用, 此处用到quick_form功能
```

from flask import Flask, render_template
from flask_uploads import UploadSet, IMAGES, configure_uploads
from flask_wtf import Form
from wtforms import SubmitField
from flask_wtf.file import FileField, FileAllowed, FileRequired
from flask_bootstrap import Bootstrap

app = Flask(__name__)

# 新建一个set用于设置文件类型、过滤等
set_mypic = UploadSet('mypic')  # mypic

# 用于wtf.quick_form()模版渲染
bootstrap = Bootstrap(app)

# mypic 的存储位置,
# UPLOADED_xxxxx_DEST, xxxxx部分就是定义的set的名称, mypi, 下同
app.config['UPLOADED_MYPIC_DEST'] = './static/img'

# mypic 允许存储的类型, IMAGES为预设的 tuple('jpg jpe jpeg png gif svg bmp'.split())
app.config['UPLOADED_MYPIC_ALLOW'] = IMAGES

# 把刚刚app设置的config注册到set_mypic
configure_uploads(app, set_mypic)

app.config['SECRET_KEY'] = 'xxxxx'

# 此处WTF的SCRF密码默认为和flask的SECRET_KEY一样
# app.config['WTF_CSRF_SECRET_KEY'] = 'wtf csrf secret key'


class UploadForm(Form):
    '''
        一个简单的上传表单
    '''

    # 文件field设置为‘必须的’，过滤规则设置为‘set_mypic’
    upload = FileField('image', validators=[
                       FileRequired(), FileAllowed(set_mypic, 'you can upload images only!')])
    submit = SubmitField('ok')


@app.route('/', methods=('GET', 'POST'))
def index():
    form = UploadForm()
    url = None
    if form.validate_on_submit():
        filename = form.upload.data.filename
        url = set_mypic.save(form.upload.data, name=filename)
    return render_template('index.html', form=form, url=url)


if __name__ == '__main__':
    app.run(debug=True)


```
 html文件:
```

1 {% import "bootstrap/wtf.html" as wtf %}
2 
3 {% block page_content %}
4     <h4>uploaded: {% if url %} ![](/data/dokuwikiurl){% endif %}</h4>
5     ![](/data/dokuwiki wtf.quick_form(form, enctype="multipart/form-data") ) 
6 {% endblock page_content %}

```

参考：http://www.cnblogs.com/himir/p/5940705.html