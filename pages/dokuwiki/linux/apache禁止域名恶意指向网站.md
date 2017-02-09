title: apache禁止域名恶意指向网站 

#   apache禁止域名恶意指向网站 

```

<VirtualHost *:80>
  DocumentRoot "/opt/lampp/htdocs/"
  <Directory />
    Order Allow,Deny
    Deny from all
  </Directory>
#  ErrorLog "/alidata/log/httpd/error.log"
#  CustomLog "/alidata/log/httpd/info.log" common
</VirtualHost>

```