dblog
=====

django练习：个人博客系统开发
1.新建项目
目前SAE上预置了两个版本的django，1.2.7和1.4，默认的版本为1.2.7，在本示例中我们使用1.4版本。

创建一个新的python应用，检出svn代码到本地目录并切换到应用目录。

创建一个django project：blog。

HansontekiMacBook-Pro:hansonsblog hanson$django-admin.py startproject blog
HansontekiMacBook-Pro:hansonsblog hanson$ls blog
HansontekiMacBook-Pro:hansonsblog hanson$manage.py  blog/
重命名该project的根目录名为1，作为该应用的默认版本代码目录。

HansontekiMacBook-Pro:hansonsblog hanson$mv mysite 1
在默认版本目录下创建应用配置文件 config.yaml ，在其中添加如下内容：

libraries:
- name: "django"
version: "1.4"
创建文件index.wsgi，内容如下:

import sae
from mysite import wsgi
application = sae.create_wsgi_app(wsgi.application)
最终目录结构如下:

HansontekiMacBook-Pro:hansonsblog hanson$ ls 1
config.yaml index.wsgi manage.py blog/
HansontekiMacBook-Pro:hansonsblog hanson/1$ ls 1/blog
__init__.py settings.py  urls.py  views.py
2.新建APP
在终端中打开项目目录输入

python manage.py startapp sblog
现在已建好一个名为sblog的博客应用

sblog/
     __init__.py
     models.py
     tests.py
     views.py
3.Models的配置
我使用的数据库是MySQL,需要先装好MySQLdb,下面介绍一下MySQL的配置，打开setting.py。

if 'SERVER_SOFTWARE' in os.environ: # 线上模式
    from sae.const import (
        MYSQL_HOST, MYSQL_PORT, MYSQL_USER, MYSQL_PASS, MYSQL_DB
    )
else: # 本地模式
    MYSQL_HOST = 'localhost'
    MYSQL_PORT = '3306'
    MYSQL_USER = 'XXXX'
    MYSQL_PASS = 'XXXXXX'
    MYSQL_DB   = 'blog'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': MYSQL_DB,                      # Or path to database file if using sqlite3.
        'USER': MYSQL_USER,                      # Not used with sqlite3.
        'PASSWORD': MYSQL_PASS,                  # Not used with sqlite3.
        'HOST': MYSQL_HOST,                      # Set to empty string for localhost. Not used with sqlite3.
        'PORT': MYSQL_PORT,                      # Set to empty string for default. Not used with sqlite3.
    }
}
如上的配置可以很方便的让我们在开发的时候使用。 接下来配置models.py。

# -*- coding: utf-8 -*-
from django.db import models
from django.db.models import permalink
from markdown import markdown

# Create your models here.
class Category(models.Model): # 分类表

    name = models.CharField(max_length=10,verbose_name=u'名称')
    slug = models.CharField(max_length=50,unique=True,verbose_name=u'Slug')

    def __unicode__(self):
        return self.name

    @permalink
    def get_absolute_url(self):
        return ('blog_category',None,{'slug':self.slug})

    class Meta:
        ordering = ['id']
        verbose_name_plural = verbose_name = u'分类'

class Tag(models.Model): # 标签

    tag_name = models.CharField(max_length=20,blank=True,verbose_name=u'名称')
    create_time = models.DateTimeField(auto_now_add=True,verbose_name=u'建立时间')

    def __unicode__(self):
        return self.tag_name

    class Meta:
        verbose_name_plural = verbose_name = u'标签'

class Blog(models.Model): # 文章

    caption = models.CharField(max_length=50,verbose_name=u'标题')
    slug = models.SlugField(max_length=50,unique=True,verbose_name=u'Slug')
    tags = models.ManyToManyField(Tag,blank=True,verbose_name=u'标签名称')
    content = models.TextField(verbose_name=u'内容')
    publish_time = models.DateTimeField(auto_now_add=True,verbose_name=u'发表时间')
    update_time = models.DateTimeField(auto_now=True,verbose_name=u'更新时间')
    counts = models.IntegerField(default=0,verbose_name=u'阅读数')
    category = models.ForeignKey(Category,verbose_name=u'分类')

    def __unicode__(self):
        return u'%s %s' % (self.caption,self.publish_time)

    @permalink
    def get_absolute_url(self):
        return ('blog_article',None,{'slug':self.slug})

    class Meta:
        get_latest_by = 'publish_time'
        ordering = ['-id']
        verbose_name_plural = verbose_name = u'文章'

class ClientInfo(models.Model):

    ip_address = models.CharField(max_length=20,verbose_name=u'IP地址')
    time = models.DateTimeField(auto_now=True,verbose_name=u'访问时间')

    def __unicode__(self):
        return u'%s %s' % (self.ip_address,self.time)

    class Meta:
        get_latest_by = 'time'
        ordering = ['-id']
        verbose_name_plural = verbose_name = u'访问时间'
接下来修改setting.py。

MIDDLEWARE_CLASSES = (
    'django.middleware.common.CommonMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    # Uncomment the next line for simple clickjacking protection:
    # 'django.middleware.clickjacking.XFrameOptionsMiddleware',
)
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
     'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
     'django.contrib.admindocs',
     'sblog',
)
然后，在终端输入一下命令，检验Model的有效性：

python manage.py validate
如果一切正确，你将会看到0 errors found。

接着在终端中输入：

python manage.py syncdb
终端会显示Would you like to create one now?(yes/no):

这是让我们新建用户用于admin管理，直接新建就可以。

4.admin的配置
修改blog目录下的urls.py,添加：

from django.contrib import admin
admin.autodiscover()
在patterns添加：

url(r'^admin/doc/', include('django.contrib.admindocs.urls')),
url(r'^admin/', include(admin.site.urls)),
至此，打开http://1270.0.1:8080/admin/，就可以使用admin管理后台。
