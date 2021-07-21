# 第四章 批量验证一个POC

上一章已经说创建了一个非常简单的poc，可以简单扫描一个网站的备份文件

接下来我们需要通过这个poc进行批量的验证，逐步推进我们的学习进度

本章我们逐渐完善这个脚本，并且让这个脚本可以进行批量的漏洞验证



``` python
# ch_04/backup_scan.py
from urllib import parse
import requests


def check(site):
    '''
    扫描备份文件的poc
    :param site: 扫描的站点
    :return:
    '''

    # 加载扫描的字典
    lists = []
    with open("dict.txt") as dicfile:  # 打开字典文件
        for line in dicfile:  # 读入文件
            line = line.strip("\r\n")  # 去掉行结束的换行符
            if not line:  # 如果是空行则跳过
                continue
            lists.append(line)  # 添加到字典列表
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
    with open("url.txt") as srcfile:  # 读取目标来源
        for site in srcfile:  # 逐行读取文件内容
            site = site.strip("\r\n")  # 去掉行结束的换行符
            if not site: # 如果是空行则跳过处理
                continue
            check(site)
```

我们对脚本进行了部分的完善

主要完善部分

- 将字典的来源设置为通过文件读取并且处理
- 将扫描的目标通过文件读取的方式作为数据源

## 当我们重复写了很多次

如果我们有很多的poc，都是如此的使用脚本，如此的编写，就会发现有很多的重复性的代码

代码中，懒即美德，当我们重复写了如此多的代码，我们就应该深入思考一下了
