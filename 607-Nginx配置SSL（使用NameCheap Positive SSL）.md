因为Github学生套餐里面含有免费一年SSL，所以就直接拿来用了，这里记录一下在服务器上配置的过程。

## 系统环境

*   Linode Ubuntu 14.04 LTS x64

*   [LNMP1.1][1] (nginx 1.6.2)

## 准备工作

*   需要nginx已经编译SSL模块，如果你使用 [LNMP][2] 安装的环境，则默认已经编译上，无须此步
*   已经在 [NameCheap][3] 上购买了 SSL 产品

## 使用 OpenSSL 生成证书

任意选择一个目录，这里使用 `/usr/local/nginx/conf/` 目录

    cd /usr/local/nginx/conf/
    

1.  生成RSA密钥
    
        openssl genrsa -out doraemonext.com.pem 2048
        

2.  生成证书
    
        openssl req -new -key doraemonext.com.pem -out doraemonext.com.csr
        
    
    中间会让你输入各种信息，如下：
    
        You are about to be asked to enter information that will be incorporated
        into your certificate request.
        What you are about to enter is what is called a Distinguished Name or a DN.
        There are quite a few fields but you can leave some blank
        For some fields there will be a default value,
        If you enter '.', the field will be left blank.
        -----
        Country Name (2 letter code) [AU]:CN  // 国家
        State or Province Name (full name) [Some-State]:Hubei  // 省份
        Locality Name (eg, city) []:Wuhan  // 城市
        Organization Name (eg, company) [Internet Widgits Pty Ltd]:Whu University  // 组织名称或公司名称
        Organizational Unit Name (eg, section) []:  // 可不填
        Common Name (e.g. server FQDN or YOUR name) []:www.doraemonext.com  // 要配置SSL的域名，注意doraemonext.com和www.doraemonext.com不同
        Email Address []:doraemonext@gmail.com  // 电子邮件地址
        
        Please enter the following 'extra' attributes
        to be sent with your certificate request
        A challenge password []:  // 可不填
        An optional company name []:  // 可不填
        

3.  现在目录下生成了 `doraemonext.com.csr` 证书文件，打开该文件并复制全部内容，即从`-----BEGIN CERTIFICATE REQUEST-----`开头到`-----END CERTIFICATE REQUEST-----`结尾，复制下来后，用这个信息去 NameCheap 生成数字证书。
    
        root@Doraemonext-Production:/usr/local/nginx/conf# cat doraemonext.com.csr
        -----BEGIN CERTIFICATE REQUEST-----
        QQQC0DCCAbgCAQAwg22xCzAJBgNVBAYTAkNOMQ4wDAYDVQQIDAVIdWJlaTEOMAwG
        A1UEBwwFV3VoYW4xFzAVBgNVBAoMDldodSBV88l2ZXJzaXR5MRwwGgYDVQQDDBN3
        d3cuZG9yYWVtb2525HQuY29tMSQwIgYJKoZIhvcNAQkBFhVkb3JhZW1vbmV4dETT
        bWFpbC5KB20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDIjZCCqPsX
        GnPeX+MUPrlr/lPg5IuCEFjWfbn4xubkpsX5S3jTTvwIEaDggqsU8ohs6mOOjLPn
        u/IP9sS1QnV0e8IcqMW2VBfhMW15pBrzfikoTl3SmsXd2s4MQDNPWPxChdNOhnrA
        JrUYMg6RdJJWZ/8W7trUtTH8IpPIBrPtadnrDHU7lmHSTueooBEJbIm9GKVF1lkZ
        9xh3SbtNibgxmvvTT6OA6NS3hA5sov0P0pP0ZbxVbeAoYWGVXo5CJWB33gB994ak
        Orj3Q1ZulemWIo5SWvVe0XTh6HXjNEyxt8/GAHnfY9Dp4xNaz6aGcS2aay2WKfCM
        DZhTT2kaDVfBAgMBAAGgADANBgkqhkiG9w0BAQsFATOWWQEAbf/Jc2CLTEC3SIeI
        PdI/g5e4AMjbhEoGiG7eIWyRxEfLnd42k5YsdTMwPECmQOIFueUNd5xwluJrltjR
        he336BC7dayOS9NfX74ipjZklTatVU/i9/J4n26+s73UkwX7BvP+TJyASfH8LUF/
        BRUO2l/nVlE7K05TX6c37I92x4oe9hgfrsnPSNWDsMpXv3URMnYX27HrcApcFcyM
        0sO0YYzGT7wId5WVPfae6eQs9d+97RBSgM7jBtijJB4hfNI1fj/aTNZfybCCnctE
        EYvt0nAUfmsct+MXt6nN2XzwNhA+C6l3FUOPLNdsKqAEKwijHB9Pzp5yOSy9D/Dn
        hnwu99==
        -----END CERTIFICATE REQUEST-----
        

## 在 NameCheap 上生成数字证书

登录NameCheap后，首页会提示你当前有未激活的 SSL 产品，直接点上面的`SSL Certificates page`进去。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/1.jpg" alt="1" width="797" height="258" class="alignnone size-full wp-image-620" />][4]

点击`Activate Now`即可开始激活对应SSL产品。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/2.jpg" alt="2" width="728" height="207" class="alignnone size-full wp-image-626" />][5]

然后提交刚才由OpenSSL生成的证书，也就是把刚才复制过来的东西全部粘贴进去，如图，注意Web Server选`nginx`。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/3.jpg" alt="3" width="807" height="593" class="alignnone size-full wp-image-628" />][6]

Next后选择域名管理员的邮箱，注意一定要保证该邮箱可以接收到邮件。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/4.jpg" alt="4" width="645" height="470" class="alignnone size-full wp-image-632" />][7]

最后一步就是提交订单，注意核实一下信息。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/5.jpg" alt="5" width="795" height="522" class="alignnone size-full wp-image-634" />][8]

接着就会出现下图的页面，同时系统也会向你的域名管理员邮箱发送确认邮件，按照邮件提示确认证书信息，一般几分钟即可，如果长时间未收到直接联系在线客服即可，会有其他验证方式（如CNAME或Text File方式）。

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/6.jpg" alt="6" width="740" height="480" class="alignnone size-full wp-image-637" />][9]

## 将证书打包并上传

确认之后，会朝你的邮件地址发送一个证书压缩包，直接下载，里面会有四个文件，如下：

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/7.jpg" alt="7" width="376" height="108" class="alignnone size-full wp-image-639" />][10]

接着你需要将他们打包为一个文件，执行如下命令，注意替换相应文件名为你自己的：

    cat COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt >> www_doraemonext_com.csr
    

把这个 `www_doraemonext_com.csr` 上传到服务器上，然后将上面一开始生成的 `doraemonext.com.pem` 和这个 `www_doraemonext_com.csr` 放置到nginx的配置文件目录下，这里放置到 `/usr/local/nginx/conf/` 目录下。

## 修改nginx配置文件

打开对应的nginx配置文件，修改相应的 `server` 段如下（这里是我的nginx配置，你只需要修改加粗的部分即可）：

<pre>server
    {
        <strong>listen 443;</strong>
        server_name doraemonext.com www.doraemonext.com;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/blog;

        <strong>ssl on;
        ssl_certificate www_doraemonext_com.crt;
        ssl_certificate_key doraemonext.com.pem;</strong>

        include wordpress.conf;
        #error_page   404   /404.html;
        location ~ [^/]\.php(/|$)
            {
                # comment try_files $uri =404; to enable pathinfo
                try_files $uri =404;
                fastcgi_pass  unix:/tmp/php-cgi.sock;
                fastcgi_index index.php;
                <strong>fastcgi_param HTTPS on;</strong>
                include fastcgi.conf;
                #include pathinfo.conf;
            }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
            {
                expires      30d;
            }

        location ~ .*\.(js|css)?$
            {
                expires      12h;
            }

        access_log  /home/wwwlogs/blog.log.log  access;
    }
</pre>

最后执行一下测试，看看配置是否正确，然后重启nginx服务器即可。

    /usr/local/nginx/sbin/nginx -t
    kill -HUP `cat /usr/local/nginx/logs/nginx.pid`
    

## 测试结果

现在直接输入 `https://www.doraemonext.com` 测试即可，就像本站一样，如下图

[<img src="https://www.doraemonext.com/wp-content/uploads/2015/04/8.jpg" alt="8" width="303" height="394" class="alignnone size-full wp-image-644" />][11]

## 禁用 HTTP 访问（可选）

如果你希望全站都使用 https 进行访问，那就可以对所有 http 访问进行 301 重定向到对应的 https 地址，在你的 nginx 配置中添加类似下面的配置即可：

    server 
        {
            listen 80;
            server_name www.doraemonext.com doraemonext.com;
            return 301 https://www.doraemonext.com$request_uri;
        }
    

到这里整个配置过程就结束了，如果有任何问题，欢迎留言。

 [1]: http://lnmp.org/download.html
 [2]: http://lnmp.org
 [3]: http://www.namecheap.com
 [4]: https://www.doraemonext.com/wp-content/uploads/2015/04/1.jpg
 [5]: https://www.doraemonext.com/wp-content/uploads/2015/04/2.jpg
 [6]: https://www.doraemonext.com/wp-content/uploads/2015/04/3.jpg
 [7]: https://www.doraemonext.com/wp-content/uploads/2015/04/4.jpg
 [8]: https://www.doraemonext.com/wp-content/uploads/2015/04/5.jpg
 [9]: https://www.doraemonext.com/wp-content/uploads/2015/04/6.jpg
 [10]: https://www.doraemonext.com/wp-content/uploads/2015/04/7.jpg
 [11]: https://www.doraemonext.com/wp-content/uploads/2015/04/8.jpg