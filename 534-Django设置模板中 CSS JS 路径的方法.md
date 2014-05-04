## 设置 settings.py

首先在 `settings.py` 中添加下面的代码

	STATICFILES_DIRS = (
	    os.path.join(BASE_DIR, "static"),
	)
	STATIC_URL = '/static/'
	STATIC_ROOT = os.path.join(BASE_DIR, 'collect_static')

*   `STATICFILES_DIRS` 是静态文件的目录，这里指定到项目目录下的 `static` 文件夹下

*   `STATIC_URL` 是访问 `static` 文件夹内容所需要使用的 URL 地址，在 `urls.py` 中用到

*   `STATIC_ROOT` 是存放静态文件的目录，与 `STATICFILES_DIRS` 的区别是该目录由 `python manage.py collectstatic`

命令填充，用于生产环境，不需人为干预

## 设置 urls.py

在 `urls.py` 的最后添加如下代码

	if settings.DEBUG and settings.STATIC_ROOT:
	    urlpatterns += static(settings.STATIC_URL,
	                          document_root=settings.STATIC_ROOT)

这里的 URL 配置只在 DEBUG 模式下启用，生产环境下直接 Nginx 或者 Apache 处理这个转发了。

## 模板中的使用

在模板中如果想使用 `static` 文件夹中的文件，只需要在文件头写入

	{% load staticfiles %}

然后在需要的地方直接用

	{% static "your_file_path" %}

即可在相应位置直接替换成对应的绝对路径。