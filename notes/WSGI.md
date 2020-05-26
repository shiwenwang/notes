---
tags: [Python]
title: WSGI
created: '2020-05-25T05:14:34.604Z'
modified: '2020-05-26T03:57:31.933Z'
---

# WSGI

## 介绍

WSGI 全称 The Python Web Server Gateway Interface， 它是描述 Web 服务器如何与 Web 应用程序通信以及如何将Web 应用程序链接在一起以处理一个请求的规范。
> :bulb: 它不是一个服务、Python 模块、框架或者任何种类的软件，它只是 web 服务器和 web 应用之间通讯的接口规范。来自 [PEP333](https://www.python.org/dev/peps/pep-3333/)。

在框架或工具包之上构建应用程序，学习 WSGI 规范不是必须的。要使用中间件，必须至少了解如何将它们与应用程序或框架进行堆叠^[WSGI applications (meaning WSGI compliant) can be stacked. 符合 WSGI 规范的应用程序是可以堆叠的，在堆叠的应用程序中间起连接作用的是中间件。]，除非它已经集成在框架中，或者框架提供某种封装器来集成那些没有集成的中间件。

## 应用接口

WSGI 应用接口被实现为一个可调用对象（callable object），可以是函数、类、方法、或者 `object.__call__()` 实例方法。

这个对象接受必须：
1. 接受两个位置参数：
    - 包含类似 CGI 变量的字典；
    - 回调函数，用于向服务器发送 HTTP 状态码、信息以及 HTTP 请求头。

2. 将响应主体作为包装在可迭代对象中的字符串返回给服务器。

应用程序例子：
```python
# The application interface is a callable object
def application ( # It accepts two arguments:
    # environ points to a dictionary containing CGI like environment
    # variables which is populated by the server for each
    # received request from the client
    environ,
    # start_response is a callback function supplied by the server
    # which takes the HTTP status and headers as arguments
    start_response
):

    # Build the response body possibly
    # using the supplied environ dictionary
    response_body = 'Request method: %s' % environ['REQUEST_METHOD']

    # HTTP response code and message
    status = '200 OK'

    # HTTP headers expected by the client
    # They must be wrapped as a list of tupled pairs:
    # [(Header name, Header value)].
    response_headers = [
        ('Content-Type', 'text/plain'),
        ('Content-Length', str(len(response_body)))
    ]

    # Send them to the server using the supplied function
    start_response(status, response_headers)

    # Return the response body. Notice it is wrapped
    # in a list although it could be any iterable.
    return [response_body]
```
> Tip: 在实例化服务器之前，以上代码是不可能运行的。

## 环境字典

服务器在客户端发出的每个请求中都使用 CGI 变量来填充环境字典。下面的脚本将输出整个字典：

```python
#! /usr/bin/env python

# Python's bundled WSGI server
from wsgiref.simple_server import make_server

def application (environ, start_response):

    # Sorting and stringifying the environment key, value pairs
    response_body = [
        '%s: %s' % (key, value) for key, value in sorted(environ.items())
    ]
    response_body = '\n'.join(response_body)

    status = '200 OK'
    response_headers = [
        ('Content-Type', 'text/plain'),
        ('Content-Length', str(len(response_body)))
    ]
    start_response(status, response_headers)

    return [response_body]

# Instantiate the server(实例化服务器)
httpd = make_server (
    'localhost', # The host name
    8051, # A port number where to wait for the request
    application # The application object name, in this case a function
)

# Wait for a single request, serve it and quit
httpd.handle_request()
```

保存上述代码至 `environment.py`, 打开终端，定位到该文件所在目录，运行 `python environment.py`。此时，服务器启动，监听 [http://localhost:8051](http://localhost:8051)。

# CGI 环境变量
以下变量必须存在，除非它们的值是一个空字符串，在这种情况下它们可以被省略，除非下面另有说明。

| 变量名 | 说明 |
| --- | --- | 
| `REQUEST_METHOD` | The HTTP request method, such as "GET" or "POST". This cannot ever be an empty string, and so is always required. |
| `SCRIPT_NAME` | The initial portion of the request URL's "path" that corresponds to the application object, so that the application knows its virtual "location". This may be an empty string, if the application corresponds to the "root" of the server. |
| `PATH_INFO` | The remainder of the request URL's "path", designating the virtual "location" of the request's target within the application. This may be an empty string, if the request URL targets the application root and does not have a trailing slash. |
| `QUERY_STRING` | The portion of the request URL that follows the "?", if any. May be empty or absent. |
| `CONTENT_TYPE` | The contents of any Content-Type fields in the HTTP request. May be empty or absent. |
| `CONTENT_LENGTH` | The contents of any Content-Length fields in the HTTP request. May be empty or absent. |
| `SERVER_NAME`, `SERVER_PORT` | When combined with SCRIPT_NAME and PATH_INFO, these two strings can be used to complete the URL. Note, however, that HTTP_HOST, if present, should be used in preference to SERVER_NAME for reconstructing the request URL. See the [URL Reconstruction](https://www.python.org/dev/peps/pep-3333/#url-reconstruction) section below for more detail. SERVER_NAME and SERVER_PORT can never be empty strings, and so are always required. |
| `SERVER_PROTOCOL` | The version of the protocol the client used to send the request. Typically this will be something like "HTTP/1.0" or "HTTP/1.1" and may be used by the application to determine how to treat any HTTP request headers.(This variable should probably be called REQUEST_PROTOCOL, since it denotes the protocol used in the request, and is not necessarily the protocol that will be used in the server's response. However, for compatibility with CGI we have to keep the existing name.) |
| `HTTP_ `**Variables** | Variables corresponding to the client-supplied HTTP request headers (i.e., variables whose names begin with "HTTP_"). The presence or absence of these variables should correspond with the presence or absence of the appropriate HTTP header in the request. |

