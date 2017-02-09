title: eclipse_idea搜索手动添加到本地maven库的jar 

#  eclipse,idea搜索手动添加到本地maven库的jar 
参考：http://blog.csdn.net/tower888/article/details/24331957
**eclipse:**
windows/show view/maven repositories/Local Repository 右击进行重新建索引
**idea:**
1. 安装第三方jar到我们的本地库中
进入cmd，F:\java_memcached-release_2.6.6>mvn install:install-file -Dfile=java_memcached-release_2.6.6.jar -DgroupId=com.dana -DartifactId=memcached -Dversion=2.6.6 -Dpackaging=jar -DgeneratePom=true
经过上面就万事大吉了么，我们就可以在Eclipse中直接添加这个jar了么？？非也，下面看看第2步：
2.**重建maven本地库（Repository）索引index**（ window->show View->other->Maven->Meven Repositorise点击  OK  然后在视图中 点击local Repositorise**右键Rubuild Index）**
其实这一步相当于让maven重新对本地的Responsitory中的jar建立索引，以让我们搜索包的时候可以搜索到我们最新安装的jar。
idea要想能从本地maven库搜索到手动添加的jar，可以settings/maven/repositories/点击对应URL的update按钮
![](/data/dokuwiki/tooluse/pasted/20150919-100530.png)