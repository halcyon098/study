# bugku自动签到
## 参考文档：

  [dota_st师傅的博客](https://www.wlhhlc.top/posts/51649/)
		 [阿里云fc文档](https://help.aliyun.com/zh/fc/?spm=5176.137990.J_4VYgf18xNlTAyFFbOuOQe.112.67791608KxH0Dd)

## 准备工作

- 阿里云账号（支付宝账号，淘宝账号）

- GitHub账号(用于登录bugku)

  抓取自己GitHub的cookie备用(user_session的值)

![D:\Document\images202402141513576.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/D%3A%5CDocument%5Cimages202402141513576.png)


- bugku账号(绑定GitHub账号)

- 签到脚本(基本使用dota_st这位师傅博客中的脚本，进行了一些小修改)

  index.py 云函数服务默认调用的文件名，不要修改

  ```python
  import urllib3
  import re
  import requests
  from urllib import request as RR
  import json
  from retrying import retry
  from bs4 import BeautifulSoup
  urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
  #定义通用的请求头
  headers = {"user-agent": " Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36",
             "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"
             }
  
  #读取txt配置
  def load_txt():
      user_list = []
      server_list = []
      git_list = []
      f = open(r'配置.txt', 'r', encoding='utf-8')
      data = f.readlines()
      for i in range(len(data)):
          if data[i] != r'******author:dota_st******':
              user_line = data[i].split(r"user=")[1].split("#")[0]
              server_line = data[i].split(r"server_key=")[1].split("#")[0]
              git_line = data[i].split(r"git_cookie=")[1].split("#")[0]
              user_list.append(user_line)
              server_list.append(server_line)
              git_list.append(git_line)
          else:
              break
      return user_list, server_list, git_list
  
  
  #使用server酱发送消息
  def server_send(user_line, server_line, message):
      data = {'desp': message}
      server_key = server_line
      requests.post("https://sctapi.ftqq.com/"+server_key+".send?title=尊贵的"+user_line+"用户bugku自动签到脚本结果", data=data)
  
  #获取签到结果返回信息
  def login_result(user_line, server_line, bug_cookie):
      global headers
      headers['X-Requested-With'] = "XMLHttpRequest"
      headers['cookie'] = bug_cookie
      req = RR.Request(url='https://ctf.bugku.com/user/checkin', headers=headers)  # 这样就能把参数带过去了
      # 下面是获得响应
      with RR.urlopen(req) as f:
          Data = f.read()
          data = json.loads(Data)
          print(data['msg'])
          server_send(user_line, server_line, data['msg'])
  #登录判断
  def login_status(user_line, server_line,res):
      if ("登录成功" in res.text):
          print("cookie提取成功!")
          for i in res.headers['Set-Cookie'].split(','):
              if ('PHPSESSID' in i):
                  login_result(user_line, server_line, i.strip())
                  break
  
  #主函数
  @retry(stop_max_attempt_number=3)
  def main_fun(user_line, server_line, git_line):
      headers = {
          "user-agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36",
          "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9"
          }
      keep = requests.Session()
      div = BeautifulSoup(keep.get("https://ctf.bugku.com/login".rstrip(), headers=headers, verify=False).text, 'lxml')
      git_url = div.find('a', class_='btn btn-floating btn-github')['href']
      git_cookie = 'user_session={git_line1}; __Host-user_session_same_site={git_line2};'.format(git_line1=git_line, git_line2=git_line )
      headers['cookie'] = git_cookie
      flag = keep.get("https://github.com/settings/profile", headers=headers, verify=False, allow_redirects=False)
      if(flag.status_code!= 200):
          server_send(user_line, server_line, "github的cookie失效了噢！")
      res = keep.get(git_url, headers=headers, verify=False)
      login_status(user_line,server_line, res)
      if("github.githubassets.com" in res.text):
          print(res.text)
          choose = res.text.split('<form action="')[1].split('<input type="hidden" name="scope"')[0]
          rule = re.compile('name="(.*?)".*?value="(.*?)"')
          form_data = rule.findall(choose)
          Data = {}
          for i in form_data:
              Data[i[0]] = i[1]
          Data['authorize'] = 1
          formurl = "https://github.com" + choose.split('"')[0]
          headers = {
              "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36",
              "cookie": git_url
          }
          res = keep.post(formurl, data=Data, headers=headers, verify=False)
          login_status(user_line,server_line, res)
      elif("登录成功" not in res.text):
          server_send(user_line, server_line, "超时错误")
      keep.cookies.clear()
      keep.close()
  
  def main():
    pass
  
  def handler(event, context):
      user_list, server_list, git_list = load_txt()
      for i in range(len(user_list)):
          main_fun(user_list[i], server_list[i], git_list[i])
  
  if __name__ == '__main__':
      main()
      
  ```

  配置.txt，记得把对应的参数加进去

  ```txt
  user=你的用户名# server_key=填写你的server酱# git_cookie=填写抓包获得的githubcookie#
  ******author:dota_st******
  ```

  - 然后去注册一下server酱，绑定你的微信。然后通过调用server酱，把打卡结果发送到微信上
    地址：https://sct.ftqq.com/

    ![image-20240214151550815](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/21a214764f831b721fdd2f846614719c_MD5.png)



## 云函数部署

### 创建云函数

[阿里云云函数网址](https://www.aliyun.com/product/fc?spm=a2c4g.750001.J_4VYgf18xNlTAyFFbOuOQe.158.726e6999Ze92Y2&scm=20140722.S_product@@%E4%BA%91%E4%BA%A7%E5%93%81@@90871._.ID_product@@%E4%BA%91%E4%BA%A7%E5%93%81@@90871-RL_fc-LOC_menu~UND~product-OR_ser-V_3-P0_0)

首次开通不收费，跟随指导直接默认开通就行

进入管理控制台 -> 选择函数 -> 创建函数 -> 选择事件函数 -> 填写函数名称，运行环境选择python，通过文件或压缩包上传代码 ->点击创建

![D:\Document\images202402141523740.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/D%3A%5CDocument%5Cimages202402141523740.png)


![D:\Document\images202402141526820.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/D%3A%5CDocument%5Cimages202402141526820.png)


### 配置运行环境

因为脚本需要用到一些外部库，因此需要下载，点击函数名进入函数，往下拉到在线编辑器界面，进入终端
![D:\Document\images202402141531768.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/D%3A%5CDocument%5Cimages202402141531768.png)



进入终端后依次输入命令：

```shell
pip install --upgrade pip
pip install retrying -t .
pip install bs4 -t .
```

这三个命令的意思是更新pip，下载retrying库和bs4库脚本所在目录，库的具体作用自行百度

完成后点击部署代码，点击测试函数，可以看到脚本执行后的回显（我已经签过到了）

![D:\Document\images202402141547406.png](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/D%3A%5CDocument%5Cimages202402141547406.png)


### 设置定时任务

往上找到配置点击后在触发器功能里创建触发器，选择定时触发器 ->输入定时器名称 自定义cron表达式 CRON_TZ=Asia/Shanghai 0 0 7 * * *（每天7点执行，可以更改数字7改变时间）点击确定。至此全部完成

![](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/63870de59e1b1a41fa574646e976b9e7_MD5.png)

![image-20240214155240449](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/f39b959c95ead0ab4f2742e89d5d9fe2_MD5.png)

![image-20240214155255335](https://cdn.jsdelivr.net/gh/halcyon098/study-img/study-images/3731a6ffaab0b834d531632d581b1e33_MD5.png)