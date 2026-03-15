# web应用安全测试指南阅读笔记

## 1. 信息泄露

### 1.1 robots.txt信息泄露

robots.txt文件太详细会泄露网站指纹，系统框架，敏感目录。**可以手动在域名后面加/robots.txt，也可以使用dirsearch工具枚举目录发现。**

命令：

``` bash
python  dirsearch.py -u http://example.com
```

