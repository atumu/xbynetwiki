title: windows下openssl 

#  windows下openssl 
下载:http://slproweb.com/products/Win32OpenSSL.html
http://blog.csdn.net/bodybo/article/details/28238805
openssl中有如下后缀名的文件
  * .key格式：私有的密钥
  * .csr格式：证书签名请求（证书请求文件），含有公钥信息，certificate signing request的缩写
  * .crt格式：证书文件，certificate的缩写
  * .crl格式：证书吊销列表，Certificate Revocation List的缩写
  * .pem格式：用于导出，导入证书时候的证书的格式，有证书开头，结尾的格式
x509证书一般会用到三类文，key，csr，crt。
  * Key 是私用密钥openssl格，通常是rsa算法。
  * Csr 是证书请求文件，用于申请证书。在制作csr文件的时，必须使用自己的私钥来签署申，还可以设定一个密钥。
  * crt是CA认证后的证书文，（windows下面的，其实是crt），签署人用自己的key给你签署的凭证。 

再说步骤：
生成CA私钥--》生成CA证书请求--》自签名得到根证书（CA给自已颁发的证书）
生成私钥--》生成证书请求--》通过CA签名得到证书

1.key的生成 
openssl genrsa -des3 -out server.key 2048 
这样是生成rsa私钥，des3算法，openssl格式，2048位强度。server.key是密钥文件名。为了生成这样的密钥，需要一个至少四位的密码。可以通过以下方法生成没有密码的key:
openssl rsa -in server.key -out server.key 
server.key就是没有密码的版本了。 
2. 生成CA的crt
openssl req -new -x509 -key server.key -out ca.crt -days 3650 
生成的ca.crt文件是用来签署下面的server.csr文件。 
3. csr的生成方法
openssl req -new -key server.key -out server.csr 
需要依次输入国家，地区，组织，email。最重要的是有一个common name，可以写你的名字或者域名。如果为了https申请，这个必须和域名吻合，否则会引发浏览器警报。生成的csr文件交给CA签名后形成服务端自己的证书。 
4. crt生成方法
CSR文件必须有CA的签名才可形成证书，可将此文件发送到verisign等地方由它验证，要交一大笔钱，何不自己做CA呢。
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server.crt
输入key的密钥后，完成证书生成。-CA选项指明用于被签名的csr证书，-CAkey选项指明用于签名的密钥，-CAserial指明序列号文件，而-CAcreateserial指明文件不存在时自动生成。
最后生成了私用密钥：server.key和自己认证的SSL证书：server.crt

5、证书合并：
cat server.key server.crt > server.pem
windows：copy /b /y server.key+server.cer server.pem