# 创建一个Django + django test framework工程

## 1. 准备开发环境
## 2. 创建一个项目tutorial

  'django-admin startproject tutorial'

  'cd tutorial'

## 3. 创建一个app，名为quickstart

  - **Django规定，如果要使用模型，必须要创建一个app。我们使用以下命令创建一个 quickstart 的 app**

  - **将app名称添加到setting.py的INSTALLED_APPS中**

    'django-admin startapp quickstart'

  - **迁移数据库**

    'python manage.py migrate'

  - **添加一个记录，超级管理员**
    ' C:\workspace\turorial\tutorial>python manage.py createsuperuser --email admin@example.com --username admin '
    
## 4. 启动app
  `python manage.py runserver`

    ```
    (venv) C:\workspace\turorial\tutorial>python manage.py runserver

    Performing system checks...

    System check identified no issues (0 silenced).

    March 06, 2018 - 15:30:03

    Django version 1.11.10, using settings 'tutorial.settings'

    Starting development server at http://127.0.0.1:8000/

    Quit the server with CTRL-BREAK.

    ```

  然后通过http://172.0.0.1:8000访问 
  
  如果使用其他ip访问该web，则`python manage.py runserver 0:8080`

## 4.切换到venv模式，安装django rest framwork 


安装sqlite3 
连接数据库
python manage.py dbshell
创建表
注意： quickstart_hostserver 表名前面一定要加一个app的名字，即quickstart_hostserver，quickstart未app的名称，hostserver对应后续的models中的类名
```
create table quickstart_hostserver (
id integer primary key  not null,
name char(100) not null,
ip_addr char(100),
descriptor text,
status integer
);
```
插入数据
```
C:\workspace\turorial\tutorial>python manage.py shell
Python 2.7.14 (v2.7.14:84471935ed, Sep 16 2017, 20:25:58) [MSC v.1500 64 bit (AM
D64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from quickstart.models import HostServer
>>> from quickstart.serializers import HostServerSerializer
>>> from rest_framework.renderers import JSONRenderer
>>> from rest_framework.parsers import JSONParser
>>> hostserver = HostServer(id=5,name='主机五',ip_addr='192.168.78.5',descriptio
n='主机五', create_date='2018-03-06 18:45:00',status=1)
>>>hostserver.save()
```

查看数据库：

```
C:\workspace\turorial\tutorial>python manage.py dbshell
SQLite version 3.22.0 2018-01-22 18:45:57
Enter ".help" for usage hints.
sqlite> .tables
auth_group                  django_admin_log
auth_group_permissions      django_content_type
auth_permission             django_migrations
auth_user                   django_session
auth_user_groups            quickstart_hostserver
auth_user_user_permissions
sqlite> select * from quickstart_hostserver;
1|主机一|192.168.78.1|主机一|2018-03-06 15:45:00|0
2|主机二|192.168.78.2|主机二|2018-03-06 16:45:00|1
3|主机三|192.168.78.3|主机三|2018-03-06 08:17:37.246000|2
4|主机四|192.168.78.4|主机四|2018-03-06 18:45:00|0
5|主机五|192.168.78.5|主机五|2018-03-06 18:45:00|1
```
## 问题汇总
1. 'ListSerializer' object is not callable
2. 没有创建表：OperationalError: no such table: quickstart_hostserver
3. 本机未安装sqlite，会报错
4. You called this URL via POST, but the URL doesn't end in a slash and you have APPEND_SLASH set. Django can't redirect to the slash URL while maintaining POS
T data. Change your form to point to 127.0.0.1:8000/testTable/2/ (note the trailing slash), or set APPEND_SLASH=False in your Django settings.
6. JSPN.parse(request)报解析错误
7. NOT NULL constraint failed: table.ID
   由于手动建表，插入数据，TestTable.objects.all() 可以获取数据；但是插入数据时，报错：NOT NULL constraint failed: table.ID
8. 访问不存在的id时，报错：http://127.0.0.1:8000/testTable/3/
   但是访问：http://127.0.0.1:8000/testTable/pk=3/ 时是报404错误
   
       
    ```
    Internal Server Error: /testTable/3/
    Traceback (most recent call last):
    File "C:\workspace\turorial\tutorial\venv\lib\site-packages\django\core\handlers\exception.py", line 41, in inner
        response = get_response(request)
    File "C:\workspace\turorial\tutorial\venv\lib\site-packages\django\core\handlers\base.py", line 187, in _get_response
        response = self.process_exception_by_middleware(e, request)
    File "C:\workspace\turorial\tutorial\venv\lib\site-packages\django\core\handlers\base.py", line 185, in _get_response
        response = wrapped_callback(request, *callback_args, **callback_kwargs)
    File "C:\workspace\turorial\tutorial\venv\lib\site-packages\django\views\decorators\csrf.py", line 58, in wrapped_view
        return view_func(*args, **kwargs)
    File "C:/workspace/turorial/tutorial\quickstart\views\testtableviews.py", line 37, in testtable_detail
        except testTable.DoesNotExist:
    UnboundLocalError: local variable 'testTable' referenced before assignment
    ```
    原因分析：
    ```
    try:
        test_table = TestTable.objects.get(id=pk)
    except TestTable.DoesNotExist:  **写成了**test_table.DoesNotExist
        return HttpResponse(status=404)
    ```

9. TypeError: unbound method save() must be called with HostServer instance as first argument (got nothing instead)
    当向数据库中插入数据库时，是新的数据，序列化时，是从request中取解析后的data，放入序列化实例中。
    
    即：
    `serializer = HostServerSerializer(data=data)  `
    
    而不是
    ```
    hostserver =  HostServer.objects.all()
    serializer = HostServerSerializer(hostServer, data=data)
    ```
10. RuntimeError: You called this URL via PUT, but the URL doesn't end in a slash and you have APPEND_SLASH set. Django can't redirect to the slash URL while maintaining PUT data. Change your form to point to 127.0.0.1:8000/hostserver/3/ (note the trailing slash), or set APPEND_SLASH=False in your Django settings.
[07/Mar/2018 17:57:09] "PUT /hostserver/3 HTTP/1.1" 500 65655

  访问url时，http://127.0.0.1:8000/hostserver/3/   最后的/不要省略

11. ValueError: The view autodeployapi.views.hostserverviews.hostserver_detail didn't return an HttpResponse object. It returned None instead
  应该put访问，但是使用了post访问，去修改数据。导致serializer = HostServerSerializer(hostserver, data=data) 参数中hostserver报错
12. TypeError: In order to allow non-dict objects to be serialized set the safe parameter to False. 
  return JsonResponse(serializer.data, safe=False)  设置JsonResponse（第二个参数为safe=false）
  https://stackoverflow.com/questions/37909800/how-to-convert-a-list-of-dictionaries-to-json-in-python-django/37909858

13. AssertionError: The `.update()` method does not support writable nested fields by default.
Write an explicit `.update()` method for serializer `autodeployapi.serializers.ServerRoleSerializer`, or set `read_only=True` on nested serializer fields.

    设置read_only=True
    ```
    class ServerRoleSerializer(serializers.ModelSerializer):
        hostserver = HostServerSerializer(many=True, read_only=True)
    ```
14. TypeError: 'ManyRelatedManager' object is not iterable
15. Django rest serializer.data is an empty OrderedDict()
16.  File "C:\autodeployworkspace\autodeploy\venv\lib\site-packages\fabric\network.py", line 259, in parse_host_string
    user_hostport = host_string.rsplit('@', 1)
AttributeError: 'list' object has no attribute 'rsplit'


17. (venv) C:\GitWorkspace\deploy>python manage.py runserver 0.0.0.0:8001
Unhandled exception in thread started by <function wrapper at 0x000000000367CAC8>
Traceback (most recent call last):
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\utils\autoreload.py", line 228, in wrapper
    fn(*args, **kwargs)
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\core\management\commands\runserver.py", line 117, in inner_run
    autoreload.raise_last_exception()
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\utils\autoreload.py", line 251, in raise_last_exception
    six.reraise(*_exception)
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\utils\autoreload.py", line 228, in wrapper
    fn(*args, **kwargs)
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\__init__.py", line 27, in setup
    apps.populate(settings.INSTALLED_APPS)
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\apps\registry.py", line 85, in populate
    app_config = AppConfig.create(entry)
  File "C:\GitWorkspace\deploy\venv\lib\site-packages\django\apps\config.py", line 94, in create
    module = import_module(entry)
  File "C:\Python27\Lib\importlib\__init__.py", line 37, in import_module
    __import__(name)
ImportError: No module named corsheaders

在命令行中执行`python manage.py runserver 0.0.0.0:8000`之前需要到venv中的Scripts激活activity.bat

在setting.py中
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    "rest_framework",
    "corsheaders",
    "deploy_api"
]
```
定义了 corsheaders ，但是在venv中并未安装：
`pip install django-cors-headers`