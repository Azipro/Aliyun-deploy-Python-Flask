# 阿里云Ubuntu18.04服务器上部署Python + Flask项目


### 将项目所在虚拟环境的依赖包导入一个生成txt文件中，便于服务器端安装所需依赖包
```在虚拟环境中```
```shell
    pip freeze > D:\Python\Project\Cattttt\requirements.txt
```


### 切记项目中所有涉及到的绝对地址都要改为相对地址
用好```import os```


### 若使用到数据库Mysql, 需要导出整个数据库，生成.sql文件
```shell
    mysqldump -u 用户名 -p 数据库名 > 项目路径/导出的文件名
```


### 项目结构
```
└── Cattttt
│ ├── logs
│ └── app //Flask
│ │ └── __init__.py 
│ ├── app.py
│ ├── requirements.txt
│ ├── *.sql
```
打包成*.tar.gz


### 把压缩包上传到服务器

可以用```Ftp``` ```WinSCP```...


### 在命令行使用ssh连接服务器
```shell
    ssh root@ip地址
```
输入密码后进入到Shell

安装与自己项目匹配的python版本
```shell
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:jonathonf/python-3.7
    sudo apt-get update
    sudo apt-get install python3.7
```
修改系统默认python版本
```shell
    cd /user/bin
    rm python
    ln -s python3.7 python
```
升级pip
```shell
    python pip install –upgrade pip
```
安装虚拟环境
```shell
    sudo  pip install virtualenv
```
将打包好的项目解压出来, 放到指定位置(/var/www)
```shell
    mv xxx.tar.gz /var/www
    tar -zxvpf xx.tar.gz
```
导入数据库
```shell
    mysql -u 用户名 -p 数据库名 < 数据库名.sql
```
切换到项目目录中, 创建虚拟环境, 并切换到虚拟环境
```shell
    cd /var/www/Cattttt
    virtualenv venv
    source venv/bin/activate
```
使用之前导出的requestments.txt文件批量安装依赖包
```shell
    pip install -r requirements.txt
```

#### 注意
查看虚拟环境所属python版本, 切换到正确版本
```shell
    ls venv/bin/ 
    virtualenv -p /usr/bin/python3.7 venv
```

此时文件结构:
```
└── Cattttt  
│   ├── logs
│   └── venv
│   │   ├── bin
│   │   │         ├── activate
│   │   │         ├── easy_install
│   │   │         ├── gunicorn
│   │   │         ├── pip
│   │   │         └── python
│   │   ├── include
│   │   │          └── python3.7
│   │   ├── lib
│   │   │         └── python3.7
│   │   ├── local
│   │   │         ├── bin
│   │   │         ├── include
│   │   │         └── lib
│   └── app  //Flask
│   │           └──  __init__.py
│   ├── app.py   
│   ├── requirements.txt
│   ├── *.sql
```

### 安装Nginx
```shell
    sudo apt-get install nginx
```
配置:
```shell
    vim /etc/nginx/sites-available/default
```
通过ip地址配置(还可以使用域名配置)
```
server {
    listen 80;  # nginx监听的端口（默认为80, 可以改为其它端口号）
    server_name 服务器ip;
    root  /var/www/Cattttt/app/static;     # 自己的Flask项目中的静态文件目录路径/ 可以不设置
    index ../templates/user/login.html;    # 主页的路径/可以不设置
    location / { # location是用以指定请求要转发到的目标服务器运行的地址
                   #try_files $uri $uri/ =404;
                include  uwsgi_params;  # uwsgi默认的配置参数名
                uwsgi_pass   127.0.0.1:8088;  # 指向uwsgi 所应用的内部地址, 所有请求将转发给uwsgi处理
                uwsgi_param UWSGI_PYHOME /var/www/Cattttt/venv; # 指向虚拟环境目录
                uwsgi_param UWSGI_CHDIR  /var/www/Cattttt; # 指向网站根目录
                uwsgi_param UWSGI_SCRIPT app:app; # 指定启动程序

    }
}
```

### 安装uwsgi
```shell
    apt-get install libpython3.x-dev //安装了这个依赖, 才能正常安装uwsgi
    pip install uwsgi
```
检查uwsgi的python版本
```shell
    uwsgi --python-version
```
如果和项目所需python版本不匹配, 先把系统默认版本改为需要版本, 然后删除uwsgi重装
```shell
    pip uninstall uwsgi
    pip install uwsgi
```

配置:
```shell
    touch config.ini
    vim config.ini
    
config.ini:
{
[uwsgi]

socket = 127.0.0.1:8088  # uwsgi启动时使用的地址和端口

chdir = /var/www/Cattttt  # 项目路径

wsgi-file = app.py    #  python启动文件

callable = app      #  app.py里的Flask变量名

processes = 1     # 处理器数

threads = 2   #  线程数

stats = 127.0.0.1:5555  # Flask默认端口

daemonize = /var/www/Cattttt/logs/uwsgi.log  # 日志
}

启动uwsgi
    uwsgi config.ini
```


### 最后
如果启动uwsgi报端口占用
```shell
    killall -9 uwsgi
    uwsgi config.ini
```
重启Nginx服务
```shell
    sudo service nginx restart
```
