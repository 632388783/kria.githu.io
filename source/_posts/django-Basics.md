---
layout: photo
title: Django 基础教程
index_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
banner_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
cover: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
date: 2025-02-27 09:30:22
tags: [Python, Django]
categories: python
description: Django 基础教程
comment: 'valine'
---

# Django 基础教程

欢迎来到 Django 的世界！Django 是一个强大的 Web 应用框架，旨在帮助开发者快速构建安全、可扩展的应用。无论您是刚接触后端开发还是有经验的开发者，Django 都能为您提供高效的工具和库。

## 快速入门

### 安装 Django

首先，确保你已安装 Python。然后，你可以使用 pip（标准 Python 包管理器）来安装 Django。

```bash
$ pip install django
```

更多信息：[安装](Installation)

### 创建新的 Django 项目

安装好 Django 后，你可以创建一个新项目。在终端中导航到你想要创建项目的目录。

```bash
$ django - admin startproject myproject
```

这将创建一个名为`myproject`的新目录，其中包含 Django 项目的基本结构。
更多信息：[项目设置](Project Setup)

### 在项目中创建新应用

Django 项目由一个或多个应用组成。要创建新应用，导航到项目目录（在本例中为`myproject`）。

```bash
$ python manage.py startapp myapp
```

这将创建一个名为`myapp`的新应用，它有自己的一组模型、视图和模板。
更多信息：[应用创建](App Creation)

### 运行开发服务器

要启动 Django 开发服务器并查看项目运行情况，在项目目录中运行以下命令。

```bash
$ python manage.py runserver
```

然后你可以在 Web 浏览器中通过`http://127.0.0.1:8000/`访问你的项目。
更多信息：[运行服务器](Running the Server)

### 创建简单视图

在你的应用（`myapp`）的`views.py`文件中，你可以定义一个简单的视图函数。例如：

```python
from django.http import HttpResponse

def hello_world(request):
    return HttpResponse("Hello, World!")
```

然后，你需要将这个视图映射到一个 URL。在应用的`urls.py`文件中，添加以下代码：

```python
from django.urls import path
from. import views

urlpatterns = [
    path('hello/', views.hello_world, name='hello_world'),
]
```

最后，将应用的`urls.py`包含在项目的`urls.py`中。

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

现在，当你在浏览器中访问`http://127.0.0.1:8000/hello/`时，你应该会看到“Hello, World!”消息。
更多信息：[编写视图](Writing Views)

### 数据库设置

Django 内置支持多种数据库。默认情况下，它使用 SQLite。如果你想使用其他数据库，如 PostgreSQL 或 MySQL，则需要在项目的`settings.py`文件中进行配置。例如，要使用 PostgreSQL：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'your_database_name',
        'USER': 'your_username',
        'PASSWORD': 'your_password',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
}
```

更改数据库设置后，你需要运行迁移以创建必要的数据库表。

```bash
$ python manage.py makemigrations
$ python manage.py migrate
```

更多信息：[数据库设置](Database Setup)

### 部署

一旦你准备好将 Django 应用部署到生产服务器，涉及几个步骤。你需要配置一个 Web 服务器（如 Nginx 或 Apache），设置一个生产就绪的数据库，并管理应用的环境。一种流行的部署 Django 应用的方法是使用 Heroku。

```bash
# 安装Heroku CLI
$ sudo snap install --classic heroku

# 登录Heroku
$ heroku login

# 创建新的Heroku应用
$ heroku create myapp - name

# 将代码推送到Heroku
$ git push heroku main
```

更多信息：[部署](Deployment)
