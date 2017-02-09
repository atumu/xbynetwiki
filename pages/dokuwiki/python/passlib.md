title: passlib 

#  Passlib加密库 
官网：https://pythonhosted.org/passlib/
当前版本:1.6.5

Passlib is a password hashing library for Python 2 & 3,cross-platform,支持30多种加密算法


pip install passlib
如果需要支持BCrypt，那么你需要安装以下三个库当中的一个bcrypt, py-bcrypt, or bcryptor

例如：
```

>>> # import the hash algorithm
>>> from passlib.hash import sha256_crypt

>>> # generate new salt, and hash a password
>>> hash = sha256_crypt.encrypt("toomanysecrets")
>>> hash
'$5$rounds=80000$zvpXD3gCkrt7tw.1$QqeTSolNHEfgryc5oMgiq1o8qCEAcmye3FoMSuvgToC'

>>> # verifying the password
>>> sha256_crypt.verify("toomanysecrets", hash)
True
>>> sha256_crypt.verify("joshua", hash)
False

```


```

>>> # import the context under an app-specific name (so it can easily be replaced later)
>>> from passlib.apps import custom_app_context as pwd_context

>>> # encrypting a password...
>>> hash = pwd_context.encrypt("somepass")

>>> # verifying a password...
>>> ok = pwd_context.verify("somepass", hash)

>>> # [optional] encrypting a password for an admin account...
>>> #            the custom_app_context is preconfigured so that
>>> #            if the category is set to "admin" instead of None,
>>> #            it uses a stronger setting of 80000 rounds:
>>> hash = pwd_context.encrypt("somepass", category="admin")

```

##  额外：Werkzeug 中的 security 模块 
其实Werkzeug 中的 security 模块能够很方便地实现密码散列值的计算。
• **generate_password_hash**(password, method= pbkdf2:sha1 ,salt_length=8) :这个函数将原始密码作为输入,以字符串形式输出密码的散列值,输出的值可保存在用户数据库中。method 和 salt_length 的默认值就能满足大多数需求。 
• **check_password_hash**(hash, password) :这个函数的参数是从数据库中取回的密码散列值和用户输入的密码。返回值为 True 表明密码正确。

from werkzeug.security import generate_password_hash, check_password_hash

##  常用算法接口 
https://pythonhosted.org/passlib/lib/passlib.hash.html#module-passlib.hash
结合多种算法：
  * passlib.hash.md5_crypt - MD5 Crypt
  * passlib.hash.bcrypt - BCrypt，需要安装bcrypt库
  * passlib.hash.pbkdf2_sha512
  * passlib.hash.sha1_crypt - SHA-1 Crypt
  * passlib.hash.sun_md5_crypt - Sun MD5 Crypt
  * passlib.hash.sha256_crypt - SHA-256 Crypt
  * passlib.hash.sha512_crypt - SHA-512 Crypt
  * passlib.hash.des_crypt - DES Crypt

**不推荐**仅单独使用一种算法，例如如下：
  *  passlib.hash.hex_md5
  *  passlib.hash.hex_sha1
  *  passlib.hash.hex_sha256
  *  passlib.hash.hex_sha512

##  配置使用CryptContext 
passlib.context.CryptContext被设计用来一次性处理很多次hash计算。如果没有这种情形，可以考虑不使用它。
```

#
# import the CryptContext class, used to handle all hashing...
#
from passlib.context import CryptContext

#
# create a single global instance for your app...
#
pwd_context = CryptContext(
    # replace this list with the hash(es) you wish to support.
    # this example sets pbkdf2_sha256 as the default,
    # with support for legacy des_crypt hashes.
    schemes=["pbkdf2_sha256", "des_crypt" ],
    default="pbkdf2_sha256",

    # vary rounds parameter randomly when creating new hashes...
    all__vary_rounds = 0.1,

    # set the number of rounds that should be used...
    # (appropriate values may vary for different schemes,
    # and the amount of time you wish it to take)
    pbkdf2_sha256__default_rounds = 8000,
    )

```
To start using your CryptContext, import the context you created wherever it’s needed:
```

>>>
>>> # import context from where you defined it...
>>> from myapp.model.security import pwd_context

>>> # encrypting a password...
>>> hash = pwd_context.encrypt("somepass")
>>> hash
'$pbkdf2-sha256$7252$qKFNyMYTmgQDCFDS.jRJDQ$sms3/EWbs4/3k3aOoid5azwq3HPZKVpUUrAsCfjrN6M'

>>> # verifying a password...
>>> pwd_context.verify("somepass", hash)
True
>>> pwd_context.verify("wrongpass", hash)
False

```
##  MD5 
```

>>> # import the desired hash
>>> from passlib.hash import md5_crypt

>>> # hash the password - encrypt() takes care of salt generation, unicode encoding, etc.
>>> hash = md5_crypt.encrypt("password")
>>> hash
'$1$IU54yC7Y$nI1wF8ltcRvaRHwMIjiJq1'

>>> # verify a password against an existing hash:
>>> md5_crypt.verify("password", hash)
True

```
