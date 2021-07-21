# 第三章 开始我们的第一个POC

按 **第二章** 列的大纲，我们这一章要编写一个简单的POC

本来是打算先随便找个漏洞看一下，但是又感觉没有那种很大众又深入人心的漏洞，就决定先用备份文件的扫描先凑合一下

```python
# ch_03/backup_scan.py
from urllib import parse
import requests


def check(site):
    '''
    扫描备份文件的poc
    :param site: 扫描的站点
    :return:
    '''

    # 扫描的字典
    lists = [
        "index.php",
        "web.zip",
        "web.rar",
        "web.7z",
        "web.tar.gz",
        "www.rar",
        "htdocs.tar.gz",
        "wwwroot.zip",
        "site.zip",
        "blog.zip"
    ]
    for filename in lists:
        target = parse.urljoin(site, filename)  # 拼接网站和文件名称
        try:
            r = requests.head(target, timeout=3)  # 开始通过head方法请求拼接好的路径，并且设置超时时间为3秒
            length = int(r.headers.get('Content-Length', 0))  # 获取响应内容的长度，如果没有这个值，就当是0
            if r.status_code == 200 and length > 1024:  # 如果请求状态码是200，并且响应长度大于1kb，就认为是有备份文件下载
                print("存在文件下载漏洞:%s 文件尺寸:%d" % (target, length))
        except:
            pass


if __name__ == '__main__':
    check("https://www.baidu.com")
```

这是我们的第一个poc，对python不熟悉的同学课后麻烦自行复习python相关的基础知识

这个脚本的主要内容是

- 创建一个用于扫描的函数，并且传入参数作为扫描目标
- 创建一个列表，当作扫描的payload列表
- 通过循环的方式，将文件名称拼接到URL中
- 创建异常处理
- 通过HEAD方法对服务器进行请求，并且设置超时参数，上一条异常处理对应本条请求超时
- 获取响应内容的长度，如果服务器没有返回长度，就当长度为0
- 如果状态码是200，并且正文长度大于1kb，则认为是存在文件下载漏洞的



这是我们第一个漏洞验证脚本，当然这个脚本还非常的粗糙

比如我们应该从响应头中取更多的信息，判断如果是html应该算是误报情况

比如我们请求的过程中应该使用更高效率的方式，例如多线程

