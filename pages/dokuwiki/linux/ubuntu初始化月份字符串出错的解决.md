title: ubuntu初始化月份字符串出错的解决 

#  ubuntu ls初始化月份字符串出错的解决 

网上作法：

1 vi /var/lib/locales/supported.d/local

在其中增加：

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8

然后：

locale-gen

然后，再修改：

vi /etc/default/locale

这里应该是很重要的一步：

LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh"
<wrap em>LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"</wrap>
LC_MONETARY="zh_CN"
LC_PAPER="zh_CN"
LC_NAME="zh_CN"
LC_ADDRESS="zh_CN"
LC_TELEPHONE="zh_CN"
LC_MEASUREMENT="zh_CN"
LC_IDENTIFICATION="zh_CN"

` 保存，重启。 `

ls -l显示，正常了。