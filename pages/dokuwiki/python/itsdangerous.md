title: itsdangerous 

#  itsdangerous生成并核对加密安全令牌 
官网：https://pythonhosted.org/itsdangerous/
中文：http://itsdangerous.readthedocs.io/en/latest/
当前版本：0.24

pip install itsdangerous

有时候你想向不可信的环境发送一些数据，但如何安全完成这个任务呢？解决的方法就是签名。使用只有你自己知道的密钥，来加密签名你的数据，并把加密后的数据发给别人。当你取回数据时，你就可以确保没人篡改过这份数据。比如发送重置密码的URL链接时。

诚然，接收者可以破译内容，来看看你的包裹里有什么，**但他们没办法修改你的内容**，除非他们也有你的密钥。所以只要你保管好你的密钥，并且密钥足够复杂，一切就OK了。
itsdangerous内部默认使用了HMAC和SHA1来签名。它也支持JSON Web 签名 (JWS)。

适用案例
  * **在取消订阅某个通讯时，你可以在URL里序列化并且签名一个用户的ID。**（这种情况下你不需要生成一个一次性的token并把它们存到数据库中。）**在任何的激活账户的链接或类似的情形下，同样适用。**
  * **被签名的对象可以被存入cookie中或其他不可信来源，这意味着你不需要在服务端保存session**，这样可以降低数据库读取的次数。
  * 通常签名后的信息可以安全地往返与服务端与客户端之间，**这个特性可以用于将服务端的状态传递到客户端再传递回来。**

##  签名接口 
最基本的接口是Signer。 Signer 类可以用来将一个签名附加到指定的字符串上：
```

>>> from itsdangerous import Signer
>>> s = Signer('secret-key')
>>> s.sign('my string')
'my string.wh6tMHxLgJqB6oY1uT73iMlyrOA'

```
签名会被加在字符串尾部，中间由句号 (.)分隔。验证字符串，使用 unsign() 方法：
```

>>> s.unsign('my string.wh6tMHxLgJqB6oY1uT73iMlyrOA')
'my string'

```

**如果被签名的是一个unicode字符串，那么它将隐式地被转换成utf-8。然而，在反签名时，你没法知道它原来是unicode还是字节串。**
如果反签名失败了，将得到一个异常：
```

>>> s.unsign('my string.wh6tMHxLgJqB6oY1uT73iMlyrOX')
Traceback (most recent call last):
  ...
itsdangerous.BadSignature: Signature "wh6tMHxLgJqB6oY1uT73iMlyrOX" does not match

```

##  使用时间戳签名 
如果你想要**可以过期的签名**，可以使用 **TimestampSigner** 类，它会加入时间戳信息并签名。在反签名时，你可以验证时间戳有没有过期：
```

>>> from itsdangerous import TimestampSigner
>>> s = TimestampSigner('secret-key')
>>> string = s.sign('foo')
>>> s.unsign(string, max_age=5)
Traceback (most recent call last):
  ...
itsdangerous.SignatureExpired: Signature age 15 > 5 seconds

```
##  序列化 
因为字符串难以处理，本模块也提供了一个与json或pickle类似的序列化接口。（**它内部默认使用simplejson**，但是可以通过子类进行修改）
```

>>> from itsdangerous import Serializer
>>> s = Serializer('secret-key')
>>> s.dumps([1, 2, 3, 4])
'[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo'
它当然也可以加载数据：
>>> s.loads('[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo')
[1, 2, 3, 4]

```
如果你想要带一个时间戳，你可以用 **TimedSerializer** 类。

##  URL安全序列化 
如果能够向只有字符受限的环境中传递可信的字符串的话，将十分有用。因此，itsdangerous也提供了一个**URL安全序列化工具：**
```

>>> from itsdangerous import URLSafeSerializer
>>> s = URLSafeSerializer('secret-key')
>>> s.dumps([1, 2, 3, 4])
'WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo'
>>> s.loads('WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo')
[1, 2, 3, 4]

```

##  JSON Web 签名 
从“itsdangerous” 0.18版本开始，也支持了JSON Web签名。它们的工作方式与原有的URL安全序列化器差不多，但是会根据当前JSON Web签名（JWS）草案（10） [draft-ietf-jose-json-web-signature] 来生成header。
```

>>> from itsdangerous import JSONWebSignatureSerializer
>>> s = JSONWebSignatureSerializer('secret-key')
>>> s.dumps({'x': 42})
'eyJhbGciOiJIUzI1NiJ9.eyJ4Ijo0Mn0.ZdTn1YyGz9Yx5B5wNpWRL221G1WpVE5fPCPKNuc6UAo'

```
在将值加载回来时，默认会像其他序列化器一样，不会返回header。

如果要带有时间撮，可以使用**TimedJSONWebSignatureSerializer**
```

s = TimedJSONWebSignatureSerializer('SECRET_KEY', 3600) #expiration=3600s
token= s.dumps({'confirm': self.id})
data = s.loads(token)

```
##  盐 
**所有的类都接受一个盐的参数。**
这名字可能会误导你，因为通常你会认为，密码学中的盐会是一个和被签名的字符串储存在一起的东西，用来防止彩虹表查找。这种盐是公开的。
**itsdangerous中的盐，是为了一个截然不同的目的而产生的。你可以将它视为成命名空间。就算你泄露了它，也不是很严重的问题，因为没有密钥的话，它对攻击者没什么帮助。**

假设你想签名两个链接。你的系统**有个激活链接**，用来激活一个用户账户，**并且你有一个升级链接**，可以让一个用户账户升级为付费用户，这两个链接使用email发送。在这两种情况下，如果你签名的都是用户ID，那么该用户可以在激活账户和升级账户时，复用URL的可变部分。现在你可以在你签名的地方加上更多信息（如升级或激活的意图），但是你也可以用不同的盐：
```

>>> s1 = URLSafeSerializer('secret-key', salt='activate-salt')
>>> s1.dumps(42)
'NDI.kubVFOOugP5PAIfEqLJbXQbfTxs'
>>> s2 = URLSafeSerializer('secret-key', salt='upgrade-salt')
>>> s2.dumps(42)
'NDI.7lx-N1P-z2veJ7nT1_2bnTkjGTE'
>>> s2.loads(s1.dumps(42))
Traceback (most recent call last):
  ...
itsdangerous.BadSignature: Signature "kubVFOOugP5PAIfEqLJbXQbfTxs" does not match

```
**只有使用相同盐的序列化器才能成功把值加载出来：**
```

>>> s2.loads(s2.dumps(42))
42

```
##  对失败的响应 
允许你在签名检查失败时，检查你的数据。
这里必须极其小心，因为这个时候，你知道有某人修改了你的数据。

示例用法:
```

from itsdangerous import URLSafeSerializer, BadSignature, BadData
s = URLSafeSerializer('secret-key')
decoded_payload = None
try:
    decoded_payload = s.loads(data)
    # This payload is decoded and safe
except BadSignature, e:
    encoded_payload = e.payload
    if encoded_payload is not None:
        try:
            decoded_payload = s.load_payload(encoded_payload)
        except BadData:
            pass
        # 这里的数据可以解码出来，但不是安全的，因为有某人改动了签名。
        # 解码步骤(load_payload)是显式的，因为将数据反序列化可能是不安全的
        #（请设想被解码的不是json,而是pickle）

```
**如果你不想检查到底是哪里出错了，你也可以使用不安全的加载方式:**
```

from itsdangerous import URLSafeSerializer
s = URLSafeSerializer('secret-key')
sig_okay, payload = s.loads_unsafe(data)

```
返回的元组中第一项是一个布尔值，表明了签名是否是正确的。

##  实例：电子邮件确认链接加密 
对于某些特定类型的程序,有必要确认注册时用户提供的信息是否正确。常见要求是能通过提供的电子邮件地址与用户取得联系。
为验证电子邮件地址,用户注册后,程序会立即发送一封确认邮件。新账户先被标记成待确认状态,用户按照邮件中的说明操作后,才能证明自己可以被联系上。账户确认过程中**,往往会要求用户点击一个包含确认令牌的特殊 URL 链接。**

**使用 itsdangerous 生成确认令牌** 
一般确认邮件的链接为：http://www.example.com/auth/confirm/<id>其中 id 是数据库分配给用户的数字 id.用户点击链接后,处理这个路由的视图函数就将收到的用户 id 作为参数进行确认,然后将用户状态更新为已确认。**但这种实现方式显然不是很安全.解决方法是把 URL 中的 id 换成将相同信息安全加密后得到的令牌。**
Flask 使用加密的签名 cookie 保护用户会话,防止被篡改。这种安全的 cookie 使用 itsdangerous 包签名。同样的方法也可用于确认令牌上。
```

from itsdangerous import TimedJSONWebSignatureSerializer as Serializer
from flask import current_app
from . import db
class User(UserMixin, db.Model):
# ...
    confirmed = db.Column(db.Boolean, default=False)
    def generate_confirmation_token(self, expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'], expiration)
        return s.dumps({'confirm': self.id})
    def confirm(self, token):
        s = Serializer(current_app.config['SECRET_KEY'])
        try:
            data = s.loads(token)
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.confirmed = True
        db.session.add(self)
        return True

```

参考：http://blog.csdn.net/sun_dragon/article/details/51735944