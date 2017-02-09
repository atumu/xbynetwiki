title: js哈希与密码加密 

#  js哈希与密码加密 
为了避免明文密码传输，我们需要在客户端通过js对密码进行加密后传输。
##  jsSHA开源库 
采用的开源加密库:
jsSHA:https://github.com/Caligatio/jsSHA
demo:http://caligatio.github.io/jsSHA/
```

<script type="text/javascript" src="src/sha.js"></script>
var shaObj = new jsSHA("SHA-512", "TEXT",{numRounds:1,encoding:"UTF8"}); // The hash type can be one of SHA-1, SHA-224, SHA-256, SHA-384, or SHA-512. The input type can be one of HEX, TEXT, B64, or BYTES.第三个参数是一个option对象，numRounds默认为1，encoding默认为UTF8,有效参数有Valid options are "UTF8", "UTF16BE", and "UTF16LE", it defaults to "UTF8".
shaObj.update("This is a test");
var hash = shaObj.getHash("HEX");//parameter (B64, HEX, or BYTES).

``` 

参考：http://stackoverflow.com/questions/9544159/jquery-jssha-and-sha512-how-to-call-function

另外一个开源库：
##  JavaScript MD5 
JavaScript MD5：http://pajhome.org.uk/crypt/md5/index.html
```

<script type="text/javascript" src="md5-min.js"></script>
<script type="text/javascript">
    hash = hex_md5("string");
    hmac = hex_hmac_md5("key", "data");
</script>

```
