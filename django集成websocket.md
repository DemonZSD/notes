python + django + django channels 实现websocket通信，解决刷新页面不中断的问题 
March 28, 2018 8:45 AM
##### 介绍
**背景**：
- web通信在项目开发中会经常遇到。为了保证在不刷新页面的前提下，使客户端（浏览器）与服务器之间进行数据交互。传统上，使用HTTP方式，进行轮询，也就是使用ajax技术。本教程会研究使用websocket方式，解决客户端与服务器之间的通信。具体关于http短连接以及websocket通信的优缺点请移目[传统轮询、长轮询、服务器发送事件与WebSocket](https://blog.zhangruipeng.me/2015/10/22/Web-Connectivity/)
- 本教程django项目中使用websocke进行通信，保证浏览器与客户端实时通信，并能解决页面刷新，websocket连接依然不再中断.
- Django channels 是介于网络协议服务和 Python 应用之间的标准接口，能够处理多种通用协议类型，包括 HTTP、HTTP2 和 WebSocket。该团队提出了ASGI（Asynchronous Server Gateway Interface， 异步服务网关接口）（[ASGI介绍](https://blog.ernest.me/post/asgi-draft-spec-zh)）的概念，是基于WSGI（[WSGI介绍](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832689740b04430a98f614b6da89da2157ea3efe2000)）的拓展。

**语言及框架**
- python 2.7  [官网](https://www.python.org/download/releases/2.7/)
- Django 1.11 [官网](https://docs.djangoproject.com/en/2.0/releases/1.11/)
- Django channels 1.1.8  [官网](http://channels.readthedocs.io/en/1.x/)

**开发工具**
- Pycharm 社区版 [官网](https://www.jetbrains.com/pycharm/download/#section=windows)

##### 环境搭建
**python**安装
- [Python 环境搭建](http://www.runoob.com/python/python-install.html)

**Django 1.11安装**
- 使用pip进行安装
   `(venv) C:\GitWorkspace\demoDjangoChannel>pip install django==1.11`
- 查看django版本
  ```
  (venv) C:\GitWorkspace\demoDjangoChannel>python -m django --version
   1.11.11
  ```

**Django channels 1.1.8安装**
- 使用pip安装channels

  `(venv) C:\workspace\demoDjangoChannel>pip install channels==1.1.8`

- 查看版本

  `(venv) C:\GitWorkspace\demoDjangoChannel>pip freeze > requirements.txt`
  
  - 说明：
  
  `pip freeze`输出已经安装的packages 
  
  具体用法请看[官方文档](https://pip.pypa.io/en/stable/reference/pip_freeze/)
  
  - 已安装的pacakages列表
  **channels==1.1.8**
  **Django==1.11.11**
  **djangorestframework==3.7.7**


