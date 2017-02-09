title: ipy 

#  处理IP地址IPy 
github:https://github.com/autocracy/python-ipy
IPy模块用于完成高效的IP规划工作。
pip install ipy

```

from IPy import IP

ip = IP('192.168.1.0/24')
ip.net()  # 取IP地址
ip.netmask()  # 取值子网掩码
ip.iptype()  # 查询为公网IP还是私网IP，公网为:PUBLIC,私网为：PRIVATE
if ip.len() > 1:  # 如果len出来的数字大于1，那么就是一个网段
    print('网络地址: %s' % ip.net())
    print('子网掩码: %s' % ip.netmask())
    print('网络广播地址: %s' % ip.reverseNames()[0])
    print('网络子网数: %s' % ip.len())
else:  ###否则就是一个地址
    print('IP反向解析: %s' % ip.reverseNames()[0])
    print('十六进制地址: %s' % ip.strHex())
    print('二进制地址: %s' % ip.strBin())
    print('地址类型: %s' % ip.iptype())


```
    

处理dns：pip install dnspython