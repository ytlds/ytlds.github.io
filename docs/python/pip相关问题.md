# pip命令安装第三方包出现retrying且ssl error问题汇总

今天pip包时一直retrying且报ssl error的错误，我弄了一上午才好，网上有很多解决方案，但是没有pip安装失败的汇总情况，如有同错，请对比以下情况，希望能解决你的问题，也烦请对不足之处指出。

一、 错误：

```
WARNING: pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
ERROR: Could not find a version that satisfies the requirement pip (from versions: none)
ERROR: No matching distribution found for pip
```

CouldnotfetchURLhttps://pypi.python.org/simple/pytest-xdist/: There wasaproblem confirmingthessl certificate: [SSL: TLSV1_ALERT_PROTOCOL_VERSION] tlsv1 alert protocolversion(_ssl.c:590)

```bash
curl https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```



二、错误：

Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, u'[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:661)'),)'

解决方案：

1.mac或者linux操作系统：

修改 ~/.pip/pip.conf (没有就创建一个)， 更换 index-url

```
[global]
index-url = http://pypi.douban.com/simple
```

附：镜像源

阿里云 http://mirrors.aliyun.com/pypi/simple/

中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/

豆瓣 http://pypi.douban.com/simple

Python官方 https://pypi.python.org/simple/

v2ex http://pypi.v2ex.com/simple/

中国科学院 http://pypi.mirrors.opencas.cn/simple/

清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/

三、错误：Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:833)'),)) - skipping

原因：在最新的 pip 版本(>=7)中，使用镜像源时，会提示源地址不受信任或不安全

解决方案1：

pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org 包名

解决方案2：(推荐)在第二个解决方案中：添加一项配置

[install]

trusted-host=http://pypi.douban.com/simple



作者：阿萱555
链接：https://www.jianshu.com/p/6f6c640b0371
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。