---
attachments: [Clipboard_2020-03-27-22-12-44.png]
tags: [Python]
title: Pip
created: '2020-03-27T14:04:15.408Z'
modified: '2020-03-27T14:18:06.430Z'
---

# Pip

- 安装
`pip install <package>` or 
`pip install <package>=version` or 
`pip install -r requirements.txt` or
`pip install <package> -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com `

- 卸载
`pip uninstall <package>` or 
`pip uninstall <package>=version`

- 配置源
  - Windows
    1. 用户文件夹下新建`pip/pip.ini`文件。`eg. c:\users\username\pip\pip.in`
    2. 文件中写入
      ```
        [global]
        index-url = http://mirrors.aliyun.com/pypi/simple
        [install]
        --trusted-host=mirrors.aliyun.com
      ```
  - Linux
    1. 新建`~/.pip/pip.conf`文件
    2. 文件中写入
      ```
        [global]
        index-url = http://mirrors.aliyun.com/pypi/simple
        [install]
        --trusted-host=mirrors.aliyun.com
      ```

- 导出环境的包
`pip freeze > requirements.txt`

