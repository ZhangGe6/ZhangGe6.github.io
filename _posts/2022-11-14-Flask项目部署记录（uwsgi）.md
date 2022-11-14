---
title: Flask项目部署记录（uwsgi）
author: Zhang Ge
date: 2022-11-14 21:17:00 +0800
categories: [实践, 硬技能]
tags: [Flask, uwsgi]
---

[Flask](https://dormousehole.readthedocs.io/en/latest/quickstart.html)是一个使用Python编写的轻量级Web应用框架。基于它可以很方便地搭建起一个Web应用，但其内建服务器不适用于生产环境。所以当在本地完成了一个Flask应用，为了让它更高效、安全、稳定地把它展示给全世界，是时候部署它了！

本页记录我把一个Flask应用部署到腾讯云服务器上的操作过程，用到了[`uwsgi`](https://uwsgi-docs.readthedocs.io/en/latest/index.html)。

> 再进一步地，为了支持高性能、高并发，还可以用到nginx。但这一步我暂时还没弄清楚，等后续有机会实操并成功后再做记录。

# 服务器准备

在腾讯云租用一个服务器，会分配一个公网ip地址。

![](/assets/img/20221114/console.png)

- （首次连接前，需要）配置服务器的SSH用户名和密码
  - 我租用的是一台“轻量级服务器”，它的SSH配置指南见[这里](https://cloud.tencent.com/document/product/1207/44643)
    - :star:<u>**关于端口**</u>
      -  创建实例时默认已开通`22`端口，可供SSH连接使用。
      - 其他端口放行情况可见服务器详情的”防火墙“栏，我们后续搭建Flask应用会使用已默认放行的`3389`端口。

- 配置本地和服务器的SSH连接
  - 配置方法可参见[这里](https://zhangge6.github.io/posts/VS-code%E4%BD%BF%E7%94%A8/#%E9%85%8D%E7%BD%AE%E7%A7%81%E9%92%A5)


ok，可以开始我们的开发了！

# Flask应用搭建和它的裸奔

以一个最简单的Flask应用为例，新建`main.py`

```python
# main.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Hello World!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3389)
```

执行

```bash
python main.py
```

输出：

```bash
 * Serving Flask app 'main'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:3389
 * Running on http://10.0.4.2:3389
Press CTRL+C to quit
```

这时，我们本地登录 `http://服务器公网ip:3389`，即可以看到如下界面

![](/assets/img/20221114/raw_flask.png)

**Well done!** 咱们的Flask应用可以访问了！但是可以看到终端输出的警告：

```
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
```

这是由于Flask内建服务器不够高效、安全、稳定，所以不适用于生产环境。按照提示，我们使用`WSGI server`来部署它。

# WSGI server部署Flask应用

## 安装`uwsgi`

- 常规的方法是用pip安装 [参考](https://uwsgi-docs.readthedocs.io/en/latest/Install.html)

  ```bash
  pip install uwsgi
  ```

- 如果使用的是Conda虚拟环境，可以从conda forge安装 [参考](https://anaconda.org/conda-forge/uwsgi)，如

  ```bash
  conda install -c "conda-forge/label/cf202003" uwsgi
  ```

## 基于uwsgi运行flask应用 

[参考](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html)

新建`uwsgi.ini`

```ini
[uwsgi]
; 指定ip和端口，应保持和之前main.py中app的定义一致 (此时main.py中的ip/端口定义可以略去)
http = 0.0.0.0:3389

; 指定flask项目启动文件地址 
chdir = path/to/your/flask_project
wsgi-file = main.py

; 显式指明main.py中定义的callable <== app = Flask(__name__)
callable = app 

; 
master = true

; 指定进程/线程数目
processes = 4
threads = 2

; 指定虚拟环境地址
virtualenv = /path/to/virtual/environment
```

**<u>注：</u>**

- 如果`uwsgi.ini`和`flask`项目启动文件在同一目录下，`chdir`项可省略。
- 如果是在虚拟环境中运行`uwsgi`，则`virtualenv`项需要指定。[参考](https://flask.palletsprojects.com/en/1.1.x/deploying/uwsgi/#starting-your-app-with-uwsgi)
  - 注意，当前运行所处的虚拟环境，应该与`virtualenv`项保持一致！
  - 终端使用 `echo $CONDA_PREFIX` 可以得到当前虚拟环境的本机路径。

- `master`项的具体含义可以参考[这里](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html#adding-concurrency-and-monitoring)，暂时还不太理解。如果没有指定`master = true`的话，会有`WARNING: you are running uWSGI without its master process manager`

执行

```bash
uwsgi uwsgi.ini
```

会有一堆输出。如果没有报错，则本地登录 `http://服务器公网ip:3389`，再次出现

![](/assets/img/20221114/raw_flask.png)

成功！



