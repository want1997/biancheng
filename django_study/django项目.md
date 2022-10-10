# HTTP请求的url路由表

1.查看是否安装django，并安装django

pip install django

pip -m django --version

![image-20221006155614825](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006155614825.png)

2.创建F:\project 文件夹

django-admin startproject bysms

![image-20221006155905035](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006155905035.png)

3.开启web服务



python .\manage.py runserver 0.0.0.0:80

![image-20221006161039917](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006161039917.png)

![image-20221006161151451](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006161151451.png)

没有设置其他的东西，其他的人就访问不了

![image-20221006161335747](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006161335747.png)

4.创建一个app （就相当于一个模块功能）

![image-20221006161819180](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006161819180.png)

5.在新建的模块sales模块中的view.py文件中写入 凡是浏览器访问的http 请求的 url 地址 是 `/sales/orders/` , 就由 views.py 里面的函数 `listorders` 来处理， 返回一段字符串给浏览器。

但是此时并不能正常访问，因为需要url路由设置

![image-20221006162414373](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006162414373.png)

6.在url中添加相对应的路由，导入sales模块中的listorders函数（发现爆红，解决方法，Project Structure中的Sources）

![image-20221006163554496](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006163554496.png)

![image-20221006163736023](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006163736023.png)

此时就可以看到显示的效果

![image-20221006164029872](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006164029872.png)

注意：

只要修改了路由表配置，添加了我们自己的路由记录，再去浏览器访问 首页，这里就是 `http://127.0.0.1` ，前面曾经出现的小火箭欢迎页就不见了！ 会出现一个 404 Not Found 的报错页面。

这是正常的，小火箭欢迎页面 是Django在调试模式下，发现路由记录没有添加的时候，缺省作为首页的。 真正的产品是不会使用这个首页的。一旦路由记录发生变动， 就会消失。

![image-20221006164125076](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006164125076.png)

7.路由子表

路由子表的主要解决的一个分级（按照不同功能模块分级）的问题，不把所有的url链接全部放入到主文件夹的匹配模式中

主要两个步骤

1.在功能模块中新建一个urls.py文件，在里面进行路径与功能函数的映射关系

![image-20221006165631259](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006165631259.png)

2.修改主url路由文件，实现模块功能路径对应相关的url





url 路由表就是可以像上面这样，一个请求对应一个处理函数。

但是有的时候，我们的项目比较大的时候， 请求的url 会特别多。

比如我们的系统提供给 客户、销售商、管理员 访问的url是不一样的，如下

```py
customer/
customer/orders/      

sales/
sales/orders/  

mgr/
mgr/customers/
mgr/medicines/
mgr/orders/
```

复杂的系统url条目多达几百甚至上千个， 放在一个表中，查看时，要找一条路由记录就非常麻烦。

这时，我们通常可以将不同的路由记录 按照功能 分拆到不同的 **url路由子表** 文件中。

比如，这里我们可以把 访问 的 url 凡是 以 `sales` 开头的全部都 由 sales app目录下面的 子路由文件 urls.py 处理。

首先我们需要在 sales 目录下面创建一个新的文件 `sales\urls.py` 。

然后在这个 `sales\urls.py` 文件中输入如下内容

```py
from django.urls import path

from . import views

urlpatterns = [
    path('orders/', views.listorders),
]
```

然后，我们再修改主url路由文件 `bysms/urls.py` , 如下

```py
from django.contrib import admin

# 导入一个include函数
from django.urls import path, include

from sales.views import listorders
urlpatterns = [
    path('admin/', admin.site.urls),

    # 凡是 url 以 sales/  开头的，
    # 都根据 sales.urls 里面的 子路由表进行路由
    path('sales/', include('sales.urls')),

]
```

当一个http请求过来时， Django检查 url，比如这里是 `sales/orders/`，

先到主url路由文件 `bysms/urls.py`中查看 是否有匹配的路由项。

如果有匹配 ( 这里匹配了 `sales/` )， 并且匹配的对象 不是 函数， 而是 一个子路由设置 , 比如这里是 `include('sales.urls')`

就会去子路由文件中查看， 这里就是 sales.urls 对应的文件 `sales\urls.py` 。

注意这时，会从请求url中去掉 前面主路由文件 已经匹配上的部分（这里是 `sales/` ）, 将剩余的部分 （这里是 `orders/` ）去子路由文件中查看是否有匹配的路由项。

这里就匹配了 `orders/` ，匹配的对象，这里是 `views.listorders` ，它是一个处理函数，就调用该函数处理 这个http请求， 将该函数的返回对象 构建 HTTP响应消息，返回给客户端。

# 创建数据库和表

关系型数据库系统，常用的开源数据库有 mysql 和 postgresql。建议大家实际工作中使用的时候，使用上面这两种。

但是上面这些数据库，都需要我们安装数据库服务系统 和 客户端库，比较麻烦，现在我们先使用另一种更简单的 数据库 **sqlite**。

sqlite 没有 独立的数据库服务进程，数据操作被做成库直接供应用程序调用。 Django中可以直接使用，无须先搭建数据服务。

后面大家要使用mysql 等其他数据库 只需修改一些配置就可以了。

创建数据库
项目中数据库的配置：在bysms/settings.py中：

![image-20221006190634461](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006190634461.png)

大家可以发现，我们使用命令创建的项目， 缺省就是使用 sqlite。 而且对于的数据库文件，缺省的文件名是 `db.sqlite3` ， 就在项目的根目录下面


首先我们需要创建数据库，执行如下命令

```py
python manage.py migrate
```

![image-20221006190827357](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006190827357.png)

![image-20221006190846367](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006190846367.png)

![image-20221006190919291](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006190919291.png)

了解ORM（Object Relational Mapping）对象关系映射

本质是利用类对对应相关的数据表，类与实例的映射



参考链接：[科普文-什么是ORM? - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/27188788)



创建一个common的应用目录，存放一些公共的表的定义

进入项目根目录，执行下面的命令。

```py
python manage.py startapp common 
```

![image-20221006191515519](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006191515519.png)

打开 common/models.py，发现里面是空的，因为我们还没有定义我们的业务所需要的表。

我们修改它，加入如下内容

```py
from django.db import models

class Customer(models.Model):
    # 客户名称
    name = models.CharField(max_length=200)

    # 联系电话
    phonenumber = models.CharField(max_length=200)

    # 地址
    address = models.CharField(max_length=200)
```

这个 Customer 类继承自 django.db.models.Model， 就是用来定义数据库表的。

里面的 name、phonenumber、address 是该表的3个字段。

定义表中的字段 就是定义一些静态属性，这些属性是 django.db.models 里面的各种 Field 对象，对应不同类型的字段。

比如这里的3个字段 都是 CharField 对象，对应 varchar类型的数据库字段。

后面的参数 `max_length` 指明了该 varchar字段的 最大长度。

Djanog 有很多字段对象类型， 对应不同的类型的数据库字段。

定义好表以后，我们怎么真正去创建数据库表呢？

首先我们需要告诉Django： 我们的 common 应用中 需要你关注， 因为其中包含了 数据库Model的定义。

怎么告诉它？ 在项目的配置文件 `settings.py `中， INSTALLED_APPS 配置项 加入如下内容

```py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # 加入下面这行
    'common.apps.CommonConfig',
]
```

‘common.apps.CommonConfig’ 告诉 Django ， CommonConfig 是 common/apps.py 文件中定义的一个应用配置的类。

...............

# 读取数据库数据

前面，我们已经创建了数据库和 Customer表。

现在我们来实现一个功能：浏览器访问 `sales/customers/` ，我们的服务端就返回系统中所有的客户记录给浏览器。

我们先实现一个函数，来处理浏览器发出的URL为 `sales/customers/` 的访问请求， 我们需要返回 数据库中 customer 表 所有记录。

Django 中 对数据库表的操作， 应该都通过 Model对象 实现对数据的读写，而不是通过SQL语句。

比如，这里我们要获取 customer 表 所有记录， 该表是和我们前面定义的 Customer 类管理的。

我们可以这样获取所有的表记录:

在文件sales/views.py 中，定义一个listcustomers 函数，内容如下：

```py
# 导入 Customer 对象定义
from  common.models import  Customer

def listcustomers(request):
    # 返回一个 QuerySet 对象 ，包含所有的表记录
    # 每条表记录都是是一个dict对象，
    # key 是字段名，value 是 字段值
    qs = Customer.objects.values()

    # 定义返回字符串
    retStr = ''
    for customer in  qs:
        for name,value in customer.items():
            retStr += f'{name} : {value} | '

        # <br> 表示换行
        retStr += '<br>'

    return HttpResponse(retStr)
```

Customer.objects.values() 就会返回一个 QuerySet 对象，这个对象是Django 定义的，在这里它包含所有的Customer 表记录。

QuerySet 对象 可以使用 for 循环遍历取出里面所有的元素。每个元素 对应 一条表记录。

每条表记录元素都是一个dict对象，其中 每个元素的 key 是表字段名，value 是 该记录的字段值

上面的代码就可以将 每条记录的信息存储到字符串中 返回给 前端浏览器。

我们还需要修改路由表， 加上对 `sales/customers/` url请求的 路由。

前面，我们在bysms\urls.py 主路由文件中，已经有如下的记录了

```py
    # 凡是 url 以 sales/  开头的，
    # 都根据 sales.urls 里面的 子路由表进行路由
    path('sales/', include('sales.urls')),
```

这条URL记录，指明 凡是 url 以 sales/ 开头的，都根据 sales.urls 里面的 子路由表进行路由。

我们只需修改 `sales/urls.py` 即可，添加如下记录

```py
    path('customers/', views.listcustomers),
```



大家可以使用 admin 登录， 再添加一些 客户记录。

然后可以在浏览器输入如下 网址： `http://127.0.0.1/sales/customers/`

![image-20221006194344994](F:\mygitplace\编程\biancheng\django_study\django项目.assets\image-20221006194344994.png)