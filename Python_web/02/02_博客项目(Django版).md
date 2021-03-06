---
title: 博客项目(Django版)项目
tags: Django
notebook: 7.0第五月 Python_web后端
---

[toc]


# 一、项目概述

## 项目运行环境
1. Python3.6+
2. Django 1.11
3. MySQL 5.7
4. 其他插件(图片处理、分页、验证码....)
## 项目详细功能介绍
### 前台功能
1. 项目首页展示
2. 轮播图
3. 博客推荐
4. 最新发布
5. 博客分类
6. 最新评论文章
7. widgets小插件
8. 搜索功能
9. 博客分类功能
10. 博客标签查询
11. 友情链接
12. 博客分页功能
13. 博客详细
14. 最新评论文章
15. 发表评论
16. 评论展示
17. 评论数
18. 阅读数
19. 登录功能
20. 注册功能
21. 邮箱验证功能
22. 注销功能
23. 页面模板
24. 标签云功能
25. 读者墙功能

### 后台功能
1. 用户维护
2. 权限管理
3. 博客分类维护
4. 标签维护
5. 友情链接
6. 轮播图维护

### 项目演示
项目演示
### 项目代码演示
代码展示

# 二、开发环境搭建

> 使用virtualenv 和 virtualenwrapper

1. MySQL 5.7

```vim
sudo apt install mysql-server mysql-client
```

2. 安装mysql驱动

```vim
pip install pymysql
```

3. 安装Django

```vim
pip install django==1.11
```



# 三、创建项目
## 创建项目和应用

- 创建项目

```vim
django-admin startproject django-blog
```

- 创建应用

```vim
python manage.py startapp userapp
python manage.py startapp blogapp
```


## 配置数据库

- 在settings中配置数据库

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_blog_db',
        'USER': 'root',
        'PASSWORD': 'wwy123',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```

## 创建数据库(执行迁移文件)

```python
python manage.py migrate
```

## 创建超级管理员
```python
python manage.py createsuperuser
```

# 四、创建数据模型

## USERAPP
###　USER(用户模型)

```python
from django.contrib.auth.models import AbstractUser

class BlogUser(AbstractUser):
    nikename = models.CharField('昵称', max_length=20, default='')
```
> 提示：需要在settings配置文件中设置:AUTH_USER_MODEL = 'users.BlogUser'

### EMAIL(邮箱验证数据模型)

```python
class EmailVerifyRecord(models.Model):
    code = models.CharField(verbose_name='验证码', max_length=50,default='')
    email = models.EmailField(max_length=50, verbose_name="邮箱")
    send_type = models.CharField(verbose_name="验证码类型", choices=(("register",u"注册"),("forget","找回密码"), ("update_email","修改邮箱")), max_length=30)
    send_time = models.DateTimeField(verbose_name="发送时间", default=datetime.now)

    class Meta:
        verbose_name = "邮箱验证码"
        # 复数
        verbose_name_plural = verbose_name

    def __str__(self):
        return '{0}({1})'.format(self.code, self.email)
```
## BLOGAPP
### Banner(轮播图模型)
```python
class Banner(models.Model):
    title = models.CharField('标题', max_length=50)
    cover = models.ImageField('轮播图', upload_to='static/images/banner')
    link_url = models.URLField('图片链接', max_length=100)
    idx = models.IntegerField('索引')
    is_active = models.BooleanField('是否是active', default=False)

    def __str__(self):
        return self.title
    class Meta:
        verbose_name = '轮播图'
        verbose_name_plural = '轮播图'
```

### Category(博客分类模型)
```python
class BlogCategory(models.Model):
    name = models.CharField('分类名称', max_length=20, default='')
    class Meta:
        verbose_name = '博客分类'
        verbose_name_plural = '博客分类'

    def __str__(self):
        return self.name
```

### Tags(标签模型)
```python
class Tags(models.Model):
    name = models.CharField('标签名称', max_length=20, default='')
    class Meta:
        verbose_name = '标签'
        verbose_name_plural = '标签'

    def __str__(self):
        return self.name
```

### Blog(博客模型)
```python
class Post(models.Model):
    user = models.ForeignKey(BlogUser, verbose_name='作者')
    category = models.ForeignKey(BlogCategory, verbose_name='博客分类', default=None)
    tags = models.ManyToManyField(Tags, verbose_name='标签')
    title = models.CharField('标题', max_length=50)
    content = models.TextField('内容')
    pub_date = models.DateTimeField('发布日期', default=datetime.now)
    cover = models.ImageField('博客封面', upload_to='static/images/post', default=None)
    views = models.IntegerField('浏览数', default=0)
    recommend = models.BooleanField('推荐博客', default=False)

    def __str__(self):
        return self.title
    class Meta:
        verbose_name = '博客'
        verbose_name_plural = '博客'
```

### Comment(评论模型)

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, verbose_name='博客')
    user = models.ForeignKey(BlogUser, verbose_name='作者')
    pub_date = models.DateTimeField('发布时间')
    content = models.TextField('内容')

    def __str__(self):
        return self.content
    class Meta:
        verbose_name = '评论'
        verbose_name_plural = '评论'
```

### FriendlyLink(友情链接模型)

```python
class FriendlyLink(models.Model):
    title = models.CharField('标题', max_length=50)
    link = models.URLField('链接', max_length=50, default='')

    def __str__(self):
        return self.title
    class Meta:
        verbose_name = '友情链接'
        verbose_name_plural = '友情链接'
```

# 五、实现首页页面模板
## 创建模板文件夹
> 创建模板文件templates,并在settings.py中设置

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```
## 配置静态文件路径

```python
STATIC_URL = '/static/'

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
```


# 六、创建首页路由

- 创建视图函数
```python
def index(request):
    return render(request, 'index.html',  {})
```
- 配置url
```python
url(r'^$', index, name='index' )
```
- 修改模板CSS　JS等静态文件的路径


# 七、实现幻灯片功能(Banner)

- 注册模型
```python
from blogs.models import Banner
admin.site.register(Banner)
```
- 编写views
```python
from .models import Banner

def index(request):
    banner_list = Banner.objects.all()
    ctx = {
        'banner_list': banner_list,    
    }
    return render(request, 'index.html',  ctx)
```

- 模板
```python
  <!-- banner 开始 -->
  <div id="focusslide" class="carousel slide" data-ride="carousel">
	<ol class="carousel-indicators">

    {% for banner in banner_list %}
    {% if banner.is_active %}
	  <li data-target="#focusslide" data-slide-to="{{banner.idx}}" class="active"></li>
    {% else %}
	  <li data-target="#focusslide" data-slide-to="{{banner.idx}}"></li>
    {% endif %}
    {% endfor %}

	</ol>
	<div class="carousel-inner" role="listbox">

    {% for banner in banner_list %}
    {% if banner.is_active %}
      <div class="item active">
      <a href="{{banner.link_url}}" target="_blank" title="{{banner.title}}" >
      <img src="{{banner.cover}}" alt="{{banner.title}}" class="img-responsive"></a>
      </div>
    {% else %}
      <div class="item">
      <a href="{{banner.link_url}}" target="_blank" title="{{banner.title}}" >
      <img src="{{banner.cover}}" alt="{{banner.title}}" class="img-responsive"></a>
      </div>
    {% endif %}
    {% endfor %}

	</div>
	<a class="left carousel-control" href="#focusslide" role="button" data-slide="prev" rel="nofollow">
     <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
     <span class="sr-only">上一个</span> </a>
  <a class="right carousel-control" href="#focusslide" role="button" data-slide="next" rel="nofollow">
    <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
    <span class="sr-only">下一个</span> </a>
  </div>

  <!-- banner 结束 -->
```
# 八、实现博客推荐
- 注册模型
```python
from blogs.models import Banner,Post,BlogCategory,Tags
...
admin.site.register(BlogCategory)
admin.site.register(Tags)
admin.site.register(Post)
```
- 编写views
```python

# 视图函数 HTTPRequest
def index(request):
    banner_list = Banner.objects.all()
    recommend_list = Post.objects.filter(recommend=1)
    ctx = {
        'banner_list': banner_list,
        'recommend_list': recommend_list,      
    }
    return render(request, 'index.html',  ctx)
```
- 模板
```html
  <!-- 推荐开始 -->
  {% for post in recommend_list %}
  <article class="excerpt-minic excerpt-minic-index">
		<h2><span class="red">【推荐】</span><a target="_blank" href="/blog/{{post.id}}/" title="{{post.title}}" >{{post.title}}</a>
		</h2>
		<p class="note">{{post.content}}</p>
	</article>
  {% endfor %}
  <!-- 推荐结束 -->
```
# 九、实现最新发布
- 编写views
```python
# 视图函数 HTTPRequest
def index(request):
    ...
    post_list = Post.objects.order_by('-pub_date').all()[:10]
    ....
    ctx = {
        'banner_list': banner_list,
        'recommend_list': recommend_list,
        'post_list': post_list,
        
    }
    return render(request, 'index.html',  ctx)
```
- 模板
```python
  <!-- 最新发布的博客开始 -->

  {% for post in post_list%}

  <article class="excerpt excerpt-1" style="">
  <a class="focus" href="/blog/{{post.id}}/" title="{{post.title}}" target="_blank" ><img class="thumb" data-original="/{{post.cover}}" src="/{{post.cover}}" alt="{{post.title}}"  style="display: inline;"></a>
		<header><a class="cat" href="#" title="{{post.category.name}}" >{{post.category.name}}<i></i></a>
			<h2><a href="/blog/{{post.id}}/" title="{{post.title}}" target="_blank" >{{post.title}}</a>
			</h2>
		</header>
		<p class="meta">
			<time class="time"><i class="glyphicon glyphicon-time"></i>{{post.pub_date|date:'Y-m-d'}}</time>
			<span class="views"><i class="glyphicon glyphicon-eye-open"></i>{{post.views}}</span> <a class="comment" href="##comment" title="评论" target="_blank" ><i class="glyphicon glyphicon-comment"></i>{{post.comment_set.count}}</a>
		</p>
		<p class="note">


      {% autoescape off %}
          {{post.content | truncatechars_html:200}}
      {% endautoescape %}

    </p>
	</article>

  {% endfor %}


  <!-- 最新发布的博客结束 -->
```
# 十、实现博客分类功能
- 编写视图
```python
# 视图函数 HTTPRequest
def index(request):
    banner_list = Banner.objects.all()
    recommend_list = Post.objects.filter(recommend=1)
    post_list = Post.objects.order_by('-pub_date').all()[:10]
    blogcategory_list = BlogCategory.objects.all()

    ctx = {
        'banner_list': banner_list,
        'recommend_list': recommend_list,
        'post_list': post_list,
        'blogcategory_list': blogcategory_list,
    }
    return render(request, 'index.html',  ctx)
```
- 模型
```html
  <div class="title">
	<h3>最新发布</h3>
	<div class="more">
      {%for c in blogcategory_list%}
			  <a href="/category/{{c.id}}" title="{{c.name}}" >{{c.name}}</a>
			{% endfor %}
		</div>
  </div>
```

# 十一、实现最新评论功能
- 编写views
```python
 <ul>

                {% for post in new_comment_list %}
                    <li><a title="{{ post.title }}" href="#"><span class="thumbnail">
				<img class="thumb" data-original="/{{ post.cover }}"
                     src="/{{ post.cover }}"
                     alt="{{ post.title }}" style="display: block;">
			</span><span class="text">{{ post.title }}</span><span class="muted"><i
                            class="glyphicon glyphicon-time"></i>
				{{ post.pub_date }}
			</span><span class="muted"><i class="glyphicon glyphicon-eye-open"></i>{{ post.views }}</span></a></li>
                {% endfor %}


            </ul>
```
# 十二、实现搜索功能
- 编写views
```python
from django.views.generic.base import View
from django.db.models import Q
class SearchView(View):
    # def get(self, request):
    #     pass
    def post(self, request):
        kw = request.POST.get('keyword')
        post_list = Post.objects.filter(Q(title__icontains=kw)|Q(content__icontains=kw))

        ctx = {
            'post_list': post_list
        }
        return render(request, 'list.html', ctx)
```
- urls
```python
url(r'^search$', SearchView.as_view(), name='search'),
```

# 十三、实现友情链接
- 编写视图  (数据源)
```python
def index(request):
    ....
    friendlylink_list = FriendlyLink.objects.all()
    .....
```
- 模板
```python

        <div class="widget widget_sentence">
            <h3>友情链接</h3>
            <div class="widget-sentence-link">
                {% for friendlylink in friendlylink_list %}

                    <a href="{{ friendlylink.link }}" title="{{ friendlylink.title }}"
                       target="_blank">{{ friendlylink.title }}</a>&nbsp;&nbsp;&nbsp;
                {% endfor %}


            </div>
        </div>
```

# 十四、实现博客列表功能
- 编写views
```python
def blog_list(request):
    post_list = POST.objects.all()
    ctx = {
        'post_list': post_list,
    }
    return render(request, 'list.html', ctx)
```
- 编写路由
```python
url(r'^list$', blog_list, name='blog_list'),
```
- base.html
```html
<!doctype html>
<html lang="zh-CN">
<head>
<meta charset="utf-8">
<meta name="renderer" content="webkit">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>{% block title %}知奇博客首页 {% endblock %}</title>
<meta name="keywords" content="">
<meta name="description" content="">


{% block custom_css %}{% endblock %}
<link rel="stylesheet" type="text/css" href="/static/css/bootstrap.min.css">
<link rel="stylesheet" type="text/css" href="/static/css/nprogress.css">
<link rel="stylesheet" type="text/css" href="/static/css/style.css">
<link rel="stylesheet" type="text/css" href="/static/css/font-awesome.min.css">
<link rel="apple-touch-icon-precomposed" href="/static/images/icon.png">
<link rel="shortcut icon" href="/static/images/favicon.ico">

<script src="/static/js/jquery-2.1.4.min.js"></script>
<script src="/static/js/nprogress.js"></script>
<script src="/static/js/jquery.lazyload.min.js"></script>

</head>
<body class="user-select">
<header class="header">
<nav class="navbar navbar-default" id="navbar">
<div class="container">
  <div class="header-topbar hidden-xs link-border">
  	<ul class="site-nav topmenu">
  	  <li><a href="#" >标签云</a></li>
  		<li><a href="#" rel="nofollow" >读者墙</a></li>
  		<li><a href="#" title="RSS订阅" >
  			<i class="fa fa-rss">
  			</i> RSS订阅
  		</a></li>
  	</ul>
			 爱学习 更爱分享
  </div>
  <div class="navbar-header">
	<button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#header-navbar" aria-expanded="false"> <span class="sr-only"></span> <span class="icon-bar"></span> <span class="icon-bar"></span> <span class="icon-bar"></span> </button>
	<h1 class="logo hvr-bounce-in"><a href="#" title="知奇课堂博客"><img src="/static/images/201610171329086541.png" alt="知奇课堂博客"></a></h1>
  </div>
  <div class="collapse navbar-collapse" id="header-navbar">

	<ul class="nav navbar-nav navbar-left">
	  <li><a data-cont="知奇课堂博客" title="知奇课堂博客" href="/">首页</a></li>
	  <li><a data-cont="博客" title="博客" href="/list">博客</a></li>
	</ul>


  <ul class="nav navbar-nav navbar-right">
    {% if user.is_authenticated %}
    <li><a data-cont="用户" title="用户" href="#">欢迎，{{user.username}}</a></li>
    <li><a data-cont="注册" title="注册" href="/logout">注销</a></li>
    {% else %}
    <li><a data-cont="登录" title="登录" href="/login">登录</a></li>
    <li><a data-cont="注册" title="注册" href="/register">注册</a></li>
    {% endif %}
  </ul>

  </div>
</div>
</nav>
</header>

{% block content %}
{% endblock %}

<footer class="footer">
<div class="container">
<p>Copyright &copy; 2016.Company name All rights reserved.</p>
</div>
<div id="gotop"><a class="gotop"></a></div>
</footer>
<script src="/static/js/bootstrap.min.js"></script>
<!-- <script src="/static/js/jquery.ias.js"></script> -->
<script src="/static/js/scripts.js"></script>
</body>
</html>

```
# 十五、实现分页功能
- 安装包
```python
pip install django-pure-pagination
```
> 参考链接: https://github.com/jamespacileo/django-pure-pagination

```python
def blog_list(request):
    post_list = Post.objects.all()
    try:
        page = request.GET.get('page', 1)
    except PageNotAnInteger:
        page = 1
    p = Paginator(post_list, per_page=1, request=request)
    post_list = p.page(page)
    ctx = {
        'post_list': post_list,
        
    }
    return render(request, 'list.html', ctx)
```
- 模板
```python
<section class="container">
<div class="content-wrap">
<div class="content">
  <div class="title">
	<h3 style="line-height: 1.3">博客列表</h3>
  </div>
  {% for post in post_list.object_list %}
  <article class="excerpt excerpt-1"><a class="focus" href="/blog/{{post.id}}" title="{{post.title}}" target="_blank" >
    <img class="thumb" data-original="/{{post.cover}}" src="/{{post.cover}}" alt="{{post.title}}"  style="display: inline;"></a>
	<header><a class="cat" href="/category/{{post.category.id}}" title="{{post.category.name}}" >{{post.category.name}}<i></i></a>
	  <h2><a href="/blog/{{post.id}}" title="{{post.title}}" target="_blank" >{{post.title}}</a></h2>
	</header>
	<p class="meta">
	  <time class="time"><i class="glyphicon glyphicon-time"></i> {{post.pub_date|date:'Y-m-d'}}</time>
	  <span class="views"><i class="glyphicon glyphicon-eye-open"></i> {{post.views}}</span>
    <a class="comment" href="##comment" title="评论" target="_blank" ><i class="glyphicon glyphicon-comment"></i>{{post.comment_set.count}}</a></p>
	<p class="note">{{post.content}}</p>
  </article>
  {% endfor %}

  {% include "_pagination.html" %}
```
- _pagination.html
```python
{% load i18n %}
<div class="pagination">
    {% if post_list.has_previous %}
        <a href="?{{ post_list.previous_page_number.querystring }}" class="prev">&lsaquo;&lsaquo; 上一页</a>
    {% else %}
        <span class="disabled prev">&lsaquo;&lsaquo; 上一页</span>
    {% endif %}
    {% for page in post_list.pages %}
        {% if page %}
            {% ifequal page post_list.number %}
                <span class="current page">{{ page }}</span>
            {% else %}
                <a href="?{{ page.querystring }}" class="page">{{ page }}</a>
            {% endifequal %}
        {% else %}
            ...
        {% endif %}
    {% endfor %}
    {% if post_list.has_next %}
        <a href="?{{ post_list.next_page_number.querystring }}" class="next">下一页 &rsaquo;&rsaquo;</a>
    {% else %}
        <span class="disabled next">下一页 &rsaquo;&rsaquo;</span>
    {% endif %}
</div>

```


# 十六、实现标签云功能
```python
class TagMessage(object):
    def __init__(self, tid, name, count):
        self.tid = tid
        self.name = name
        self.count = count

def blog_list(request):
    post_list = Post.objects.all()
    try:
        page = request.GET.get('page', 1)
    except PageNotAnInteger:
        page = 1

    p = Paginator(post_list, per_page=1, request=request)

    post_list = p.page(page)

    tags = Tags.objects.all()
    tag_message_list = []
    for t in tags:
        count = len(t.post_set.all())
        tm = TagMessage(t.id, t.name, count)
        tag_message_list.append(tm)

    ctx = {
        'post_list': post_list,
        'tags': tag_message_list
    }
    return render(request, 'list.html', ctx)
```
- 模板
```html
	<h3>标签云</h3>
	<div class="widget-sentence-content">
		<ul class="plinks ptags">

      {% for t in tags %}
			<li><a href="/tags/{{t.tid}}" title="{{t.name}}" draggable="false">{{t.name}} <span class="badge">{{t.count}}</span></a></li>
      {% endfor %}
		</ul>
	</div>
  </div>
```
# 十七、实现分类查询功能
- 编写视图
```python
def blog_list(request, cid=-1):
    post_list = None
    if cid != -1:
       cat = BlogCategory.objects.get(id=cid)
       post_list = cat.post_set.all()
    else:
       post_list = Post.objects.all()

    ....

    ctx = {
        'post_list': post_list,
        'tags': tag_message_list
    }
    return render(request, 'list.html', ctx)
```
- 编写路由
```python
url(r'^category/(?P<cid>[0-9]+)/$', blog_list),
```
- 模板 index
```html
  <div class="title">
	<h3>最新发布</h3>
	<div class="more">
      {%for c in blogcategory_list%}
			  <a href="/category/{{c.id}}" title="{{c.name}}" >{{c.name}}</a>
			{% endfor %}
		</div>
  </div>
```
# 十八、实现按标签查询功能
- 编写views
```python
def blog_list(request, cid=-1, tid=-1):
    post_list = None
    if cid != -1:
       cat = BlogCategory.objects.get(id=cid)
       post_list = cat.post_set.all()
    elif tid != -1:
       tag = Tags.objects.get(id=tid)
       post_list = tag.post_set.all()
    else:
       post_list = Post.objects.all()

    ....

    ctx = {
        'post_list': post_list,
        'tags': tag_message_list
    }
    return render(request, 'list.html', ctx)
```
- 路由设置
```python
url(r'^tags/(?P<tid>[0-9]+)/$', blog_list),
```
- 模板
```python
	<h3>标签云</h3>
	<div class="widget-sentence-content">
		<ul class="plinks ptags">

      {% for t in tags %}
			<li><a href="/tags/{{t.tid}}" title="{{t.name}}" draggable="false">{{t.name}} <span class="badge">{{t.count}}</span></a></li>
      {% endfor %}
		</ul>
	</div>
  </div>
</div>
```
# 十九、实现博客详情功能
- 定义视图函数
```python
def blog_detail(request,bid):
    post = Post.objects.get(id=bid)
    post.views = post.views + 1
    post.save()

   

    # 博客标签
    tag_list = post.tags.all()

   
    ctx = {
        'post': post,
        

    }
    return render(request, 'show.html', ctx)
```
- 路由设置
```python
url(r'^blog/(?P<bid>[0-9]+)/$', blog_detail, name='blog_detail'),
```

- 前端展示

```html
{% extends 'base.html' %}
{% block title %}知奇博客-详细 {% endblock %}

{% block content %}
<section class="container">
<div class="content-wrap">
<div class="content">
  <header class="article-header">
	<h1 class="article-title"><a href="#" title="{{post.title}}" >{{post.title}}</a></h1>
	<div class="article-meta"> <span class="item article-meta-time">
	  <time class="time" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="发表时间：{{post.pub_date|date:'Y-m-d'}}">
      <i class="glyphicon glyphicon-time"></i> {{post.pub_date|date:'Y-m-d'}}</time>
	  </span>
    <span class="item article-meta-source" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="来源：{{post.user.username}}">
      <i class="glyphicon glyphicon-globe"></i> {{post.user.username}}</span>
      <span class="item article-meta-category" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="{{post.category.name}}">
        <i class="glyphicon glyphicon-list"></i> <a href="#" title="{{post.category.name}}" >{{post.category.name}}</a></span>
        <span class="item article-meta-views" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="浏览量：{{post.views}}">
          <i class="glyphicon glyphicon-eye-open"></i> {{post.views}}</span>
          <span class="item article-meta-comment" data-toggle="tooltip" data-placement="bottom" title="" data-original-title="评论量">
          <i class="glyphicon glyphicon-comment"></i> </span> </div>
  </header>
  <article class="article-content">

    <p>{{post.content}}</p>


  </article>
  <div class="article-tags">标签：
    {% for tag in post.tags.all %}
      <a href="/tags/{{tag.id}}">{{tag.name}}</a>
    {% endfor %}
	</div>
 
{% endblock %}
```

# 二十、实现相关推荐功能
- 编写视图
```python
def blog_detail(request, pid):
    post = Post.objects.get(id=pid)

    post.views = post.views + 1
    post.save()

    comment_list = Comment.objects.order_by('-pub_date')
    # 最新评论的博客  列表
    new_comment_list = []

    # 去重
    for c in comment_list:
        if c.post not in new_comment_list:
            new_comment_list.append(c.post)

    # 相关推荐
    # 首先  我们需要取到 这篇文章的tag
    tag_post_list = []
    for tag in post.tags.all():
        tag_post_list.extend(tag.post_set.all())

    ctx = {
        'post': post,
        'new_comment_list': new_comment_list,
        'tag_post_list': tag_post_list
    }
    return render(request, 'show.html', ctx)

```

- 模板
```html
 {% for tag_post in tag_post_list %}
                            <li><a href="{% url 'blog:detail' tag_post.id %}" title="">{{ tag_post.title }}</a></li>
                        {% endfor %}
```


# 二十一、实现发表评论的功能
- show.html
```html
	<h3>评论</h3>
  </div>
  <div id="respond">
		<form id="comment-form" name="comment-form" action="/comment/{{post.id}}" method="POST">
			<div class="comment">
				<input name="username" id="" value="{{user.username}}" class="form-control" size="22" placeholder="您的昵称（必填）" maxlength="15" autocomplete="off" tabindex="1" type="text">
				<!-- <input name="" id="" class="form-control" size="22" placeholder="您的网址或邮箱（非必填）" maxlength="58" autocomplete="off" tabindex="2" type="text"> -->
				<div class="comment-box">
					<textarea placeholder="您的评论或留言（必填）" name="content" id="comment-textarea" cols="100%" rows="3" tabindex="3"></textarea>
					<div class="comment-ctrl">
						<div class="comment-prompt" style="display: none;"> <i class="fa fa-spin fa-circle-o-notch"></i> <span class="comment-prompt-text">评论正在提交中...请稍后</span> </div>
						<div class="comment-success" style="display: none;"> <i class="fa fa-check"></i> <span class="comment-prompt-text">评论提交成功...</span> </div>
						<button type="submit" name="comment-submit" id="comment-submit" tabindex="4">评论</button>
					</div>
				</div>
			</div>
      {% csrf_token %}
		</form>
```
- 编写视图类
```python
class CommentView(View):
    def get(self, request):
        pass
    def post(self, request, bid):

        comment = Comment()
        comment.user = request.user
        comment.post = Post.objects.get(id=bid)
        comment.content = request.POST.get('content')
        comment.pub_date = datetime.now()
        comment.save()
        # Ajax
        return HttpResponseRedirect(reverse("blog_detail", kwargs={"bid":bid}))
```
- 编写路由
```python
url(r'^comment/(?P<bid>[0-9]+)$', CommentView.as_view(), name='comment'),
```
# 二十二、实现评论列表功能
- 编写视图
```python
def blog_detail(request,bid):
    post = Post.objects.get(id=bid)
    post.views = post.views + 1
    post.save()

    # 最新评论博客
    new_comment_list = Comment.objects.order_by('-pub_date').all()[:10]

    comment_list = post.comment_set.all()

    # 去重
    new_comment_list1 = []
    post_list1 = []
    for c in new_comment_list:
        if c.post.id not in post_list1:
            new_comment_list1.append(c)
            post_list1.append(c.post.id)

    # 博客标签
    tag_list = post.tags.all()

    # 相关推荐（标签相同的）
    post_recomment_list = set(Post.objects.filter(tags__in=tag_list)[:6])

    ctx = {
        'post': post,
        'new_comment_list': new_comment_list1,
        'post_recomment_list': post_recomment_list,
        'comment_list': comment_list

    }
    return render(request, 'show.html', ctx)
```
- 模板
```html
 {% for comment in comment_list %}
	<li class="comment-content"><span class="comment-f">#{{forloop.counter}}</span><div class="comment-main"><p><a class="address" href="#" rel="nofollow" target="_blank">{{comment.user.username}}</a><span class="time">({{comment.pub_date|date:'Y-m-d'}})</span><br>{{comment.content}}</p></div></li>
  {% endfor %}
```
# 二十三、实现登录功能
```html

{% extends 'base.html' %}
{% block title %}知奇博客-详细 {% endblock %}

{% block custom_css %}
<style type="text/css">
	.panel
	{
		padding: 80px 20px 0px;
		min-height: 400px;
		cursor: default;
	}
	.text-center
	{
		margin: 0 auto;
		text-align: center;
		border-radius: 10px;
		max-width: 900px;
		-moz-box-shadow: 0px 0px 5px rgba(0,0,0,.3);
		-webkit-box-shadow: 0px 0px 5px rgba(0,0,0,.3);
		box-shadow: 0px 0px 5px rgba(0,0,0,.1);
	}
	.float-left
	{
		float: left !important;
	}
	.float-right
	{
		float: right !important;
	}
	img
	{
		border: 0;
		vertical-align: bottom;
	}
	h2
	{
		padding-top: 20px;
		font-size: 20px;
	}
	.padding-big
	{
		padding: 20px;
	}
	.alert
	{
		border-radius: 5px;
		padding: 15px;
		border: solid 1px #ddd;
		background-color: #f5f5f5;
	}
</style>

{% endblock %}


{% block content %}


<section class="container">
<div class="panel">
<div class="row">
	<div class="col-md-4 col-md-offset-4">
		<p style="color:red">{{error_msg}}</p>
		<form action="/login/" method="post">
	  <div class="form-group">
	    <label for="exampleInputEmail1">用户名称</label>
	    <input type="text" name='username' class="form-control" id="exampleInputEmail1" placeholder="请输入用户名称...">
	  </div>
	  <div class="form-group">
	    <label for="exampleInputPassword1">密码</label>
	    <input type="password" name='password' class="form-control" id="exampleInputPassword1" placeholder="请输入密码...">
	  </div>

	  <div class="checkbox">
	    <label>
	      <input type="checkbox"> 自动登录
	    </label>
	  </div>
	  <button type="submit" class="btn btn-default">登录</button>
		{% csrf_token %}
	</form>

</div>

</div>
</div>
</section>

{% endblock %}

```
- 编写路由
```python
url(r'^login/', LoginView.as_view(), name='login'),
```

- 编写视图
```python
from django.shortcuts import render
from django.views.generic.base import View
from django.contrib.auth import authenticate, login, logout

class LoginView(View):
    def get(self, request):
        return render(request, 'login.html', {})
    def post(self, request):
        username = request.POST.get('username')
        password = request.POST.get('password')

        user = authenticate(username=username, password=password)

        if user:
            if user.is_active:
                login(request, user)
                return HttpResponseRedirect(reverse("index"))
            else:
                return render(request, 'login.html', {'error_msg':'用户未激活！'})
        else:
            return render(request, 'login.html', {'error_msg':'用户名或者密码错误！'})
```
- 修改模板 base.html
```html
 <ul class="nav navbar-nav navbar-right">
    {% if user.is_authenticated %}
    <li><a data-cont="用户" title="用户" href="#">欢迎，{{user.username}}</a></li>
    <li><a data-cont="注册" title="注册" href="/logout">注销</a></li>
    {% else %}
    <li><a data-cont="登录" title="登录" href="/login">登录</a></li>
    <li><a data-cont="注册" title="注册" href="/register">注册</a></li>
    {% endif %}
  </ul>
```
# 二十四、实现注册功能
```html


{% extends 'base.html' %}
{% block title %}知奇博客-注册{% endblock %}

{% block custom_css %}
<style type="text/css">
	.panel
	{
		padding: 80px 20px 0px;
		min-height: 400px;
		cursor: default;
	}
	.text-center
	{
		margin: 0 auto;
		text-align: center;
		border-radius: 10px;
		max-width: 900px;
		-moz-box-shadow: 0px 0px 5px rgba(0,0,0,.3);
		-webkit-box-shadow: 0px 0px 5px rgba(0,0,0,.3);
		box-shadow: 0px 0px 5px rgba(0,0,0,.1);
	}
	.float-left
	{
		float: left !important;
	}
	.float-right
	{
		float: right !important;
	}
	img
	{
		border: 0;
		vertical-align: bottom;
	}
	h2
	{
		padding-top: 20px;
		font-size: 20px;
	}
	.padding-big
	{
		padding: 20px;
	}
	.alert
	{
		border-radius: 5px;
		padding: 15px;
		border: solid 1px #ddd;
		background-color: #f5f5f5;
	}
</style>

{% endblock %}


{% block content %}


<section class="container">
<div class="panel">
<div class="row">
	<div class="col-md-4 col-md-offset-4">
		<p style="color:red">{{error_msg}}</p>
		<form action="/register/" method="post">
	  <div class="form-group">
	    <label for="exampleInputEmail1">用户名称</label>
	    <input type="text" name='username' class="form-control" id="exampleInputEmail1" placeholder="请输入用户名称...">
	  </div>

	  <div class="form-group">
	    <label for="exampleInputPassword1">密码</label>
	    <input type="password" name='password' class="form-control" id="exampleInputPassword1" placeholder="请输入密码...">
	  </div>

		<div class="form-group">
			<label for="exampleInputEmail">邮箱</label>
			<input type="email" name='email' class="form-control" id="exampleInputEmail" placeholder="请输入邮箱...">
		</div>

	  <button type="submit" class="btn btn-default">注册</button>
		{% csrf_token %}
	</form>

</div>

</div>
</div>
</section>

{% endblock %}

```
- 编写视图

```python
from django.contrib.auth.hashers import make_password

....

class RegisterView(View):
    def get(self, request):
        return render(request, 'register.html')
    def post(self, request):
        username = request.POST.get('username')
        password = request.POST.get('password')
        email = request.POST.get('email')

        # my_send_email(email)

        user = BlogUser()
        user.username = username
        user.password = make_password(password)
        user.email = email
        user.is_active = False

        user.save()

        return render(request, 'login.html', {})
```
# 二十五、实现注册验证功能
- settings.py中配置
```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.163.com'
EMAIL_PORT = 25
#发送邮件的邮箱
EMAIL_HOST_USER = 'xxx'
#在邮箱中设置的客户端授权密码
EMAIL_HOST_PASSWORD = '123456'
#收件人看到的发件人
EMAIL_FROM = '知奇课堂'
```

- 视图函数
```python
from random import Random
from django.core.mail import send_mail
from .models import EmailVerifyRecord
from blogpro.settings import EMAIL_FROM


# 生成随机字符串
def make_random_str(randomlength=8):
    str = ''
    chars = 'AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz0123456789'
    length = len(chars) - 1
    random = Random()
    for i in range(randomlength):
        str+=chars[random.randint(0, length)]
    return str

# 发送邮件
def my_send_email(email, send_type="register"):
    email_record = EmailVerifyRecord()
    if send_type == "update_email":
        code = make_random_str(4)
    else:
        code = make_random_str(16)
    email_record.code = code
    email_record.email = email
    email_record.send_type = send_type
    email_record.save()

    email_title = ""
    email_body = ""

    if send_type == "register":
        email_title = "知奇博客-注册激活链接"
        email_body = "请点击下面的链接激活你的账号: http://127.0.0.1:8000/active/{0}".format(code)

        send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
        if send_status:
            pass
    elif send_type == "forget":
        email_title = "知奇博客-网注册密码重置链接"
        email_body = "请点击下面的链接重置密码: http://127.0.0.1:8000/reset/{0}".format(code)

        send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
        if send_status:
            pass
    elif send_type == "update_email":
        email_title = "知奇博客-邮箱修改验证码"
        email_body = "你的邮箱验证码为: {0}".format(code)

        send_status = send_mail(email_title, email_body, EMAIL_FROM, [email])
        if send_status:
            pass
    


class ActiveView(View):
    def get(self, request, active_code):
        all_records = EmailVerifyRecord.objects.filter(code=active_code)
        if all_records:
            for record in all_records:
                email = record.email
                user = BlogUser.objects.get(email=email)
                user.is_active = True
                user.save()
        else:
            return render(request, "active_fail.html")
        return render(request, "login.html")
```
- 路由设置
```python
 url(r'^login/', LoginView.as_view(), name='login'),
    url(r'^register/', RegisterView.as_view(), name='register'),
    url(r'^active/(?P<active_code>[a-zA-Z0-9]+)', ActiveView.as_view(), name='active'),
    url(r'^logout/', LogoutView.as_view(), name='logout'),
```
# 二十六、实现注销功能
```python
class LogoutView(View):
    def get(self, request):
        logout(request)
        return HttpResponseRedirect(reverse("index"))
```
# 二十七、实现后台富文本功能
- 下载kindeditor
地址: http://kindeditor.net/demo.php

- 配置
在static/js 下面创建一个editor目录,添加
![选区_025](https://i.loli.net/2018/06/13/5b20fe6451b64.png)

创建一个config.js配置如下:
```js
 KindEditor.ready(function(K) {
                window.editor = K.create('#editor_id',{
                    //指定大小
                    width:'800px',
                    height:'200px',
                });
        });
```
- 在admin中注册(blogs下面的admin.py中)
```python
class PostAdmin(admin.ModelAdmin):
    class Media:
        js=(
            'js/editor/kindeditor-all.js',
            'js/editor/config.js',
        )
admin.site.register(Post,PostAdmin)
```
