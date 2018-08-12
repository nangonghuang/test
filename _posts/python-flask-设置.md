---
title: python flask 设置
date: 2018-07-20 16:07:36
tags:
categories: python
---

几个月之前刚学python后看完了狗书，照着上面写完了flask的项目，然后在公司内部写了个后台网站，主要是数据库的操作和调用外部工具.然后就没管了。
现在接入sdk的时候需要一个游戏服务器，只好自己搭一个来测试。发现flask相关的东西又忘光了。。。<!--more-->

环境 ：python 3.6 ,vscode 

1. 首先是建立虚拟目录 : `python -m venv venv` 
2. windows下，编写一个bat : `start venv\scripts\activate.bat` ，双击bat启动虚拟环境
    
    linux下，terminal中执行 `source venv/bin/activate ` 启动虚拟环境
3. 进入虚拟环境，pip install flask
4. 新建python文件，输入
    ```python
    # -*- coding:utf-8 -*-
    from flask import Flask

    app = Flask(__name__)


    @app.route('/',methods=['GET', 'POST'])
    def hello_world():
        return 'Hello Flask!'


    if __name__ == '__main__':
         app.run(host='0.0.0.0',port=5000,debug=True)
    ```
5. 在vscode中ctrl+p, 输入>python ,选择调试器，选择venv,这时候flask应该不会报错了
6. 虚拟环境中直接运行python文件
